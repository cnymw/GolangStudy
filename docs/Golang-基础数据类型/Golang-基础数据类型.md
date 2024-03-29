# Golang 基础数据类型

Golang 语言将数据类型分为四类：基础类型、复合类型、引用类型和接口类型，本章介绍基础类型。

## 整型

- 有符号整数: int8、int16、int32 和 int64
- 无符号整数: uint8、uint16、uint32 和 uint64
- 对应特定 CPU 平台机器字大小的有符号整数: int(32 或 64 bit)
- 对应特定 CPU 平台机器字大小的无符号整数: uint(32 或 64 bit)
- rune 类型是和 int32 等价的类型，通常用于表示一个 Unicode 码点
- byte 是 uint8 等价的类型，byte 一般用于强调数值是一个原始的数据而不是一个小的整数
- 无符号的整数类型 uintptr，没有指定具体的 bit 大小但是足以容纳指针，在底层编程时才需要

## 浮点数

- float32：可以提供大约6个十进制数的精度。
- float64：可以提供约15个十进制数的精度

通常应该优先使用 float64 类型，因为 float32 类型的累计计算误差很容易扩散，并且 float32 能精确表示的正整数并不是很大。

## 复数

- complex64：对应 float32 浮点数精度。
- complex128：对应 float64 浮点数精度。

内置的 complex 函数用于构建复数，内建的 real 和 imag 函数分别返回复数的实部和虚部：

```go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

## 布尔型

- true
- false

布尔值并不会隐式转换为数字值 0 或 1，反之亦然。

```go
b := true
if b == 1 {} // compile error , Invalid operation: b == 1 (mismatched types bool and untyped int)
i := 1
if i {} // compile error , Non-bool 'i' (type int) used as condition
```

## 字符串

一个字符串是一个不可改变的字节序列。

文本字符串通常被解释为采用 UTF-8 编码的 Unicode 码点（rune）序列。

因为字符串是不可修改的，因此尝试修改字符串内部数据的操作也是被禁止的。

### 原生字符串

一个原生的字符串面值形式是 \`...\` ，使用反引号代替双引号。

在原生的字符串面值中，没有转义操作；全部的内容都是字面的意思，包含退格和换行，因此一个程序中的原生字符串面值可能跨越多行。

原生字符串面值用于编写正则表达式会很方便，因为正则表达式往往会包含很多反斜杠。原生字符串面值同时被广泛应用于 HTML 模板、JSON 面值、命令行提示信息以及那些需要扩展到多行的场景。

### 字符串和 byte 切片

字符串和字节 slice 之间可以相互转换，一个 []byte(s) 转换是分配了一个新的字节数组用于保存字符串数据的拷贝，然后引用这个底层的字节数组。

```go
s := "abc"
b := []byte(s)
s2 := string(b)
```

## 常量

常量表达式的值在编译期计算，而不是在运行期。每种常量的潜在类型都是基础类型：boolean、string 或数字。

常量的值不可修改，这样可以防止在运行期被意外或恶意的修改。

### iota 常量生成器

常量声明可以使用 iota 常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。

在一个 const 声明语句中，在第一个声明的常量所在的行，iota 将会被置为 0，然后在每一个有常量声明的行加一。

### 无类型常量

一个常量可以有任意一个确定的基础类型，也可以没有一个明确的基础类型。编译器为这些没有明确基础类型的数字常量提供比基础类型更高精度的算术运算。

通过延迟明确常量的具体类型，无类型的常量不仅可以提供更高的运算精度（你可以认为至少有 256bit 的运算精度），而且可以直接用于更多的表达式而不需要显式的类型转换。

# 思维导图

![Golang-基础数据类型-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Golang-基础数据类型/Golang-基础数据类型-思维导图.png)

