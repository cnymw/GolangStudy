# grpool

## 线程池简介

在介绍 grpool 之前简单介绍一下线程池的概念。

### 线程池概念

线程池是一种多线程处理形式，处理过程中将任务添加到队列，然后在创建线程后自动启动这些任务。

构建服务器应用程序的一个过于简单的模型是：每当一个请求到达就创建一个新的服务对象，然后在新的服务对象中为请求服务。

但当有大量请求并发访问时，服务器不断的创建和销毁对象的开销很大。所以提高服务器效率的一个手段就是尽可能减少创建和销毁对象的次数，特别是一些很耗资源的对象创建和销毁，这样就引入了“池”的概念，“池”的概念使得人们可以定制一定量的资源，然后对这些资源进行复用，而不是频繁的创建和销毁。

### 工作机制

- 线程池是预先创建线程的一种技术。
  
- 线程池在还没有任务到来之前，创建一定数量的线程，放入空闲队列中。
  
- 这些线程都是处于睡眠状态，即均为启动，不消耗CPU，而只是占用较小的内存空间。
  
- 当请求到来之后，缓冲池给这次请求分配一个空闲线程，把请求传入此线程中运行，进行处理。
  
- 当预先创建的线程都处于运行状态，即预制线程不够，线程池可以自由创建一定数量的新线程，用于处理更多的请求。
  
- 当系统比较闲的时候，也可以通过移除一部分一直处于停用状态的线程。

### 使用线程池的原因

- 线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。 这避免了在处理短时间任务时创建与销毁线程的代价。
  
- 线程池不仅能够保证内核的充分利用，还能防止过分调度。
  

### 线程池注意点

- 线程池大小：多线程应用并非线程越多越好，需要根据系统运行的软硬件环境以及应用本身的特点决定线程池的大小。一般来说，如果代码结构合理的话，线程数目与 CPU 数量相适合即可。如果线程运行时可能出现阻塞现象，可相应增加池的大小；如有必要可采用自适应算法来动态调整线程池的大小，以提高 CPU 的有效利用率和系统的整体性能。

- 并发错误：多线程应用要特别注意并发错误，要从逻辑上保证程序的正确性，注意避免死锁现象的发生。

- 线程泄漏：这是线程池应用中一个严重的问题，当任务执行完毕而线程没能返回池中就会发生线程泄漏现象。

### 组成部分

1. 线程池管理器：用于创建并管理线程池
   
2. 工作线程：线程池中线程
   
3. 任务接口：每个任务必须实现的接口，以供工作线程调度任务的执行
   
4. 任务队列：用于存放没有处理的任务，提供缓冲的能力

## grpool 源码解析

区别于 JDK 的 Executor，grpool 是轻量级 Goroutine 线程池，代码量少（不到 150 行代码），逻辑简洁。

客户端可以提交 job，调度程序接受 job，并将其发送给第一个可用的 worker 进程。 当 worker 完成处理作业时，将返回到 worker 池。 worker 数量和 job 队列大小是可配置的。

### 示例代码

以下是官网上面推荐的最基本的 grpool 使用示例，可以看到， grpool 的使用特别的简单：

```go
func TestExample(t *testing.T) {
	// 配置worker的数量和job队列的大小
	pool := grpool.NewPool(100, 50)

	// 释放线程池的资源
	defer pool.Release()

	// 向线程池提交一个或多个job
	for i := 0; i < 10; i++ {
		count := i

		pool.JobQueue <- func() {
			fmt.Printf("I am worker! Number %d\n", count)
		}
	}

	// 比较简单的等待job完成
	time.Sleep(1 * time.Second)
}
```

### 代码分析

#### 构造函数 func NewPool()

首先看下 grpool 的构造函数 `NewPool()` 的实现：

```go
// 该函数将创建（make）线程池。
// numWorkers —— 这个线程池将创建 numWorkers 个 worker
// queueLen —— 线程池在阻塞（block）前能够接受 queueLen 个 job
//
// 返回的对象包含 JobQueue 引用，你可以使用它将发送到线程池。
func NewPool(numWorkers int, jobQueueLen int) *Pool {
	jobQueue := make(chan Job, jobQueueLen)
	workerPool := make(chan *worker, numWorkers)

	pool := &Pool{
		JobQueue:   jobQueue,
		dispatcher: newDispatcher(workerPool, jobQueue),
	}

	return pool
}
```

在这个初始化函数里，定义了两个重要的对象：`worker` 和 `Job`，这两个对象即线程池概念里的`工作线程`和`任务接口`。

---

#### 数据结构 struct worker

其中，数据结构`worker`的定义如下所示：

```go
// 可以接受客户端 job 的 Gorouting 实例
type worker struct {
	workerPool chan *worker
	jobChannel chan Job
	stop       chan struct{}
}
```

worker 中有 3 个对象：

1. workerPool：线程池，该对象的大小决定了线程池能够容纳的线程的大小，也就是同时运行的线程的大小。
2. jobChannel：任务队列，用于存放未完成的任务列表，该队列的大小决定了能够容纳的任务队列的大小，如果该队列满了的话，那么再往里面添加任务会 block。
3. stop：信号量，当这个参数接受到信号时，代表着这个 worker 不需要进行工作了，可以销毁线程。

---

#### 任务接口 func Job()

任务接口`job`的结构如下所示：

```go
// 表示用户请求，该函数应该在某个工作线程 worker 中执行。
type Job func()
```

`Job`即每个任务必须实现的接口，实现的逻辑由客户端定义，工作线程`worker`的核心工作便是执行`Job`。

---

#### 数据结构 struct pool

回到构造函数`NewPool()`，该函数还会创建`pool`对象，`pool`数据结构如下：

```go
type Pool struct {
	JobQueue   chan Job
	dispatcher *dispatcher
	wg         sync.WaitGroup
}
```

pool 中有 3 个对象：

1. JobQueue：任务队列，该变量是 grpool 唯一一个包外可见的变量，用于接受用户传入的`Job`接口。
2. dispatcher：线程调度器，用于对`worker`和`job`进行分发。
3. wg：同步等待组`WaitGroup`，能够等待 goroutine 的执行结束，可以避免 goroutine 还没执行完就退出程序。

---

#### 数据结构 struct dispatcher

`dispatcher`的定义如下：

```go
// dispatcher负责从客户端接受job，同时等待第一个空闲的worker来分配job
type dispatcher struct {
	workerPool chan *worker
	jobQueue   chan Job
	stop       chan struct{}
}
```

`dispatcher`的结构和`worker`类似，`dispatcher`负责分发，`worker`负责接收并处理，处理完成后再返回给`dispatcher`重新再次操作分发逻辑。

对象`pool`通过包外可见的参数`JobQueue`从客户端那里接受进来，给到`dispatcher`的参数`jobQueue`，进而分发到各个`worker`。

从客户端到`worker`执行`job`的流程图如下所示:

![go-grpool-grpool流程图.png](https://cnymw.github.io/GolangStudy/docs/img/go-grpool-grpool流程图.png)

---

#### func newDispatcher()

dispatcher 由 newDispatcher 负责创建。

```go
func newDispatcher(workerPool chan *worker, jobQueue chan Job) *dispatcher {
	d := &dispatcher{
		workerPool: workerPool,
		jobQueue:   jobQueue,
		stop:       make(chan struct{}),
	}

	for i := 0; i < cap(d.workerPool); i++ {
		worker := newWorker(d.workerPool)
		worker.start()
	}

	go d.dispatch()
	return d
}
```

首先会创建 dispatcher 对象，具体的参数含义之前已经描述过了。

然后会创建 numWorkers 个 worker 对象，这个数量 numWorkers 由客户端指定。

当`worker.start()`调用完成后，worker 将会持续运行，但这个时候还没有 job 分配过来，所以会有一段时间空闲。

---

#### func dispatch()

新建完成 dispatcher，会立即开始调用`d.dispatch()`来调度线程。

```go
func (d *dispatcher) dispatch() {
	for {
		select {
		case job := <-d.jobQueue:
			worker := <-d.workerPool
			worker.jobChannel <- job
		case <-d.stop:
			for i := 0; i < cap(d.workerPool); i++ {
				worker := <-d.workerPool

				worker.stop <- struct{}{}
				<-worker.stop
			}

			d.stop <- struct{}{}
			return
		}
	}
}
```

调度线程分为以下两步步：

1. 从空闲 worker 池 workerPool 里取出一个空闲的 worker。
2. 从 jobChannel 里取出一个 job，分配给空闲的 worker。

需要注意的是，worker 的 jobChannel 是个不带缓存的 channel，代表着一个 worker 只能处理一个 job，如果继续往 jobChannel 里添加 job 会阻塞。

dispatcher 的结束信号也是在`dispatch()`里处理，当客户端调用`pool.Release()`时，会发送一个信号量`struct{}{}`给 stop，在这里会 select 到该 stop，从而批量的结束 worker 线程，原理和结束 dispatcher 一样的。

---