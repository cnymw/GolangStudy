# 1、Go 起源

## 1.1 Go 家族树

![Go家族树](/docs/img/go基础/go-tree.png)

# 2、基本概念

## 2.1 基本类型
### 2.1.1 整数
- 有符号整数: int8、int16、int32 和 int64
- 无符号整数: uint8、uint16、uint32 和 uint64
- 对应特定 CPU 平台机器字大小的有符号整数: int(32 或 64 bit)
- 对应特定 CPU 平台机器字大小的无符号整数: uint(32 或 64 bit)
- 无符号的整数类型 uintptr，没有指定具体的 bit 大小但是足以容纳指针，在底层编程时才需要

```go
var apples int32 = 1
var oranges int16 = 2
var compote int = apples + oranges // compile error , invalid operation: apples + oranges (mismatched types int32 and int16)
var compote = int(apples) + int(oranges) // compote=3
```
apples 和 oranges 不是 int 类型，所以赋值给 int 类型变量会编译报错。
```go
i := 123
j := int32(i)
i = j // compile error , Cannot use 'j' (type int32) as type int in assignment
```

变量 i 属于类型 int，在内存中用一个 32 位字长(word)表示。(32 位内存布局方式）

变量 j 由于做了精确的转换，属于 int32 类型。尽管 i 和 j 有着相同的内存布局，但是它们属于不同的类型：赋值操作 i = j 是一种类型错误，必须写成更精确的转换方式：i = int(j)。

### 2.1.2 浮点数
- float32
- float64

函数 math.IsNaN 用于测试一个数是否是非数 NaN，math.NaN 则返回非数对应的值，但是 NaN 和任何数都是不相等的。

```go
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

### 2.1.3 复数
- complex64
- complex128

内置的 complex 函数用于构建复数，内建的 real 和 imag 函数分别返回复数的实部和虚部：

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

### 2.1.4 布尔型
- true
- false

布尔值并不会隐式转换为数字值 0 或 1，反之亦然。
```go
b := true
if b == 1 {} // compile error , Invalid operation: b == 1 (mismatched types bool and untyped int)
i := 1
if i {} // compile error , Non-bool 'i' (type int) used as condition
```

### 2.1.5 字符串
文本字符串通常被解释为采用 UTF8 编码的 Unicode 码点（rune）序列，第 i 个字节并不一定是字符串的第 i 个字符，因为对于非 ASCII 字符的 UTF8 编码会要两个或多个字节。

字符串的值是不可变的：一个字符串包含的字节序列永远不会被改变。

因为字符串是不可修改的，因此尝试修改字符串内部数据的操作也是被禁止的：
```go
s := "left foot"
s[0] = 'L' // compile error: cannot assign to s[0]
```

### 2.1.6 原生的字符串
一个原生的字符串面值形式是\`...\`，使用反引号代替双引号。

在原生的字符串面值中，没有转义操作；全部的内容都是字面的意思，包含退格和换行，因此一个程序中的原生字符串面值可能跨越多行。

原生字符串面值用于编写正则表达式会很方便，因为正则表达式往往会包含很多反斜杠。原生字符串面值同时被广泛应用于 HTML 模板、JSON 面值、命令行提示信息以及那些需要扩展到多行的场景。

```go
const GoUsage = `Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...`
```

### 2.1.7 常量
常量表达式的值在编译期计算，而不是在运行期。每种常量的潜在类型都是基础类型：boolean、string 或数字。

const 不能用于初始化 struct 对象。

```go
const entity = &Entity{Concurrent: 1} // compile error,Const initializer '&UploadLimitEntity{Concurrent: 1}' is not a constant
```

常量声明可以使用 iota 常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。

```go
type Weekday int

const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)
```

### 2.1.8 无类型常量
- 无类型的布尔型
- 无类型的整数
- 无类型的字符
- 无类型的浮点数
- 无类型的复数
- 无类型的字符串

通过延迟明确常量的具体类型，无类型的常量不仅可以提供更高的运算精度，而且可以直接用于更多的表达式而不需要显式的类型转换。


## 2.2 复合数据类型

### 2.2.1 数组
数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。

数组的长度是数组类型的一个组成部分，因此 [3]int 和 [4]int 是两种不同的数组类型。

数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。

```go
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```

### 2.2.2 slice
Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。

slice 由三个部分构成
- 指针,指向第一个 slice 元素对应的底层数组元素的地址,要注意 slice 的第一个元素并不一定就是数组的第一个元素。
- 长度,对应 slice 中元素的数目,长度不能超过容量,内置函数 len 返回 slice 长度。
- 容量,容量一般是从 slice 的开始位置到底层数据的结尾位置，cap 返回 slice 容量。

和数组不同的是，slice 之间不能比较，我们不能使用 == 操作符来判断两个 slice 是否含有全部相等元素。

slice 不直接支持比较运算符的原因有两个：
1. 一个 slice 的元素是间接引用的，一个 slice 甚至可以包含自身。
2. 因为 slice 的元素是间接引用的，一个固定的 slice 值在不同的时刻可能包含不同的元素，因为底层数组的元素可能会被修改。

测试一个 slice 是否是空的，使用 len(s) == 0 来判断，而不应该用 s == nil 来判断。
```go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
```

每次扩展 slice 时直接将长度翻倍从而避免了多次内存分配，也确保了添加单个元素操的平均时间是一个常数时间。
```go
func main() {
    var x, y []int
    for i := 0; i < 10; i++ {
        y = appendInt(x, i)
        fmt.Printf("%d cap=%d\t%v\n", i, cap(y), y)
        x = y
    }
}

0  cap=1    [0]
1  cap=2    [0 1]
2  cap=4    [0 1 2]
3  cap=4    [0 1 2 3]
4  cap=8    [0 1 2 3 4]
5  cap=8    [0 1 2 3 4 5]
6  cap=8    [0 1 2 3 4 5 6]
7  cap=8    [0 1 2 3 4 5 6 7]
8  cap=16   [0 1 2 3 4 5 6 7 8]
9  cap=16   [0 1 2 3 4 5 6 7 8 9]
```

### 2.2.3 Map
Map 的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。

在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。

这是故意的，每次都使用随机的遍历顺序可以强制要求程序不会依赖具体的哈希函数实现。

```go
for name, age := range ages {
    fmt.Printf("%s\t%d\n", name, age)
}
```

### 2.2.4 结构体
一个命名为 S 的结构体类型将不能再包含 S 类型的成员：因为一个聚合的值不能包含它自身。（该限制同样适用于数组。）

但是 S 类型的结构体可以包含 *S 指针类型的成员，这可以让我们创建递归的数据结构，比如链表和树结构等。

Go 语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。

```go
type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
var w Wheel
w.X = 8            // equivalent to w.Circle.Point.X = 8
w.Y = 8            // equivalent to w.Circle.Point.Y = 8
w.Radius = 5       // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```
## 2.3 运算
### 运算符优先级
以下运算符按照优先级递减:
- *，/，%，<<，>>，&，&^
- +，-，|，^
- ==，!= ，<，<=，>，>=
- &&
- ||

# 3、思维导图

go 基础的思维导图的原件以及 PDF 在 [go 基础思维导图](/docs/mind/go基础) 下面，有需要的可以下载以便随时查看。

![go 基础](/docs/mind/go基础/go基础.jpg)