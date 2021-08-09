# go 接口

## 接口类型

接口类型是一种抽象的类型，它不会暴露出它所代表的对象的内部值的结构和这个对象支持的基础操作的集合，它们只会表现出它们自己的方法。

接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。

### 接口举例

io.Writer 类型是用得最广泛的接口之一，因为它提供了所有类型的写入bytes的抽象，包括文件类型，内存缓冲区，网络链接，HTTP客户端，压缩工具，哈希等等。

io 包中定义了很多其它有用的接口类型。Reader 可以代表任意可以读取 bytes 的类型，Closer 可以是任意可以关闭的值，例如一个文件或是网络链接。

```go
package io
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Closer interface {
    Close() error
}
```