# Golang 复合数据类型

## 数组

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。

数组的长度是数组类型的一个组成部分，因此 [3]int 和 [4]int 是两种不同的数组类型。

数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。

```go
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```

## slice

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

## Map

Map 的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。

在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。

这是故意的，每次都使用随机的遍历顺序可以强制要求程序不会依赖具体的哈希函数实现。

```go
for name, age := range ages {
    fmt.Printf("%s\t%d\n", name, age)
}
```

## 结构体

一个命名为 S 的结构体类型将不能再包含 S 类型的成员：因为一个聚合的值不能包含它自身。（该限制同样适用于数组。）

但是 S 类型的结构体可以包含 *S 指针类型的成员，这可以让我们创建递归的数据结构，比如链表和树结构等。

Golang 语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。

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
