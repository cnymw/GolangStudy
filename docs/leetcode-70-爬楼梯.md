# leetcode 70 爬楼梯
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

示例 1：
```go
输入: 2
输出: 2
解释: 有两种方法可以爬到楼顶

1. 1 阶 + 1 阶
2. 2 阶
```


示例 2：
```go
输入: 3
输出: 3
解释: 有三种方法可以爬到楼顶

1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶
```

题解

爬到第 n 阶，要么从第 n-1 阶爬 1 步达到，要么从第 n-2 阶爬 2 步达到。

所以递推公式为 d[p] = d[p-1] + d[p-2]

```go
func climbStairs(n int) int {
    if n == 0 {
        return 0
    }
    if n == 1 {
        return 1
    }
    if n == 2 {
        return 2
    }
    oneStep, twoStep, total := 1, 2, 0
    for index := 3; index <= n; index++ {
        total = oneStep + twoStep
        oneStep = twoStep
        twoStep = total
    }
    return total
}
```