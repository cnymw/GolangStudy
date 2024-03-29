# go 接口

## 接口类型

接口类型是一种抽象的类型，它不会暴露出它所代表的对象的内部值的结构和这个对象支持的基础操作的集合，它们只会表现出它们自己的方法。

接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。

每一个具体类型的组基于它们相同的行为可以表示成一个接口类型。

不像基于类的语言，他们一个类实现的接口集合需要进行显式的定义，在Go语言中我们可以在需要的时候定义一个新的抽象或者特定特点的组，而不需要修改具体类型的定义。

当具体的类型来自不同的作者时这种方式会特别有用。当然也确实没有必要在具体的类型中指出这些共性。

## 实现接口的条件

一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。

## 空接口类型 interface{}

interface{} 被成为空接口类型，因为空接口类型对实现它的类型没有要求，所以我们可以将任意一个值赋给空接口类型。

```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```

## 接口值

概念上讲一个接口的值，接口值，由两个部分组成，一个具体的类型和那个类型的值。它们被称为接口的动态类型和动态值。

对于像Go语言这种静态类型的语言，类型是编译期的概念；因此一个类型不是一个值。

在Go语言中，变量总是被一个定义明确的值初始化，即使接口类型也不例外。对于一个接口的零值就是它的类型和值的部分都是nil。

一个接口值基于它的动态类型被描述为空或非空，所以这是一个空的接口值。你可以通过使用 w==nil 或者 w!=nil 来判断接口值是否为空。调用一个空接口值上的任意方法都会产生 panic:

```go
w.Write([]byte("hello")) // panic: nil pointer dereference
```

接口值可以使用 == 和 !＝ 来进行比较。两个接口值相等仅当它们都是 nil 值，或者它们的动态类型相同并且动态值也根据这个动态类型的 == 操作相等。因为接口值是可比较的，所以它们可以用在 map 的键或者作为 switch
语句的操作数。

然而，如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（比如切片），将它们进行比较就会失败并且panic:

```go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```

考虑到这点，接口类型是非常与众不同的。其它类型要么是安全的可比较类型（如基本类型和指针）要么是完全不可比较的类型（如切片，映射类型，和函数），但是在比较接口值或者包含了接口值的聚合类型时，我们必须要意识到潜在的
panic。同样的风险也存在于使用接口作为 map 的键或者 switch 的操作数。只能比较你非常确定它们的动态值是可比较类型的接口值。

## 类型断言

类型断言是一个使用在接口值上的操作。语法上它看起来像 x.(T) 被称为断言类型，这里 x 表示一个接口的类型和T表示一个类型。一个类型断言检查它操作对象的动态类型是否和断言的类型匹配。

这里有两种可能：

- 第一种，如果断言的类型T是一个具体类型，然后类型断言检查x的动态类型是否和T相同。如果这个检查成功了，类型断言的结果是x的动态值，当然它的类型是T。换句话说，具体类型的类型断言从它的操作对象中获得具体的值。如果检查失败，接下来这个操作会抛出panic。
- 第二种，如果相反地断言的类型 T 是一个接口类型，然后类型断言检查是否 x 的动态类型满足 T。如果这个检查成功了，动态值没有获取到；这个结果仍然是一个有相同动态类型和值部分的接口值，但是结果为类型 T。换句话说，对一个接口类型的类型断言改变了类型的表述方式，改变了可以获取的方法集合（通常更大），但是它保留了接口值内部的动态类型和值的部分。



