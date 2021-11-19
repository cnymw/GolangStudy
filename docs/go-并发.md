# Go 并发机制

## Goroutines

主函数返回时，所有的goroutine都会被直接打断，程序退出。

```go
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```

以下的代码不能保证 f()函数执行过或者执行完毕。

```go
func main() {
    go f()
}
```

## Channels

### channel

如果说 goroutine 是 Go 语言程序的并发体的话，那么 channels 则是它们之间的通信机制。

两个相同类型的 channel 可以使用==运算符比较。如果两个 channel 引用的是相同的对象，那么比较的结果为真。

Channel 还支持 close 操作，用于关闭 channel，随后对基于该 channel 的任何发送操作都将导致 panic 异常。

```go
close(ch)
```

对一个已经被 close 过的 channel 进行接收操作依然可以接受到之前已经成功发送的数据。

```go
func main(){
	ch := make(chan int)
	go f1(ch)
	go f2(ch)

	time.Sleep(3 * time.Second)
}
func f1(ch chan int){
	ch <- 1
	close(ch)
}
func f2(ch chan int){
	x := <- ch  // ok , x = 1
	fmt.Println(x)
}
```

### 不带缓存的 channel

调用 make 函数创建的是一个无缓存的 channel，但是我们也可以指定第二个整型参数，对应 channel 的容量。如果 channel 的容量大于零，那么该 channel 就是带缓存的 channel。

```go
ch = make(chan int)    // unbuffered channel
ch = make(chan int, 0) // unbuffered channel
ch = make(chan int, 3) // buffered channel with capacity 3
```

一个基于无缓存 Channels 的发送操作将导致发送者 goroutine 阻塞，直到另一个 goroutine 在相同的 Channels 上执行接收操作，当发送的值通过 Channels 成功传输之后，两个 goroutine 可以继续执行后面的语句。

反之，如果接收操作先发生，那么接收者 goroutine 也将阻塞，直到有另一个 goroutine 在相同的 Channels 上执行发送操作。

基于无缓存 Channels 的发送和接收操作将导致两个 goroutine 做一次同步操作。

### 带缓存的 Channels

带缓存的 Channel 内部持有一个元素队列。队列的最大容量是在调用 make 函数创建 channel 时通过第二个参数指定的。

```go
ch = make(chan string, 3)
```

我们可以在无阻塞的情况下连续向新创建的 channel 发送三个值,如果有第四个发送操作将发生阻塞。

```go
ch <- "A" // ok
ch <- "B" // ok
ch <- "C" // ok
ch <- "D" // blocked
```

在继续执行三次接受操作后 channel 内部的缓存队列将又成为空的，如果有第四个接收操作将发生阻塞。
```go
fmt.Println(<-ch) // "A"
fmt.Println(<-ch) // "B"
fmt.Println(<-ch) // "C"
fmt.Println(<-ch) // blocked
```

## select

如果多个 case 同时就绪时，select 会随机地选择一个执行，这样来保证每一个 channel 都有平等的被 select 的机会。

select 会有一个 default 来设置当其它的操作都不能够马上被处理时程序需要执行哪些逻辑。

下面的 select 语句会在 abort channel 中有值时，从其中接收值；无值时什么都不做。这是一个非阻塞的接收操作；反复地做这样的操作叫做“轮询 channel”。

```go
select {
case <-abort:
    fmt.Printf("Launch aborted!\n")
    return
default:
    // do nothing
}
```

# 同步

## sync.Mutex 互斥锁
### 通过 channel 实现互斥锁
我们可以用一个容量只有 1 的 channel 来保证最多只有一个 goroutine 在同一时刻访问一个共享变量。

一个只能为1和0的信号量叫做二元信号量(binary semaphore)。

```go
var (
    sema    = make(chan struct{}, 1) // a binary semaphore guarding balance
    balance int
)

func Deposit(amount int) {
    sema <- struct{}{} // acquire token
    balance = balance + amount
    <-sema // release token
}

func Balance() int {
    sema <- struct{}{} // acquire token
    b := balance
    <-sema // release token
    return b
}
```

### sync.Mutex
在 Lock 和 Unlock 之间的代码段中的内容 goroutine 可以随便读取或者修改，这个代码段叫做临界区。

每一个函数在一开始就获取互斥锁并在最后释放锁，从而保证共享变量不会被并发访问。这种函数、互斥锁和变量的编排叫作"监控 monitor"。
```go
import "sync"

var (
    mu      sync.Mutex // guards balance
    balance int
)

func Deposit(amount int) {
    mu.Lock()
    balance = balance + amount
    mu.Unlock()
}

func Balance() int {
    mu.Lock()
    b := balance
    mu.Unlock()
    return b
}
```

go 里没有重入锁(Re-entrant lock)，没法对一个已经锁上的 mutex 来再次上锁--这会导致程序死锁，没法继续执行下去。

- [what-is-the-re-entrant-lock-and-concept-in-general](https://stackoverflow.com/questions/1312259/what-is-the-re-entrant-lock-and-concept-in-general)

## sync.RWMutex读写锁
允许多个只读操作并行执行，但写操作会完全互斥。这种锁叫作“多读单写”锁(multiple readers, single writer lock)，Go 语言提供的这样的锁是 sync.RWMutex

```go
var mu sync.RWMutex
var balance int
func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
func Write() {
    mu.Lock() // writers lock
    defer mu.Unlock()
    balance = 1
}
```

RLock 只能在临界区共享变量没有任何写入操作时可用。

RWMutex 只有当获得锁的大部分 goroutine 都是读操作，而锁在竞争条件下，也就是说，goroutine 们必须等待才能获取到锁的时候，RWMutex 才是最能带来好处的。

RWMutex 需要更复杂的内部记录，所以会让它比一般的无竞争锁的 mutex 慢一些。

## WaitGroup

该类型有三个指针方法：Add，Done，Wait。

sync.WaitGroup 是一个结构体类型，其中有一个字段用四个字节表示给定计数，用四个字节表示等待计数。通过 Add 方法增大或减少给定计数：
```go
wg.Add(3)
wg.Add(-3)
```

如果让给定计数变为负数，会引发一个运行恐慌。

还可以通过调用 Done 方法使给定计数减一。

```go
wg.Done() // = wg.Add(-1)
```

当 wg.Done 让给定计数变为负数时，也会引发一个运行时恐慌。

当调用 Wait 方法时，会去检查给定计数，如果计数等于 0，那么该方法会立即返回，否则会阻塞，同时等待计数会加一。直到给定计数变为 0，才会唤醒所有阻塞的 goroutine，同时清零等待计数。

等待 goroutine 任务的完成有两种方法：

1.使用 channel

```go
sign := make(chan struct{}, 2)
go func(){
    // Do something 
    sign <- struct{}	
}()
go func(){ 
    // Do something
    sign <- struct{}
}()
// 阻塞，直到两个goroutine都结束执行
<-sign
<-sign
```

2.使用 sync.WaitGroup

```go
var wg sync.WaitGroup
wg.Add(2)

go func(){
    // Do something
    wg.Done()
}()

go func(){
    // Do something
    wg.Done()
}()

// 阻塞，直到 wg 的给定计数为 0，也就是两次 wg.Done() 都执行完毕
wg.Wait()
```