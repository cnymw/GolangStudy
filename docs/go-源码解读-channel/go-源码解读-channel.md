# Go 源码解读-channel

> 源码地址：runtime/chan.go
> 
> 源码版本：1.17.6

## 数据结构 hchan

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
	recvq    waitq  // 接收 waiters 列表
	sendq    waitq  // 发送 waiters 列表

	// lock 保护了 hchan 中的所有字段，
	// 以及在这个 channel 里面阻塞的几个 sudog 中的字段
	//
	// 不要在持有这个锁的时候改变另一个 G 的状态
	// （特别是不要使 G 状态变成 ready）
	// 因为这会在堆栈缩容 shrinking 的时候死锁
	lock mutex
}
```

waiters 列表指代的是等待该 channel 的 goroutine 列表，可以是等待发送/接收 channel。

## 数据结构 waitq

```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

recvq 和 sendq 是等待队列，waitq 是一个双向链表。

sudog 是链表结构，所以 waitq 只需要记录 first 和 last 指针，即可获取 sudog 整个链表。

## 数据结构 sudog

> 源码地址：runtime/runtime2.go

```go
// sudog 表示等待列表中的 g，例如用于在 channel 上发送/接收的 g。
//
// sudog 对象是有必要的，因为 g 和同步对象关系是多对多的。
// 一个 g 可以在许多 wait 列表上面，因此一个 g 可能有很多 sudog；
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
	waitlink *sudog // g.waiting 列表或者 semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```

semaRoot 是用于 sync.Mutex 的异步信号量，拥有一个具有不同地址的 sudog 平衡树，semaRoot 实现在：go/src/runtime/sema.go

## 数据结构 chantype

> 源码地址：runtime/type.go

在初始化函数中，有一个核心参数 chantype 记录了 channel 里的元素类型 ，该参数定义如下：

```go
type chantype struct {
	typ  _type // chantype 也是一种类型，所以需要 _type 来记录 chantype 的类型（经典套娃）
	elem *_type // elem 记录了 channel 里的元素类型
	dir  uintptr // dir 记录了 channel 的方向 direction
}
```

channel 是具有传输方向的，如果不指定传输方向的话，channel 是双向的。

有两种定义传输方向的语法：

```go
out chan<- int
in <-chan int
```

类型 chan<- int 表示一个只发送 int 的 channel，只能发送不能接收。

类型 <-chan int 表示一个只接收 int 的 channel，只能接收不能发送。

## 初始化函数 makechan

当使用 make 函数创建 channel 时，会调用 makechan 来实现初始化逻辑，例如执行以下代码，底层会调用 makechan：

```go
// 初始化无 buffer 的 channel
c := make(chan int)
// 初始化有 buffer 的 channel
c := make(chan int , 10)
```

makechan 的实现如下：

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

	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	// 当 buf 中存储的元素不包含指针时，hchan 不包含 GC 感兴趣的指针。
	// buf 指向相同的内存分配，elemtype 是永久的。
	// sudog 被它们所属的线程所引用，因此它们无法被 GC。
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

## 数据发送函数 send

先看下 channel send 的几种场景，以及对应的底层实现。

首先是 c <- x 场景：

```go
// 编译代码中 c<-x 的入口点
//go:nosplit
func chansend1(c *hchan, elem unsafe.Pointer) {
	chansend(c, elem, true, getcallerpc())
}
```

> go:nosplit 指令是用于指定文件中声明的下一个函数不得包含堆栈溢出检查（简单来讲，就是这个函数跳过堆栈溢出的检查。）。在不安全地抢占调用 goroutine 的时间调用的低级运行时源最常使用此方法。

接下来是 select 场景：

```go
// 编译期实现如下：
//
//	select {
//	case c <- v:
//		... foo
//	default:
//		... bar
//	}
//
// 编译期编译为：
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

## 参考

- [深入 Go 并发原语 — Channel 底层实现](https://halfrost.com/go_channel/)
- [深入理解Go语言(01): interface源码分析](https://www.cnblogs.com/jiujuan/p/12653806.html)
- [在 Go 中恰到好处的内存对齐](https://eddycjy.gitbook.io/golang/di-1-ke-za-tan/go-memory-align)
- [golang chan 最详细原理剖析](https://zhuanlan.zhihu.com/p/299592156)
