# Go 源码解读-标识符

> 源码地址：builtin/builtin.go
>
> 源码版本：1.17.6

标识符是指 Go 语言对各种变量、方法、函数等命名时使用的字符序列，标识符由若干个字母、下划线`_`和数字组成。

通俗的讲就是凡可以自己定义的名称都可以叫做标识符。

---

## 空白标识符

下划线`_`是一个特殊的标识符，称为空标识符，它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用`_`作为变量对其它变量进行赋值或运算。

---

## 标识符命名规则

- 由 26 个英文字母、0~9、下划线"_"组成
- 不能以数字开头，例如 var 1num int 是错误的
- Go 语言中严格区分大小写
- 标识符不能包含空格
- 不能以系统保留关键字作为标识符，比如 break，if 等等

---

## Go SDK 预先声明的标识符

Go 预先声明的标识符定义在 Go SDK 的 `/builtin/builtin.go` 文件中。

### 布尔型

```go
// bool 是一组布尔值，true 和 false
type bool bool
```

一个布尔类型的值只有两种：

- true
- false

### 无符号整型

```go
// uint 是大小至少为 32 位的无符号整数类型。
// 但是，他是一个独特的类型，而不是 uint32 的别名。
type uint uint

// uint8 是所有无符号 8 位整数的集合。
// 范围：0～255
type uint8 uint8

// uint16 是所有无符号 16 位整数的集合。
// 范围：0～65535
type uint16 uint16

// uint32 是所有无符号 32 位整数的集合。
// 范围：0～4294967295
type uint32 uint32

// uint64 是所有无符号 64 位整数的集合。
// 范围：0～18446744073709551615
type uint64 uint64
```

无符号整型有如下几种类型：

- uint
- uint8
- uint16
- uint32
- uint64

其中，uint 有两种大小：32 或 64 bit，但是我们不能对此做任何假设，因为不同的编译器即使在相同的硬件平台上可能产生不同的大小。

### 有符号整型

```go
// int 是大小至少为 32 位的有符号整数类型。
// 但是，他是一个独特的类型，而不是 int32 的别名。
type int int

// int8 是所有有符号 8 位整数的集合。
// 范围：-128～127
type int8 int8

// int16 是所有有符号 16 位整数的集合。
// 范围：-32768～32767
type int16 int16

// int32 是所有有符号 32 位整数的集合。
// 范围：-2147483648~2147483647
type int32 int32

// int64 是所有有符号 64 位整数的集合。
// 范围：-9223372036854775808～9223372036854775807
type int64 int64
```

有符号整型有如下几种类型：

- int
- int8
- int16
- int32
- int64

其中，int 有两种大小：32 或 64 bit，但是我们不能对此做任何假设，因为不同的编译器即使在相同的硬件平台上可能产生不同的大小。

### 浮点数

```go
// float32 是所有 IEEE-754 32 位浮点数的集合。
type float32 float32

// float64 是所有 IEEE-754 64 位浮点数的集合。
type float64 float64
```

浮点数的类型有如下几种类型：

- float32
- float64

浮点数的范围极限值可以在 `math/const.go` 包找到：

```go
// 浮点数限制值。
// Max 是由类型表示的最大有限值。
// SmallestNonzero 是由类型表示的最小的非负值。
const (
	MaxFloat32             = 3.40282346638528859811704183484516925440e+38  // 2**127 * (2**24 - 1) / 2**23
	SmallestNonzeroFloat32 = 1.401298464324817070923729583289916131280e-45 // 1 / 2**(127 - 1 + 23)

	MaxFloat64             = 1.797693134862315708145274237317043567981e+308 // 2**1023 * (2**53 - 1) / 2**52
	SmallestNonzeroFloat64 = 4.940656458412465441765687928682213723651e-324 // 1 / 2**(1023 - 1 + 52)
)
```

### 复数

```go
// complex64 是所有对应 float32 浮点数精度的复数集合。
type complex64 complex64

// complex128 是所有对应 float64 浮点数精度的复数集合。
type complex128 complex128
```

复数的类型有如下几种类型：

- complex64
- complex128

### 字符串

```go
// string 是所有 8 位字节的字符串的集合，但是通常不一定代表 UTF-8 编码的文本。
// 字符串可以为空，但不能为 nil。
// 字符串类型的值是不可变的。
type string string
```

字符串的类型有如下一种类型：

- string

### 指针

```go
// uintptr 是一个整数类型，其大小足以容纳任何指针的位模式。
type uintptr uintptr
```

uintptr 和 unsafe.Pointer的区别：

- unsafe.Pointer 只是单纯的通用指针类型，用于转换不同类型指针，它不可以参与指针运算
- uintptr 是用于指针运算的，GC 不把 uintptr 当指针，也就是说 uintptr 无法持有对象， uintptr 类型的目标会被回收
- unsafe.Pointer 可以和 普通指针 进行相互转换
- unsafe.Pointer 可以和 uintptr 进行相互转换

```go
var ptr uintptr
ptr = uintptr(unsafe.Pointer(t))
fmt.Println(unsafe.Pointer(t)) // output:0xc000102900
fmt.Println(ptr) // output:824634779904
```

通过以上代码可以看出，unsafe.Pointer 是 16 进制的内存地址，uintptr 是 10 进制的内存地址。

所以这是为什么，unsafe.Pointer 和普通指针，uintptr 之间可以进行转换。

### 字节

```go
// byte 是 uint8 的别名，在所有方面都等效于 uint8。
// 按照惯例，它用于区分字节和 8 位无符号整数。
type byte = uint8
```

byte 代表了 ASCII 码的一个字符。

### rune

```go
// rune 是 uint32 的别名，在所有方面都等效于 uint32。
// 按照惯例，它用于区分字符值和整数值。
type rune = int32
```

rune 代表一个 UTF-8 字符，当需要处理中文，日文或者其他复合字符时，需要用到 rune 类型。

### iota

```go
// itoa 是一个预先声明的标识符，表示 const 声明（通常用括号括起来）中当前 const 规范的非类型化整数序列。
// itoa 是从 0 开始索引的。
const iota = 0 // Untyped int.
```

iota 是 golang 里的常量计数器，只能在常量的表达式中使用。

### nil

```go
// nil 是预声明标识符，代表指针 pointer，channel 通道，func 函数，interface 接口，map 映射，slice 切片类型的空值。
var nil Type // Type must be a pointer, channel, func, interface, map, or slice type
// Type 必须是 pointer 指针，channel 通道，func 函数，interface 接口，map 映射，slice 切片类型
```

在 Go 语言中，布尔类型的零值（初始值）为 false，数值类型的零值为 0，字符串类型的零值为空字符串""，而指针、切片、映射、通道、函数和接口的零值则是 nil。

### append

```go
// append 内置函数将元素追加到切片的末尾。
// 如果目标切片有足够的容量，将在目标切片里分配新的元素。
// 如果目标切片没有足够的容量，将分配一个新的切片来容纳所有的元素。
// Append 函数返回更新过后的切片。
// 因此，有必要将 append 的结果存放在切片原来的变量中：
//      slice = append(slice, elem1, elem2)
//      slice = append(slice, anotherSlice...)
// 有一个特殊的情况，将字符串 append 到字节切片中去是合法的，例如：
//      slice = append([]byte("hello "), "world"...)
func append(slice []Type, elems ...Type) []Type
```

内置的 append 函数用于向 slice 追加元素。

### copy

```go
// 内置函数 copy 将元素从源 slice 复制到目标 slice（有一个特殊情况，它还会将字符串中的字节复制到字节 slice 中）。
// 源 slice 和目标 slice 可能重叠。
// copy 返回复制的元素数，它是 len(src) 和 len(dst) 中的最小值。
func copy(dst, src []Type) int
```

copy 函数的第一个参数是要复制的目标 slice，第二个参数是源 slice，目标和源的位置顺序和 dst = src 赋值语句是一致的。

copy 函数将返回成功复制的元素的个数，等于两个 slice 中较小的长度，所以我们不用担心覆盖会超出目标 slice 的范围。

### delete

```go
// 内置函数 delete 从映射中删除具有指定键（m[key]）的元素。
// 如果 m 是 nil 或者没有这样的元素，delete 就是空操作。
func delete(m map[Type]Type1, key Type)
```

内置的 delete 函数可以删除 map 元素，delete 操作是安全的，即使这些元素不在 map 中也没有关系。

### len

```go
// 内置函数 len 根据 v 的类型返回 v 的长度：
//      数组：v 中的元素数。
//      指向数组的指针：*v 中的元素数（即使 v 为 nil）。
//      切片或映射：v 中的元素数，如果 v 为 nil，len（v）为零。
//      字符串：v 中的字节数。
//      通道：通道缓冲区中队列（未读）的元素数。如果 v 为 nil，len（v）为零。
// 对于某些参数，例如字符串或简单数组表达式，结果可以是常数。
// 细节可以参见 Go 语言规范的"长度和容量"章节。
func len(v Type) int
```

### cap

```go
// 内置函数 cap 根据 v 的类型返回 v 的容量：
//      数组：v 中的元素数（与 len（v）相同）
//      指向数组的指针：*v 中的元素数（与 len（v）相同）
//      切片：重新切片时切片可以达到的最大长度，如果 v 为 nil，那么 cap（v）为零
//      通道：通道缓冲量，以元素为单位，如果 v 为 nil，那么 cap（v）为零
// 对于一些参数，例如简单的数组表达式，结果可以是常量。
// 细节可以参见 Go 语言规范的"长度和容量"章节。
func cap(v Type) int
```

### make

```go
// 内置函数 make 分配并初始化 slice，map 或 chan 类型的对象。
// 与 new 一样，第一个参数是类型，而不是值。
// 与 new 不同的是，make 的返回类型与其参数的类型相同，而不是指向它的指针。
// 结果的定义取决于类型：
//      Slice：size 指定了长度。切片的容量等于它的长度。可以提供第二个整数参数来指定不同的容量；
//      它不能小于长度。例如，make（[]int,0,10）分配一个大小为 10 的底层数组，并返回一个长度为 0，
//      容量为 10 的切片，该切片由这个底层数组支持。
//      Map：为空 map 分配足够的空间来容纳指定数量的元素。可以忽略大小，在这种情况下，系统会
//      分配一个小的起始大小。
//      Channel：使用指定的缓冲区容量初始化通道的缓冲区。如果大小指定为 0，那么通道是无缓冲的。
func make(t Type, size ...IntegerType) Type
```

### new

```go
// 内置函数 new 分配内存。
// 第一个参数是类型，不是值，new 返回的值是指向该类型新分配的零值的指针。
func new(Type) *Type
```

### complex

```go
// 内置函数 complex 从两个浮点值构造一个复数。
// 实部和虚部必须是同样的大小，可以是 float32 或 float64（或可分配给它们）。
// 返回值将是相应的复数类型（complex64 对应 float32，complex128 对应 float64）。
func complex(r, i FloatType) ComplexType
```

### real

```go
// 内置函数 real 返回复数 c 的实数部分。
// 返回值将是与 c 类型相对应的浮点类型。
func real(c ComplexType) FloatType
```

- 如果 c 类型是 complex64，那么 real 返回类型 float32。
- 如果 c 类型是 complex128，那么 real 返回类型 float64。

### imag

```go
// 内置函数 imag 返回复数 c 的虚数部分。
// 返回值将是与 c 类型相对应的浮点类型。
func imag(c ComplexType) FloatType
```

- 如果 c 类型是 complex64，那么 imag 返回类型 float32。
- 如果 c 类型是 complex128，那么 imag 返回类型 float64。

### close

```go
// 内置函数 close 关闭一个通道，该通道必须是双向的或仅发送的。
// close 应该只由发送方执行，而不是由接收方执行，并且具有在接受最后一个发送的值之后关闭通道的效果。
// 在从一个已经关闭的通道 c 里接受到最后一个值之后，从 c 接收的任何值都将成功而不阻塞，并且返回 channel 里元素的零值。
// 对于一个已经关闭的通道，代码
//      x, ok := <-c
// 将设置 ok 为false。
func close(c chan<- Type)
```

channel 在 close 之后，还可以读取，不过读取完最后一个值之后 ok 会为 false，读取的元素值为零值。

### panic

```go
// 内置函数 panic 停止当前 goroutine 的正常执行。
// 当函数 F 调用 panic 时，F 的正常执行立即停止。
// 任何被 F defer 执行的函数都以正常的方式运行，然后 F 返回给它的调用者。
// 对调用方 G 来说，F 的调用行为就像对 panic 的调用，终止 G 的执行并运行任何 defer 函数。
// 将持续以上逻辑，直到所有的函数按照相反方向停止为止。
// 此时，程序以非零退出代码终止。
// 这个终止链被称为 panicking，可由内置函数 recover 控制。
func panic(v interface{})
```

在多层嵌套的函数调用中调用 panic，可以马上中止当前函数的执行，所有的 defer 语句都会保证执行并把控制权交还给接收到 panic 的函数调用者。

这样向上冒泡直到最顶层，并执行（每层的） defer，在栈顶处程序崩溃，并在命令行中用传给 panic 的值报告错误情况：这个终止过程就是 panicking。

不能随意地用 panic 中止程序，必须尽力补救错误让程序能继续执行。

### recover

```go
// 内置函数 recover 允许程序去管理一个发生 panicking 的协程。
// 在 defer 函数（而不是它调用的任何函数）内执行 recover 调用，将恢复正常执行来停止 panicking 链并获取到 panic 调用时所传递的 error 对象。
// 如果在 defer 函数外部调用 recover，它将不会停止 panicking 链。
// 在这种情况下，或者当 goroutine 没有 panic 时，或者如果提供给 panic 的参数为 nil，recover 返回 nil。
// 因此，recover 的返回值反映了 goroutine 是否 panicking。
func recover() interface{}
```

### print

```go
// 内置函数 print 以特定实现的方式格式化其参数，并将结果写入标准错误。
// Print 对于引导和调试很有用；
// 它不能保证留在语言中。
func print(args ...Type)
```

print 在 golang 中负责输出到标准错误流中并打印,官方不建议写程序时候用它，可以在 debug 时候用。

官方不保证后续版本 SDK 会保留这个函数，最好还是用 fmt.print

### println

```go
// 内置函数 println 以特定实现的方式格式化其参数，并将结果写入标准错误。
// 在参数中会添加空格，并且在最后会添加换行符。
// Println 对于引导和调试很有用；
// 它不能保证留在语言中。
func println(args ...Type)
```

println 在 golang 中负责输出到标准错误流中并打印,官方不建议写程序时候用它，可以在 debug 时候用。

官方不保证后续版本 SDK 会保留这个函数，最好还是用 fmt.println

### error

```go
// 内置接口 error 是表示错误条件的常规接口，nil 值表示没有错误。
type error interface {
	Error() string
}
```

error 类型是 go 语言的一种内置类型，他本质上是一个接口。

只有实现了 Error() 方法才能使用 error。

在 Go SDK 中，errors 包实现了 error 接口，所以我们通常使用 errors.New() 来声明一个 error 对象：

```go
package errors

// New 返回一个格式为给定文本的 error。
// 每次调用 New 都会返回一个不同的错误值，即使文本是相同的。
func New(text string) error {
	return &errorString{text}
}

// errorString 是 error 的一个简单实现。
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

---

## 思维导图

![go-源码解读-标识符-思维导图.png](https://cnymw.github.io/GolangStudy/docs/go-源码解读-标识符/go-源码解读-标识符.png)