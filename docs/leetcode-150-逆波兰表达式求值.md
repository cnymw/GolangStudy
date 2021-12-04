# leetcode 150 逆波兰表达式求值

## 题目

根据逆波兰表示法，求表达式的值。

有效的运算符包括 +, -, *, / 。每个运算对象可以是整数，也可以是另一个逆波兰表达式。

说明：

整数除法只保留整数部分。
给定逆波兰表达式总是有效的。换句话说，表达式总会得出有效数值且不存在除数为 0 的情况。

示例 1：
```text
输入: ["2", "1", "+", "3", "*"]
输出: 9
解释: ((2 + 1) * 3) = 9
```

示例 2：
```text
输入: ["4", "13", "5", "/", "+"]
输出: 6
解释: (4 + (13 / 5)) = 6
```

示例 3：
```text
输入: ["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]
输出: 22
解释: 
  ((10 * (6 / ((9 + 3) * -11))) + 17) + 5
= ((10 * (6 / (12 * -11))) + 17) + 5
= ((10 * (6 / -132)) + 17) + 5
= ((10 * 0) + 17) + 5
= (0 + 17) + 5
= 17 + 5
= 22
```

## 题解

这道题是栈的经典应用，逆波兰表达式也就是后缀表达式，有关于前中后缀表达式的笔记请看 [前中后缀表达式-后缀转中缀](https://cnymw.github.io/GolangStudy/docs/算法-前中后缀表达式.html)

知道如何从后缀转中缀之后，这道题就非常简单了。

go 实现如下:

```go
import (
    "regexp"
    "strconv"
)

var pattern = "\\d+"

func evalInterface(v interface{}) int {
    if i, ok := v.(string); ok {
        r, _ := strconv.Atoi(i)
        return r
    } else {
        return v.(int)
    }
}
func evalRPN(tokens []string) int {
    s := New()
    for _, t := range tokens {
        if isDigit(t) {
            s.Push(t)
        } else {
            one, two := evalInterface(s.Pop()), evalInterface(s.Pop())
            result := 0
            if t == "+" {
                result = two + one
            } else if t == "-" {
                result = two - one
            } else if t == "*" {
                result = two * one
            } else if t == "/" {
                result = two / one
            }
            s.Push(result)
        }
    }
    result := evalInterface(s.Pop())
    return result
}

func isDigit(s string) bool {
    m, _ := regexp.MatchString(pattern, s)
    return m
}

type (
    Stack struct {
        top    *node
        length int
    }
    node struct {
        value interface{}
        prev  *node
    }
)

// Create a new stack
func New() *Stack {
    return &Stack{nil, 0}
}

// Return the number of items in the stack
func (this *Stack) Len() int {
    return this.length
}

// View the top item on the stack
func (this *Stack) Peek() interface{} {
    if this.length == 0 {
        return nil
    }
    return this.top.value
}

// Pop the top item of the stack and return it
func (this *Stack) Pop() interface{} {
    if this.length == 0 {
        return nil
    }
    
    n := this.top
    this.top = n.prev
    this.length--
    return n.value
}

// Push a value onto the top of the stack
func (this *Stack) Push(value interface{}) {
    n := &node{value, this.top}
    this.top = n
    this.length++
}

func (this *Stack) Empty() bool {
    return this.length == 0
}
```