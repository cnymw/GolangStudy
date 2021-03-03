# Go 源码解读 同步模块 sync

## sync 包概览

sync 包提供基本的同步原语，如互斥锁。除了 Once 和 WaitGroup 类型之外，大多数类型都是供比较简单的多线程场景。

更高级别的同步最好通过 channel 和通信来实现。

需要注意的是，sync 包里定义的类型的值不允许被拷贝。

### Mutex

```go
// Mutex 是互斥锁。
// 互斥锁的零值是一个未加锁的 mutex。
//
// 第一次使用之后，不能复制互斥锁。
type Mutex struct {
	state int32
	sema  uint32
}
```

Mutex 的接口是 Locker。

```go
// Locker 表示可以加锁和解锁的对象。
type Locker interface {
	Lock()
	Unlock()
}
```

下面将详细描述 Mutex 是如何实现加锁/解锁功能。

```go
// Lock 方法给 m 加锁。
// 如果锁已经在使用中，那么调用的线程 goroutine 将会阻塞，直到 mutex 可用。
func (m *Mutex) Lock() {
	// Fast path 快速路径：获取解锁的互斥锁。
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path 慢速路径(简略操作以便快速路径可以内联)。
	m.lockSlow()
}
```

> 在Linux中，有两种方法来处理传入TCP数据段：快速路径（Fast Path）和慢速路径（Slow Path）。
> 使用快速路径只进行最少的处理，如处理数据段、发生ACK、存储时间戳等。
> 使用慢速路径可以处理乱序数据段、PAWS、socket内存管理和紧急数据等。

在 Lock 方法中，使用了两种方式来确保能够给 m 加锁（只不过一个操作快，一个操作慢而已）。

如果 Fast path 加锁成功了，那么便不会执行 Slow path 的方法。

在这里，为了更快的运行 Fast path，采用了内联的方式，也就是把逻辑直接在 Lock 方法里实现。

而 Slow path 则把实现封装到了`m.lockSlow()`里，这样在编译的时候会把子方法的代码加载到`Lock()`方法里，速度也会受到影响。

### CompareAndSwapInt32
```go
// CompareAndSwapInt32 对 int32 值执行比较和交换操作。
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
```

CompareAndSwapInt32 原子操作由底层支持，所以执行速度非常快。

如果原子操作失败了，那会执行`lockSlow()`方法进行自旋抢锁，直到抢到锁为止。

### mutex 解决竞争问题

当 go 的 build 和 run 命令使用选项`-race`也就是启用数据竞争检测时，`race.Enabled=true`，如果多个线程抢占同一个变量，便会引发竞态报警。

下面编写测试文件 mutex.go 来模拟`Lock()`方法的竞态检测。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)
var m sync.Mutex

func main() {
	var x int
	go func() {
		for {
			m.Lock()
			x = 1
			fmt.Println(x)
			m.Unlock()
		}
	}()

	go func() {
		for {
			m.Lock()
			x = 2
			fmt.Println(x)
			m.Unlock()
		}
	}()

	go func() {
		for {
			m.Lock()
			x = 3
			fmt.Println(x)
			m.Unlock()
		}
	}()

	time.Sleep(10 * time.Second)
}
```

当使用命令`go run -race mutex.go`开启竞态检测之后，发现并发调用不会触发`Lock()`方法竞争，也不会触发变量`x`的数据竞争。

当去掉 mutex 加锁，解锁代码后，如下代码运行，则会报 warning 错误。

```go
package main

import (
	"time"
)

func main() {
	var x int
	go func() {
		for {
			x = 1
		}
	}()

	go func() {
		for {
			x = 2
		}
	}()

	go func() {
		for {
			x = 3
		}
	}()

	time.Sleep(10 * time.Second)
}
```

竞态报错如下所示：

```go
1
==================
WARNING: DATA RACE
Write at 0x00c000136008 by goroutine 8:
  main.main.func2()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:23 +0x3c

Previous write at 0x00c000136008 by goroutine 7:
  main.main.func1()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:15 +0x3c

Goroutine 8 (running) created at:
  main.main()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:21 +0x9c

Goroutine 7 (running) created at:
  main.main()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:13 +0x7a
==================
==================
WARNING: DATA RACE
Write at 0x00c000136008 by goroutine 9:
  main.main.func3()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:31 +0x3c

Previous write at 0x00c000136008 by goroutine 7:
  main.main.func1()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:15 +0x3c

Goroutine 9 (running) created at:
  main.main()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:29 +0xbe

Goroutine 7 (running) created at:
  main.main()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/sdk/sync/mutex.go:13 +0x7a
==================
2
3

```

从上可以看出，mutex 加锁是去除竞争的有效手段。

### Mutex fairness 互斥公平性

在看`lockSlow()`方法之前，需要先了解一下 mutex 里的一些常量以及互斥的两种模式。

```go
const (
    // 表示 mutex 是上锁状态
	mutexLocked = 1 << iota
    // 表示 mutex 是唤醒状态
	mutexWoken 
	// 表示 mutex 处于饥饿模式
	mutexStarving
	// 表示 mutex.state 右移 3 位后即为等待的 goroutine 的数量。 
	mutexWaiterShift = iota
	
    // 互斥公平性。
    // 
    // Mutex 可以有 2 种操作模式：正常模式，饥饿模式。
    // 在正常模式，waiters 是按先进先出的顺序排队，但是一个唤醒状态的 waiters 不拥有互斥锁，
    // 并与新到达的 goroutines 竞争所有权。
    // 新到达的 goroutines 有一个优势 -- 他们已经在 CPU 上运行，而且可能有很多这样的 goroutine，
    // 所以一个唤醒状态的 waiter 很有可能会输。
    // 在这种情况下，它排在等待队列的前面。
    // 如果 waiter 在超过 1ms 的时间内无法获取互斥，它会将互斥切换到饥饿模式。
    //
    // 在饥饿模式下，mutex 的所有权直接从解锁的 goroutine 传递给队列前面的 waiter。
    // 新到达的 goroutines 不会尝试获取互斥锁，即使它看起来是解锁的，也不会尝试自旋·。
    // 相反，他们在等待队列的尾部排队。
    // 
    // 如果 waiter 接收到 mutex 的所有权，并看到
    // （1）它是队列中的最后一个 waiter，
    // 或者（2）它等待的时间不到 1ms，
    // 它将会互斥对象切换回到正常操作模式。
    //
    // 正常模式具有相当好的性能，因为 goroutine 可以连续多次获取 mutex，即使有阻塞的 waiter。
    // 饥饿模式是预防高并发系统中尾延迟（tail latency）的重要方法。
    //
	// starvationThresholdNs 值为 1000000 纳秒，即 1ms，表示将 mutex 切换到饥饿模式的等待时间阈值。
	starvationThresholdNs = 1e6
)
```

### lockSlow

接下来，来看下`lockSlow()`的实现。

```go
func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false
	awoke := false
	iter := 0
	old := m.state
	for {
		// 不要在饥饿模式下自旋，所有权交给 waiters。
		// 这样我们无论如何都无法获得 mutex。
		// if m.state = 1，then（old&(mutexLocked|mutexStarving) == mutexLocked） = true
		// if m.state = 0，then（old&(mutexLocked|mutexStarving) == mutexLocked） = false
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			// 活跃的自旋是有效的。
			// 也就是说，如果一个线程抢到了 mutex 后，再次抢该 mutex，也是可以的。
			// 尝试设置 mutexWoken 标志以通知 Unlock 不要唤醒其他被阻塞的 goroutines。
			// 在这里 old = m.state = 1
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				// 重入标记
				awoke = true
			}
			runtime_doSpin()
			iter++
			old = m.state
			continue
		}
		new := old
		// 不要试图获取饥饿的 mutex，新到达的 goroutines 必须排队。
		if old&mutexStarving == 0 {
			new |= mutexLocked
		}
		if old&(mutexLocked|mutexStarving) != 0 {
			new += 1 << mutexWaiterShift
		}
		
		// 当前 goroutine 将 mutex 切换到饥饿模式。
		// 但是如果 mutex 当前处于解锁状态，则不要进行切换。
		// Unlock 期望饥饿的 mutex 拥有 waiters，这在这种情形下不能实现。
		if starving && old&mutexLocked != 0 {
			new |= mutexStarving
		}
		if awoke {
			// goroutine 已经从睡眠中唤醒，所以我们需要在任何情况下重置标志。
			if new&mutexWoken == 0 {
				throw("sync: inconsistent mutex state")
			}
			new &^= mutexWoken
		}
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			if old&(mutexLocked|mutexStarving) == 0 {
				// 使用 CAS 原子操作给 mutex 加锁
				break 
			}
			// 如果我们之前已经在等了，就在队伍前面排队。
			queueLifo := waitStartTime != 0
			if waitStartTime == 0 {
				waitStartTime = runtime_nanotime()
			}
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state
			if old&mutexStarving != 0 {
				// 如果这个 goroutine 被唤醒并且 mutex 处于饥饿模式，
				// 那么所有权会被移交给我们，但 mutex 处于某种不一致的状态：
				// mutexLocked 未设置，并且我们仍被视为 waiter。得修复好它。
				if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
					throw("sync: inconsistent mutex state")
				}
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
				if !starving || old>>mutexWaiterShift == 1 {
					// 退出饥饿模式。
					// 这个操作很关键，并考虑了等待时间。
					// 饥饿模式是如此的低效，以至于两个 goroutines 一旦将 mutex 切换到饥饿模式，
					// 就可以无限地进入锁步（lock-step）。
					delta -= mutexStarving
				}
				atomic.AddInt32(&m.state, delta)
				break
			}
			awoke = true
			iter = 0
		} else {
			old = m.state
		}
	}

	if race.Enabled {
		race.Acquire(unsafe.Pointer(m))
	}
}
```

for 循环中`runtime_doSpin()`这段代码的目的是，尝试通过自旋方式获取锁资源，自旋是可以避免 goroutine 切换，但是消耗的资源更多。

```go
//go:linkname sync_runtime_doSpin sync.runtime_doSpin
//go:nosplit
func sync_runtime_doSpin() {
	procyield(active_spin_cnt)
}
```

`runtime_doSpin()`和`sync_runtime_doSpin()`链接，该方法会在 CPU 上执行若干次 PAUSE 指令，什么功能也没有，但是会占用 CPU 资源。