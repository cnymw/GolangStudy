# Go 源码解读-垃圾回收

> 源码地址：runtime/mgc.go
>
> 源码版本：1.17.6

## Golang 的垃圾回收 GC 介绍

Golang GC 与 mutator 线程并发运行，允许多个 GC 线程并行运行。使用了写屏障的并发标记和清理，是非分代和非压缩的。

> mutator 指的是用户线程，因为可能会改变内存的状态，所以命名为 mutator。

分配是使用每个 P 分割区域隔离的大小来完成的，以最大程度地减少碎片，同时消除常见情况下的锁定。

> P 是 Golang GMP 模型中的 Processor，是 Golang 的处理器但不是电脑 CPU，包含运行 Go 代码的必要资源，也有调度 Goroutine 的能力。

Golang GC 算法分为以下几个步骤，目前 Golang 所使用的 GC 实现是该算法的高级实现。

### 第一步：GC 执行清理终止

1、STW。这会使得所有的 P 达到 GC 安全点。 

> STW 即 Stop the world。当前运行的所有程序将被暂停。

2、清理任何未清理过的 span。只有在预期时间之前强制执行此 GC 周期时，才会有未清理的 span。GC 周期以 runtime.forcegcperiod 变量为准，默认 2 分钟。

> span 是 Golang 内存管理的基本单位，每个 span 管理指定规格（以 Golang 中的 page 为单位）的内存块，内存池分配出不同规格的内存块就是通过 span 体现出来的，应用程序创建对象就是通过找到对应规格的 span 来存储的。

### 第二步：GC 执行标记阶段

1、通过将 gcphase 设置为 _GCmark（来自于 _GCoff），启用写屏障，启用 mutator 辅助，并且将 GC root 标记为任务队列，为标记阶段作准备。在所有 P 都启用写屏障之前，不能清理任何对象，这是通过 STW 来实现的。 

> 可达性算法的原理是以一系列叫做 GC root 的对象为起点出发，引出它们指向的下一个节点，再以下个节点为起点，引出此节点指向的下一个节点，直到所有的节点都遍历完毕。
> 
> 哪些对象可以作为 GC root呢？
> 
> 1、全局变量：程序在编译期就能确定的那些存在于程序整个生命周期的变量。
> 
> 2、执行栈：每个 goroutine 都包含自己的执行栈，这些执行栈上包含栈上的变量及指向分配的堆内存区块的指针。
> 
> 3、寄存器：寄存器的值可能表示一个指针，参与计算的这些指针可能指向某些分配的堆内存区块。

2、Start the world。从这个节点开始，GC 工作由调度程序启动的标记 worker 以及分配的一部分 assist 程序来完成。写屏障给任何指针的写入操作屏蔽了被写入的指针和新的指针值（可以参阅 mbarrier.go）。新分配的对象立即被标记为黑色。 

3、GC 执行 GC root 标记作业。这包括扫描所有堆栈，对所有全局变量进行着色，以及对在 off-heap（堆外）runtime 的任何堆指针进行着色。扫描堆栈会停止当前的 goroutine，隐藏在堆栈上找到的任何指针，扫描完成后，会恢复当前 goroutine。 

4、GC 清空灰色对象的工作队列，将每个灰色对象扫描成黑色，并对对象中找到的所有指针进行着色（这反过来可能会将这些指针添加到工作队列中，也就是会持续迭代，直到遍历完所有的指针）。

5、由于 GC 工作分布在本地缓存中，因此 GC 使用了一个分布式终止算法来检测何时不再有 GC root 标记作业或灰色对象（参阅 gcMarkDone）。此时，GC 过渡到标记终止。

### 第三步：GC 执行标记终止

1、STW。

2、将 gcphase 设置为 _GCmarktermination，并禁用标记 worker 和 assist 程序。

3、执行刷新 mcache 的清理工作。

> 在 GMP 模型中，会在每个 P 下都有一个 mcache 字段，用来表示内存信息。

### 第四步：GC 执行清理阶段

1、将 gcphase 设置为 _GCoff，设置清理状态并禁用写屏障，来为清理阶段作准备。

2、Start the world。从这个时间点开始，新分配的对象是白色的，如果有必要的话，在使用新对象之前会分配清理 span。

3、GC 在后台执行并发清理并响应分配。

### 第五步：重复执行 GC

1、当内存分配达到 GC 条件时，重复上面从 1 开始的序列。

## 并发清理 Concurrent sweep

### 清理是如何并发进行的？

清理阶段与正常程序同时进行，也就是说清理不影响正常程序的运行。

堆内存被一个 span 接一个 span 的逐渐被清理（因为一个 goroutine 可能会依赖另一个 span），这个操作在一个后台 goroutine 中执行（这有助于不受 CPU 限制）。

在 STW 标记终止的结尾，所有的 span 都标记为"需要清理"。

### 清理的特点

后台清理线程 goroutine 逐个清理 span。

### 如何实现清理逻辑？

为了避免在还有未清理到的 span 时，请求更多的操作系统内存，当 goroutine 需要另一个 span 时，它首先尝试回收清理时产生的大量内存。

当一个 goroutine 需要分配一个新的小容量对象的 span，GC 会清理相同大小的小对象 span，除非 GC 释放至少一个对象。

当一个 goroutine 需要从堆中分配大对象 span 时，GC 会扫描 span，直到 GC 至少将那么多的 span 释放到堆中。

只有在这一种情况下可能不会清理 span：如果一个 goroutine 清理并释放两个不相邻的一页 span 到堆中，GC 将分配一个新的两页 span，但仍然可能存在其他的一页 span 未清理，这个 span 可能是被合并到两页 span 中的那个 span。

### 并发清理需要关注的问题

确保在未清理 span 上不进行任何操作至关重要（这会破坏 GC bitmap 中的 bits）。在 GC 期间，所有 mcache 都被刷新到中央缓存中，因此它们都是空的。

当一个 goroutine 将一个新的 span 抓取到 mcache 中时，GC 会清理它。

当一个 goroutine 显式释放一个对象或设置一个 finalizer 时，GC 会确保 span 被清理（通过清理，或者通过等待并发清理来实现）。

> finalizer 是与对象关联的一个函数，通过 runtime.SetFinalizer 来设置，它在对象被 GC 的时候，这个 finalizer 会被调用，以完成对象生命中最后一程。
> 
> 由于 finalizer 的存在，导致了对象在三色标记中，不可能被标为白色对象，所以，这个对象的生命也会得以延续一个 GC 周期。正如 defer 一样，我们也可以通过 finalizer 完成一些类似于资源释放的操作。

finalizer goroutine 仅在清理所有的 span 时才启动。当下一次 GC 开始时，它会清理所有未清理的 span。

## GC rate

当所分配的堆大小达到一定比例（由控制器计算的触发堆的大小）时，将会触发 GC，具体的公式是已经使用的内存量和额外使用内存量的比例，比例由 GOGC 环境变量控制（默认为 100）。

如果 GOGC=100 并且我们正在使用 4M，我们将达到 8M 时再次进行 GC（该标记在 gcController.heapGoal 变量中进行跟踪）。

这可以保持 GC 的成本和分配内存的成本保持线性比例。调整 GOGC 只会改变线性常数（以及使用的额外内存量）。

## 如何清理大型对象

为了防止在清理大型对象时出现长时间的停顿，同时为了提高并行性，GC 将大于 maxObletBytes 的对象的清理工作分解为最多为 maxObletBytes 的 oblets。

当清理遇到一个大对象的开头时，它只清理第一个 oblet 并将剩余的 oblet 作为新的清理作业排入队列。

## GC 源码解读

```go
// GC 运行一个垃圾回收线程并阻塞调用者，直到垃圾回收完成。
// GC 也有可能阻塞整个程序。
func GC() {
	// GC 的一个周期是：
	// 1、清理终止
	// 2、标记
	// 3、标记终止
	// 4、清理
	// 从开始到结束，在完成整个 GC 周期之前，该 GC 函数都不应返回。
	// 因此，程序总是想结束当前的 GC 周期，并开始新的 GC 周期。
	// 这意味着：
	//
	// 1. 在 GC 周期 N 的清理终止，标记或标记终止操作中，会持续等待直到标记终止 N 完成并转换到清理 N。
	//
	// 2. 在清理 N 中，GC 会协助清理 N。此时，我们又可以开始一个完整的 GC 周期 N+1。
	//
	// 3. 通过启动清理终止 N+1 来触发周期 N+1。
	//
	// 4. 等待标记终止 N+1 来完成整个 GC 周期 N+1。
	//
	// 5. GC 会协助清理 N+1 直到清理结束。
	//
	// 上述这些逻辑都必须实现，来解决 GC 可能会自行向前推进的逻辑。
	// 例如，当我们阻塞在标记终止 N 时，我们可能会在 GC 周期 N+2 中醒来（结束阻塞）。

	// 等待当前清理终止，标记和标记终止完成。
	n := atomic.Load(&work.cycles)
	gcWaitOnMark(n)

	// GC 目前处于清理 N 或者后续操作。
	// 将要触发 GC 周期 N+1，如果有必要的话，它将首先完成清理 N，然后进入扫描终止 N+1。
	gcStart(gcTrigger{kind: gcTriggerCycle, n: n + 1})

	// 等待标记终止 N+1 完成。
	gcWaitOnMark(n + 1)

	// 在返回结果之前先完成清理 N+1。
	// 这样做既是为了完成循环周期，也是因为 runtime.GC() 经常用作测试和 benchmarks 测试的一部分，
	// 所以这样做是为了让系统进入相对稳定和独立的状态。
	for atomic.Load(&work.cycles) == n+1 && sweepone() != ^uintptr(0) {
		sweep.nbgsweep++
		Gosched()
	}

	// 调用方可能会假设堆配置文件显示当前刚刚完成了一次 GC 周期，
	// （发生这种情况的原因是这是一个 STG GC）
	// 但是现在配置文件仍然显示当前是标记终止 N，而不是标记终止 N+1。
	//
	// As soon as all of the sweep frees from cycle N+1 are done,
	// we can go ahead and publish the heap profile.
	//
	// First, wait for sweeping to finish. (We know there are no
	// more spans on the sweep queue, but we may be concurrently
	// sweeping spans, so we have to wait.)
	for atomic.Load(&work.cycles) == n+1 && !isSweepDone() {
		Gosched()
	}

	// Now we're really done with sweeping, so we can publish the
	// stable heap profile. Only do this if we haven't already hit
	// another mark termination.
	mp := acquirem()
	cycle := atomic.Load(&work.cycles)
	if cycle == n+1 || (gcphase == _GCmark && cycle == n+2) {
		mProf_PostSweep()
	}
	releasem(mp)
}
```

## 参考

- [Go 语言设计与实现 — 垃圾收集器](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)
