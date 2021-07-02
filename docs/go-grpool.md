# grpool

## 线程池简介

在介绍 grpool 之前简单介绍一下线程池的概念。

### 线程池概念

线程池是一种多线程处理形式，处理过程中将任务添加到队列，然后在创建线程后自动启动这些任务。

### 工作机制

- 线程池线程都是后台线程
- 每个线程都使用默认的堆栈大小，以默认的优先级运行，并处于多线程单元中
- 如果某个线程在托管代码中空闲（如正在等待某个事件）,则线程池将插入另一个辅助线程来使所有处理器保持繁忙
- 如果所有线程池线程都始终保持繁忙，但队列中包含挂起的工作，则线程池将在一段时间后创建另一个辅助线程但线程的数目永远不会超过最大值
- 超过最大值的线程可以排队，但他们要等到其他线程完成后才启动

### 使用线程池的原因

线程过多会带来调度开销，进而影响缓存局部性和整体性能。 而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。 这避免了在处理短时间任务时创建与销毁线程的代价。

线程池不仅能够保证内核的充分利用，还能防止过分调度。

可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。 例如，线程数一般取cpu数量+2比较合适，线程数过多会导致额外的线程切换开销。

### 组成部分

1. 线程池管理器：用于创建并管理线程池
2. 工作线程：线程池中线程
3. 任务接口：每个任务必须实现的接口，以供工作线程调度任务的执行
4. 任务队列：用于存放没有处理的任务，提供缓冲的能力

## grpool 源码解析

grpool 是轻量级 Goroutine 线程池。

客户端可以提交 job，调度程序接受 job，并将其发送给第一个可用的 worker 进程。

当 worker 完成处理作业时，将返回到 worker 池。

worker 数量和 job 队列大小是可配置的。

### 示例代码

以下是 github 上面推荐的最基本的 grpool 使用示例：

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

#### NewPool

首先看下 grpool.NewPool 的实现：

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

在这个函数里，描述了两个重要的对象：worker 和 job，这两个对象即线程池概念里的工作线程和任务接口。

#### worker

其中，worker 的数据结构如下所示：

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

#### Job

job 的结构如下所示：

```go
// 表示用户请求，该函数应该在某个 worker 中执行。
type Job func()
```

该 Job 即每个任务必须实现的接口，worker 的核心工作便是执行 Job。

#### pool

NewPool 里会创建 pool 对象，pool 数据结构如下：

```go
type Pool struct {
	JobQueue   chan Job
	dispatcher *dispatcher
	wg         sync.WaitGroup
}
```

pool 中有 3 个对象：

1. JobQueue：任务队列，该参数是包外可见的，用于接受用户传入的 Job 接口。
2. dispatcher：线程调度器，用于对 worker 和 job 进行分发。
3. wg：并发控制，能够控制 goroutine 的并发，可以避免 goroutine 还没执行完就退出程序，也能避免创建了过多的 goroutine。

#### dispatcher

dispatcher 的数据结构如下：

```go
// dispatcher负责从客户端接受job，同时等待第一个空闲的worker来分配job
type dispatcher struct {
	workerPool chan *worker
	jobQueue   chan Job
	stop       chan struct{}
}
```

dispatcher 的数据结构和 worker 类似，只不过 jobQueue 是包内可见的，pool 将包外可见的 JobQueue 接受进来给到 dispatcher，进而分发到各个 worker。

从客户端到 worker 执行 job 的流程图如下所示:

![go-grpool-grpool流程图.png](https://cnymw.github.io/GolangStudy/docs/img/go-grpool-grpool流程图.png)

#### newDispatcher

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