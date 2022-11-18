# Go 源码解读-channel

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


## 参考

- [深入 Go 并发原语 — Channel 底层实现](https://halfrost.com/go_channel/)
