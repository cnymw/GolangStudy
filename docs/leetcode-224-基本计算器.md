# leetcode 224 基本计算器
实现一个基本的计算器来计算一个简单的字符串表达式的值。

字符串表达式可以包含左括号 ( ，右括号 )，加号 + ，减号 -，非负整数和空格  。

示例 1:

```go
输入: "1 + 1"
输出: 2
```

示例 2:

```go
输入: " 2-1 + 2 "
输出: 3
```

示例 3:

```go
输入: "(1+(4+5+2)-3)+(6+8)"
输出: 23
```

题解：

这道题有了 [leetcode-150-逆波兰表达式求值](https://cnymw.github.io/go-study/docs/数据结构-栈.html) 的基础之后就会变得非常简单了，由题 150 可以知道，如果表达式转换为了后缀表达式，那么通过栈，计算机可以很快的计算出表达式的值。

因为输入的是中缀表达式，所以这道题的方法就是将中缀表达式转换为后缀表达式，然后通过栈计算出后缀表达式的值，在这里我们只需要将中缀转后缀的实现写出即可，计算后缀表达式的代码可以直接复用题 150 的代码。

```go
import (
    "regexp"
    "strconv"
)

var pattern = "\\d+"

func calculate(s string) int {
    runes := []rune(s)
    
    var tokens []string
    stack := New()
    
    nums := make([]rune, 0)
    var v interface{}
    for _, r := range runes {
        switch r {
        case '(':
            stack = stack.Push(string(r))
        case ')':
            if len(nums) != 0 {
                tokens = append(tokens, string(nums))
                nums = make([]rune, 0)
            }
    
            for {
                stack, v = stack.Pop()
                if v == "(" || stack.Empty() {
                    break
                }
                tokens = append(tokens, v.(string))
            }
        case '+', '-':
            if len(nums) != 0 {
                tokens = append(tokens, string(nums))
                nums = make([]rune, 0)
            }
            if stack.Empty() {
                stack = stack.Push(string(r))
            } else {
                pop := stack.Peek()
                if pop == "(" {
                    stack = stack.Push(string(r))
                    continue
                } else {
                    stack, _ = stack.Pop()
                    tokens = append(tokens, pop.(string))
                    stack = stack.Push(string(r))
                }
            }
        case ' ':
    
        default:
            nums = append(nums, r)
        }
    }
    if len(nums) != 0 {
        stack = stack.Push(string(nums))
    }
    for !stack.Empty() {
        stack, v = stack.Pop()
        tokens = append(tokens, v.(string))
    }
    
    var one, two interface{}
    stack = New()
    for _, t := range tokens {
        if isDigit(t) {
            stack = stack.Push(t)
        } else {
            stack, one = stack.Pop()
            stack, two = stack.Pop()
            one, two := evalInterface(one), evalInterface(two)
            result := 0
            if t == "+" {
                result = two + one
            } else if t == "-" {
                result = two - one
            }
            stack = stack.Push(result)
        }
    }
    stack, v = stack.Pop()
    result := evalInterface(v)
    return result
}

func isDigit(s string) bool {
    m, _ := regexp.MatchString(pattern, s)
    return m
}

func evalInterface(v interface{}) int {
    if i, ok := v.(string); ok {
        r, _ := strconv.Atoi(i)
        return r
    } else {
        return v.(int)
    }
}

type stack []interface{}

func New() stack {
    s := make(stack, 0)
    return s
}
func (s stack) Push(v interface{}) stack {
    return append(s, v)
}

func (s stack) Pop() (stack, interface{}) {
    if s.Len() == 0 {
        return s, nil
    }
    return s[:s.Len()-1], s[s.Len()-1]
}

func (s stack) Len() int {
    return len(s)
}

func (s stack) Peek() interface{} {
    return s[s.Len()-1]
}

func (s stack) Empty() bool {
    return s.Len() == 0
}

```