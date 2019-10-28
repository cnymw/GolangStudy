# 一、Go 起源

## Go 家族树

![Go家族树](/docs/img/Go基础/Go-tree.png)

# 二、数据类型

## 基本类型
### 整数
- 有符号整数: int8、int16、int32 和 int64
- 无符号整数: uint8、uint16、uint32 和 uint64
- 对应特定CPU平台机器字大小的有符号整数: int(32 或 64 bit)
- 对应特定CPU平台机器字大小的无符号整数: uint(32 或 64 bit)
- 无符号的整数类型 uintptr，没有指定具体的 bit 大小但是足以容纳指针，在底层编程时才需要

```go
var apples int32 = 1
var oranges int16 = 2
var compote int = apples + oranges // compile error , invalid operation: apples + oranges (mismatched types int32 and int16)
var compote = int(apples) + int(oranges) // compote=3
```

### 浮点数
- float32
- float64

函数math.IsNaN用于测试一个数是否是非数NaN，math.NaN则返回非数对应的值，但是NaN和任何数都是不相等的。

```go
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

### 复数
- complex64
- complex128

内置的complex函数用于构建复数，内建的real和imag函数分别返回复数的实部和虚部：

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

### 布尔型
- true
- false

布尔值并不会隐式转换为数字值0或1，反之亦然。
```go
b := true
if b == 1 {} // compile error , Invalid operation: b == 1 (mismatched types bool and untyped int)
i := 1
if i {} // compile error , Non-bool 'i' (type int) used as condition
```

### 字符串
文本字符串通常被解释为采用UTF8编码的Unicode码点（rune）序列，第i个字节并不一定是字符串的第i个字符，因为对于非ASCII字符的UTF8编码会要两个或多个字节。

字符串的值是不可变的：一个字符串包含的字节序列永远不会被改变。

因为字符串是不可修改的，因此尝试修改字符串内部数据的操作也是被禁止的：
```go
s := "left foot"
s[0] = 'L' // compile error: cannot assign to s[0]
```

### 原生的字符串
在原生的字符串面值中，没有转义操作；全部的内容都是字面的意思，包含退格和换行，因此一个程序中的原生字符串面值可能跨越多行。

原生字符串面值用于编写正则表达式会很方便，因为正则表达式往往会包含很多反斜杠。原生字符串面值同时被广泛应用于HTML模板、JSON面值、命令行提示信息以及那些需要扩展到多行的场景。

```go
const GoUsage = `Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...`
```

### 常量
常量表达式的值在编译期计算，而不是在运行期。每种常量的潜在类型都是基础类型：boolean、string或数字。

```go
const entity = &Entity{Concurrent: 1} // compile error,Const initializer '&UploadLimitEntity{Concurrent: 1}' is not a constant
```

常量声明可以使用iota常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。

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

### 无类型常量
- 无类型的布尔型
- 无类型的整数
- 无类型的字符
- 无类型的浮点数
- 无类型的复数
- 无类型的字符串

通过延迟明确常量的具体类型，无类型的常量不仅可以提供更高的运算精度，而且可以直接用于更多的表达式而不需要显式的类型转换。

# 三、运算
## 运算符优先级
以下运算符按照优先级递减:
- *，/，%，<<，>>，&，&^
- +，-，|，^
- ==，!= ，<，<=，>，>=
- &&
- ||
