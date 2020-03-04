# leetcode 64 最小路径和
给定一个包含非负整数的 m x n 网格，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

说明：每次只能向下或者向右移动一步。

示例:

输入:
```go
[
    [1,3,1],
    [1,5,1],
    [4,2,1]
]
```

输出: 
```go
7
解释: 因为路径 1→3→1→1→1 的总和最小
```

题解：

以示例输入的值为例，画出状态图:

![状态图](https://cnymw.github.io/go-study/docs/img/算法-动态规划/算法-动态规划-leetcode64状态图.png)

通过观察状态图，可以看出当 j==i==0 时，dp[j][i] = nums[0][0] = 1

因为每次只能向下或者向右移动一步，所以第二步必须往右或往下走，观察一下一直往下或一直往右的情况，会发现 dp[j][i] 的数值只取决于上一个 dp[j-1][0] 或 dp[0][i-1] 的值，即：

当 i==0 或 j==0 时, dp[j][i] = nums[j][i] + dp[j-1][0] 或 dp[j][i] = nums[j][i] + dp[0][i-1]

当 i>0 同时 j>0 时，dp[j][i] 的值可以做选择，可以来自于向右的一步或向下的一步，这个取决于哪个动作总和最小，所以可以得出:

dp[j][i] = nums[j][i] + min{dp[j-1][i],dp[j][i-1]}

于是可以得出如下伪代码：

```go
if i==j==0
    dp[j][i] = nums[0][0]
else if j==0 and i>0
    dp[j][i] = nums[j][i] + dp[j][i-1] = nums[0][i] + dp[0][i-1]
else if i==0 and j>0
    dp[j][i] = nums[j][i] + dp[j-1][i] = nums[j][0] + dp[j-1][0]
else if j>0 and i>0
    dp[j][i] = nums[j][i] + min{dp[j-1][i],dp[j][i-1]}
```

根据伪代码可以写出实现：
```go
func minPathSum(grid [][]int) int {
    jLen, iLen := len(grid), len(grid[0])
    dp := make([][]int, jLen)
    for i := range dp {
        dp[i] = make([]int, iLen)
    }
    
    for i := 0; i < iLen; i++ {
        for j := 0; j < jLen; j++ {
            if i == 0 && j == 0 {
                dp[j][i] = grid[j][i]
            } else if j == 0 && i > 0 {
                dp[j][i] = dp[j][i-1] + grid[j][i]
            } else if i == 0 && j > 0 {
                dp[j][i] = dp[j-1][i] + grid[j][i]
            } else {
                dp[j][i] = int(math.Min(float64(dp[j-1][i]), float64(dp[j][i-1]))) + grid[j][i]
            }
        }
    }
    return dp[jLen-1][iLen-1]
}
```
