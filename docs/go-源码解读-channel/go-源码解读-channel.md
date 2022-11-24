# Go 源码解读-channel

> 源码地址：runtime/chan.go
> 
> 源码版本：1.17.6

## 数据结构 hchan

![go-源码解读-channel-hchan.png](https://cnymw.github.io/GolangStudy/docs/go-源码解读-channel/go-源码解读-channel-hchan.png)

```go
type hchan struct {
	qcount   uint           // 队列中总共的数据量
	dataqsiz uint           // 循环队列的数据量
	buf      unsafe.Pointer // 指向 dataqsiz 循环队列元素数组的指针
	elemsize uint16
	closed   uint32
	elemtype *_type // 元素类型
	sendx    uint   // 发送索引 index
	recvx    uint   // 接收索引 index
	recvq    waitq  // 接收 waiters 队列
	sendq    waitq  // 发送 waiters 队列

	// lock 保护了 hchan 中的所有字段，
	// 以及在这个 channel 里面阻塞的几个 sudog 中的字段。
	//
	// 不要在持有这个锁的时候改变另一个 G 的状态
	// （特别是不要使 G 状态变成 ready）
	// 因为这会在堆栈缩容 shrinking 的时候死锁
	lock mutex
}
```

`waiters` 队列指代的是等待该 `channel` 的 `goroutine` 队列，可以是等待发送/接收 `channel`。

## 数据结构 waitq

```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

`recvq` 和 `sendq` 是等待队列，`waitq` 是一个双向链表。

`sudog` 是链表结构，所以 `waitq` 只需要记录 `first` 和 `last` 指针，即可获取 `sudog` 整个链表。

## 数据结构 sudog

> 源码地址：runtime/runtime2.go

channel 最核心的数据结构是 sudog。

```go
// sudog 表示等待队列中的 g，例如用于在 channel 上发送/接收的 g。
//
// sudog 对象是非常重要的，因为 g 和同步对象关系是多对多的。
// 一个 g 可以在许多 wait 队列上面，因此一个 g 可能有很多 sudog；
// 并且许多 g 可能正在等待同一个同步对象，因此一个对象可能有许多 sudog。
//
// sudog 是从一个特殊的池 pool 中分配的。
// 使用 acquireSudog 和 releaseSudog 来分配和释放它们。
type sudog struct {
	// 以下所有字段受该 sudog 阻塞的 channel 的 hchan.lock 所保护。
	// 对于参与 channel 操作的 sudog，shrinkstack 依赖这些字段。

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // 数据元素(可能指向一个栈 stack)

	// 以下字段永远不会同时被访问。
	// 对于 channal，waitlink 只能被 g 访问。
	// 对于信号量，所有字段（包括上面的字段），只有在持有 semaRoot 锁时才能访问。

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect 表示 g 正在参与 select，
	// 因此 g.selectDone 必须经过 CAS 处理才能赢得唤醒的竞争 race。
	isSelect bool

	// success 表示通过 channel c 的通信是否成功。
	// 如果 goroutine 因为通过 channel c 传递了一个值而被唤醒，则 success 为 true。
	// 如果因为 channel c 已关闭而唤醒，则 success 为 false。
	success bool

	parent   *sudog // semaRoot 二叉树
	waitlink *sudog // g.waiting 队列或者 semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```

`semaRoot` 是用于 `sync.Mutex` 的异步信号量，拥有一个具有不同地址的 `sudog` 平衡树，`semaRoot` 实现在：go/src/runtime/sema.go

## 数据结构 chantype

> 源码地址：runtime/type.go

在初始化函数中，有一个核心参数 `chantype` 记录了 `channel` 里的元素类型 ，该参数定义如下：

```go
type chantype struct {
	typ  _type // chantype 也是一种类型，所以需要 _type 来记录 chantype 的类型（经典套娃）
	elem *_type // elem 记录了 channel 里的元素类型
	dir  uintptr // dir 记录了 channel 的方向 direction
}
```

`channel` 是具有传输方向的，如果不指定传输方向的话，`channel` 是双向的。

有两种定义传输方向的语法：

```go
out chan<- int
in <-chan int
```

类型 `chan<- int` 表示一个只发送 int 的 channel，只能发送不能接收。

类型 `<-chan int` 表示一个只接收 int 的 channel，只能接收不能发送。

## 初始化函数 makechan

当使用 `make` 函数创建 `channel` 时，会调用 `makechan` 来实现初始化逻辑，例如执行以下代码，底层会调用 `makechan`：

```go
// 初始化无 buffer 的 channel
c := make(chan int)
// 初始化有 buffer 的 channel
c := make(chan int , 10)
```

`makechan` 的实现如下：

```go
// chantype 是 channel 传递的元素类型，可以是基础类型，也可以是对象
func makechan(t *chantype, size int) *hchan {
	elem := t.elem

	// elem.size 是类型 type 占用的空间大小
	// 类型的大小不会超过 16 位
	if elem.size >= 1<<16 {
		throw("makechan: invalid channel element type")
	}
	// 内存对齐
	if hchanSize%maxAlign != 0 || elem.align > maxAlign {
		throw("makechan: bad alignment")
	}
    // 缓冲区大小检查，判断是否溢出
	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// 当 buf 中存储的元素不包含指针时，hchan 不包含 GC 感兴趣的指针（也就是说 GC 不会回收此时的 hchan）。
	// buf 指向相同的元素类型的内存，elemtype 是固定不变的。
	// sudog 被它们所属的线程所引用，因此它们无法被 GC。
	// 受到垃圾回收器的限制，指针类型的缓冲 buf 需要单独分配内存。
	// 所以官方在这里加了一个 TODO，垃圾回收的时候这段代码逻辑需要重新考虑。
	// TODO(dvyukov,rlh): Rethink when collector can move allocated objects.
	var c *hchan
	switch {
	case mem == 0:
		// 如果队列或者元素大小是 0
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// 竞态检测使用这个内存地址来进行同步
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
		// 如果 channel 元素中不包含指针
		// 在一个调用中分配 hchan 和 buf
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// 如果元素包含指针
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}
    // 赋值元素大小
	c.elemsize = uint16(elem.size)
    // 赋值元素类型
	c.elemtype = elem
    // 赋值元素队列大小 size
	c.dataqsiz = uint(size)
	lockInit(&c.lock, lockRankHchan)
	
	......
	return c
}
```

上面这段 makechan() 代码主要目的是生成 *hchan 对象。重点关注 switch-case 中的 3 种情况：

1. 当队列或者元素大小为 0 时，调用 mallocgc() 在堆上为 channel 开辟一段大小为 hchanSize 的内存空间。
2. 当元素类型不是指针类型时，调用 mallocgc() 在堆上开辟为 channel 和底层 buf 缓冲区数组开辟一段大小为 hchanSize + mem 连续的内存空间。
3. 默认情况元素类型中有指针类型，调用 mallocgc() 在堆上分别为 channel 和 buf 缓冲区分配内存。

> 就是因为 channel 的创建全部调用的 mallocgc()，在堆上开辟的内存空间，channel 本身会被 GC 自动回收。
> 
> 正是有了这一性质，所以才有了下文关闭 channel 中优雅关闭的方法。

## 数据发送

`channel` `send` 会有以下几种场景，每种场景有不同的实现，但是底层实现都是 `chansend`。

### 发送数据 c<-x 场景实现：chansend1

首先是向 `channel` 发送数据时 `c <- x` 场景，当编译器遇到这种场景，会将代码编译成如下代码：

```go
// 编写代码：
// c <- x
// 
// 编译器编译为：
// chansend1(c,x)
//go:nosplit
func chansend1(c *hchan, elem unsafe.Pointer) {
	chansend(c, elem, true, getcallerpc())
}
```

> go:nosplit 指令是用于指定文件中声明的下一个函数不得包含堆栈溢出检查（简单来讲，就是这个函数跳过堆栈溢出的检查。）。在不安全地抢占调用 goroutine 的时间调用的低级运行时源最常使用此方法。

### 多路复用 select 场景实现：selectnbsend

接下来是多路复用 `select` 场景实现：

```go
// 编写代码如下：
//
//	select {
//	case c <- v:
//		... foo
//	default:
//		... bar
//	}
//
// 编译器会编译为：
//
//	if selectnbsend(c, v) {
//		... foo
//	} else {
//		... bar
//	}
//
func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
	return chansend(c, elem, false, getcallerpc())
}
```

### 底层实现 chansend 步骤 1：异常检查

接下来，我们来分步骤来看下 `chansend` 的实现，首先是前置的异常检查：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
	// 步骤 1：异常检查
	// 当 channel 还未初始化或为 nil 时，向其中发送数据将会永久阻塞
	if c == nil {
		// 如果此时未阻塞，那么直接返回 false
		if !block {
			return false
		}
        // gopark 会使当前 goroutine 休眠，并通过 unlockf 唤醒，但是此时传入的 unlockf 为 nil, 因此，goroutine 会一直休眠
		gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	......

	// 竞态检测
	if raceenabled {
		racereadpc(c.raceaddr(), callerpc, funcPC(chansend))
	}

	// Fast path 快速路径: 在未获取锁的情况下检查失败的非阻塞操作。
	//
	// 在 observe 到 channel 未关闭（c.closed == 0）后，我们 observe 到 channel 尚未准备好 send（于是 return false）。
	// 在每一个 observe 中，都是对单个变量的读取（第一个 c.closed 和第二个 full()）。
	// 即使两次 observe 之间 channel 是关闭的，一个关闭的 channel 也不能从"准备好 send"转换为"未准备好 send"，
	// 也就意味着，在两次 observe 之间的某个时刻，channel 尚未关闭，也未准备 send。
	// 我们表现的行为就像，我们在那个时刻，observe 到了 channel，并报告无法继续 send。
	//
	// 如果读取数据可以指令重排的话，那这一切就很合理了：
	// 如果我们 observe 到 channel 没有准备好 send，然后 observe 到 channel 没有关闭，
	// 这意味着在第一次 observe 期间 channel 没有关闭。
	// 于是，没有任何理由可以让逻辑往后面继续执行。
	// 我们依赖 chanrecv() 和 closechan() 中锁的释放所带来的副作用来更新这个线程的 c.closed 和 full()。
	if !block && c.closed == 0 && full(c) {
		return false
	}
    
	......
	// 步骤 2：同步发送
	// 步骤 3：异步发送
	// 步骤 4：阻塞发送
}
```

> gopark 函数做的主要事情分为两点：
>
> 1、解除当前 goroutine 的 m 的绑定关系，将当前 goroutine 状态机切换为等待状态；
>
> 2、调用一次 schedule() 函数，在局部调度器P发起一轮新的调度。

---

> 竞态检测
>
> go 的 build 和 run 命令支持选项 -race。如果启用该选项，发现存在数据竞态就会报警。
>
> -race 在源码中对应的变量是 raceenabled，当启用 -race， raceenabled 就是 true。

---

> Fast path 快速路径
>
> 在程序设计中，快速路径是指在一个程序中比起一般路径有更短指令路径长的路径。
>
> 有效的快速路径会在处理最常出现的的情形上比一般路径更有效率，让一般路径处理特殊情形、边角情形、错误处理与其它反常状况。快速路径是程序优化的一种形式。

---
### 底层实现 chansend 步骤 2：同步发送

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 步骤 1：异常检查
	......
	
	// 步骤 2：同步发送
	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	lock(&c.lock)

	if c.closed != 0 {
		unlock(&c.lock)
		panic(plainError("send on closed channel"))
	}

	if sg := c.recvq.dequeue(); sg != nil {
		// 获取到一个等待的 receiver。
		// 我们将需要发送的值直接传递给 receiver，绕过 channel 缓冲区（如果有的话）。
		send(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true
	}
    
	......
	// 步骤 3：异步发送
	// 步骤 4：阻塞发送
}
```

### 底层实现 chansend 步骤 3：异步发送

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 步骤 1：异常检查
	// 步骤 2：同步发送
	......
	
	// 步骤 3：异步发送
	if c.qcount < c.dataqsiz {
		// channel buffer 中有可用空间。对要发送的元素进行排队。
		qp := chanbuf(c, c.sendx)
		if raceenabled {
			racenotify(c, c.sendx, nil)
		}
		typedmemmove(c.elemtype, qp, ep)
		c.sendx++
		if c.sendx == c.dataqsiz {
			c.sendx = 0
		}
		c.qcount++
		unlock(&c.lock)
		return true
	}
    
	......
	// 步骤 4：阻塞发送
}
```

### 底层实现 chansend 步骤 4：阻塞发送

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 步骤 1：异常检查
	// 步骤 2：同步发送
	// 步骤 3：异步发送
	......
	
	// 步骤 4：阻塞发送
	if !block {
		unlock(&c.lock)
		return false
	}

	// channel 阻塞。某个 receiver 未来会继续完成后续操作。
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
    // No stack splits between assigning elem and enqueuing mysg
    // on gp.waiting where copystack can find it.
	mysg.elem = ep
	mysg.waitlink = nil
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.waiting = mysg
	gp.param = nil
	c.sendq.enqueue(mysg)
	// 向任何试图 shrink 堆栈的线程发出信号，我们将要在 channel 上 gopark。
	// G 的状态改变和我们设置 gp.activeStackChans 之间的窗口期，对于堆栈 shrink 不安全。
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)
	// 确保发送的值保持 alive，直到 receiver 将其复制出来。
	// sudog 有一个指向堆栈对象的指针，但 sudog 不被视为堆栈的 root。
	KeepAlive(ep)

	// 某个线程唤醒了我们。
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	closed := !mysg.success
	gp.param = nil
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	mysg.c = nil
	releaseSudog(mysg)
	if closed {
		if c.closed == 0 {
			throw("chansend: spurious wakeup")
		}
		panic(plainError("send on closed channel"))
	}
	return true
}
```

> stack shrink 栈收缩
> 
> 收缩栈是在 mgcmark.go 中触发的，主要是在 scanstack 和 markrootFreeGStacks 函数中，也就是垃圾回收的时候会根据情况收缩栈。

### chansend 调用函数：full

`chansend` fast path 用到函数 `full`：

```go
// full 函数返回 channel 上的 send 是否会阻塞（即 channel 是否已满）。
// full 通过读取可变状态的单个变量的值，来判断 channel 是否已满。
// 所以尽管返回的那个时刻为正确的返回，但在上层函数接收到返回值时，可能正确的答案已经改变。
func full(c *hchan) bool {
	// c.dataqsiz 是不可变的 (在 channel 被创建后，永久不会被写入新的值)
	// 因此在 channel 操作期间，任何时刻读取值都是安全的。
	if c.dataqsiz == 0 {
		// 假设读取指针的操作是 relaxed-atomic 操作的（可以理解为原子操作的一种）
		return c.recvq.first == nil
	}
	// 假设 uint 的读取是 relaxed-atomic 操作的（可以理解为原子操作的一种）
	return c.qcount == c.dataqsiz
}
```

> relaxed-atomic
> 
> 我们可以对单个变量进行具有屏障或者不具有屏障的原子操作。当屏障没有使用，只有原子性保证时，我们称之为"relaxed atomic"。

## 参考

- [深入 Go 并发原语 — Channel 底层实现](https://halfrost.com/go_channel/)
- [深入理解Go语言(01): interface源码分析](https://www.cnblogs.com/jiujuan/p/12653806.html)
- [在 Go 中恰到好处的内存对齐](https://eddycjy.gitbook.io/golang/di-1-ke-za-tan/go-memory-align)
- [golang chan 最详细原理剖析](https://zhuanlan.zhihu.com/p/299592156)
