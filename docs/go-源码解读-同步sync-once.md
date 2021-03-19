# Go 源码解读 同步模块 sync once

Once 是仅仅执行一次操作的对象，如果业务中存在某些操作仅仅只能执行一次，可以用到 once。

一个简单的例子如下所示：

```go
var once sync.Once

func main() {
	for index := 0; index < 10; index++ {
		once.Do(func() {
			fmt.Println("once")
		})
		fmt.Printf("loop time:%d\n", index)
	}
}

```

可以看到，无论循环多少次，`once.Do`也只会执行一次。

## 数据结构

```go
type Once struct {
	// 参数 done 指示操作是否已经完成。
	// 它是 struct 的第一个参数是因为它用在 hot path 里。
	// 在每一个调用点（call site），hot path 是内联的。
	// 把 done 参数放在第一位允许某些体系结构（amd64/x86）上使用更紧凑的指令，
	// 而在其他体系结构上使用更少的指令（用于计算偏移量）。
	done uint32
	m    Mutex
}
```

Once 使用 done 参数来标记操作是否被执行过了，Once 的核心业务是围绕这个参数来展开的。

## Do

```go
// 当且仅当该实例第一次调用 Do 时，Do 才会调用函数 f。
// 换句话说，给定了
//  var once Once
// 即使每次调用中的 f 的值不同，即使被调用多次，也只有第一次调用才会调用 f。
// 每次要执行函数 f 都需要一个 Once 的新实例。
// 
// Do 用于必须只运行一次的初始化。
// 因为 f 是没有参数的，所以可能需要使用 function 语法来获取 Do 调用的函数的参数：
//  config.once.Do(func() { config.init(filename) })
//
// 因为只有在函数 f 返回时，才会返回 Do 的调用，所以如果函数 f 导致 Do 被调用，那将会死锁。
// 
// 如果函数 f panic 了，Do 认为它已经返回，未来调用 Do 会直接返回而不调用 f。
func (o *Once) Do(f func()) {
	// 注意：以下是 Do 的一个不正确的实现：
	//
	//  if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
	//      f()
	//  }
	// 
	// Do 能够保证当它返回时，f 已经完成了。
	// 以上的实现不能实现这个保证：
	// 给定两个同时调用，cas 的胜者会调用 f，
	// 第二个将立即返回，而不必等待第一个对 f 的调用完成。
	// 这就是为什么 slow path 会退回到互斥锁，
	// 以及原子操作 atomic.StoreUint32 必须推迟到 f 返回之后。
	if atomic.LoadUint32(&o.done) == 0 {
		// 外联 slow-path 来让 fast-path 进行内联。
		o.doSlow(f)
	}
}
```

在 Do 中，不能在 fast-path 里判断 done 是否等于 1 来快速返回，这样会导致在并发状态下有一个协程会在 f 没有执行或没有执行完就返回了。

合理的做法，是在 slow-path 里对 done 是否等于 1 和 f 是否执行完成做双重校验。

## doSlow

```go
func (o *Once) doSlow(f func()) {
	// 同一时间只允许一个协程执行 doSlow
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

互斥锁 m 限制同时只能存在一个协程在 slow-path 里运行，这个操作的主要目的是等待获取到锁的协程执行完成，也是等待函数 f 的执行完成。

互斥锁 m 解决了同一时间只能一个协程执行 slow-path 问题。

`defer atomic.StoreUint32(&o.done, 1)`解决了在 f 执行完成后，才允许其他协程返回的问题。

可以执行以下代码来验证效果：

```go
var once sync.Once

func main() {
	for index := 0; index < 10; index++ {
		go func() {
			once.Do(func() {
				fmt.Println("begin to wait")
				time.Sleep(5 * time.Second)
				fmt.Println("once")
			})
		}()
		fmt.Printf("time:%d\n", index)
	}
	time.Sleep(10 * time.Second)
}
/*
output:
time:0
time:1
time:2
time:3
time:4
time:5
time:6
time:7
time:8
time:9
begin to wait
once
 */
```

可以从输出看出：

1. 创建了 10 个协程，只有一个协程执行了`fmt.Println`
2. 立马输出一次"begin to wait"，但在执行`fmt.Println("once")`前等待了 5 秒，说明有一个协程在持续获取互斥锁，其他的协程在持续等待中，直到函数 f 执行结束。

