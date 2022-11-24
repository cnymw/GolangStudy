# Channel

## 什么是 channel

一个 channel 是一个通信机制，它可以让一个 goroutine 通过它给另一个 goroutine 发送值信息。

## 什么时候用到 channel

不要通过共享内存进行通信。相反，应该通过通信来共享内存。

Do not communicate by sharing memory; instead, share memory by communicating 

这是 Go 语言并发的哲学座右铭。

## golang 的并发原语

Go 中的并发原语主要分为 2 大类：

- sync 包：主要是 WaitGroup，互斥锁和读写锁，cond，once，sync.Pool
- channel

在 2 种情况下推荐使用 sync 包：

1. 对性能要求极高的临界区
2. 保护某个结构内部状态和完整性

## 创建 channels

使用内置的make函数，我们可以创建一个channel：

```go
ch := make(chan int) // ch 拥有类型 'chan int'
```

## channel 是引用类型

当我们复制一个 channel 用于函数参数传递时，我们只是拷贝了一个 channel 引用，因此调用者和被调用者将引用同一个channel对象。

和其它的引用类型一样，channel 的零值也是 nil。

两个相同类型的 channel 可以使用 == 运算符比较。如果两个 channel 引用的是相同的对象，那么比较的结果为真。一个 channel 也可以和 nil 进行比较。

## channel 基础操作

channel 有发送和接受两个主要操作，都是通信行为。

```go
ch <- x  // 发送语句
x = <-ch // 赋值语句中的接收表达式
<-ch     // 接收语句
```

## channel 支持 close 操作

channel 还支持 close 操作，用于关闭 channel，随后对基于该 channel 的任何发送操作都将导致 panic 异常。

对一个已经被 close 过的 channel 进行接收操作依然可以接受到之前已经成功发送的数据；如果 channel 中已经没有数据的话将产生一个零值的数据。

```go
close(ch)
```

## 不带缓存的 channel

一个基于无缓存 channel 的发送操作将导致发送者 goroutine 阻塞，直到另一个 goroutine 在相同的 channel 上执行接收操作，当发送的值通过 channel 成功传输之后，两个 goroutine 可以继续执行后面的语句。

```go
ch = make(chan int)
```

## 带缓存的 Channels

带缓存的 channel 内部持有一个元素队列。队列的最大容量是在调用 make 函数创建 channel 时通过第二个参数指定的。

```go
ch = make(chan string, 3)
```

向缓存 channel 的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个 goroutine 执行接收操作而释放了新的队列空间。