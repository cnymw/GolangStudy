# leetcode 115 不同的子序列
给定一个字符串 S 和一个字符串 T，计算在 S 的子序列中 T 出现的个数。

一个字符串的一个子序列是指，通过删除一些（也可以不删除）字符且不干扰剩余字符相对位置所组成的新字符串。（例如，"ACE" 是 "ABCDE" 的一个子序列，而 "AEC" 不是）

示例 1:
```go
输入: S = "rabbbit", T = "rabbit"
输出: 3
```
解释:

如下图所示, 有 3 种可以从 S 中得到 "rabbit" 的方案。
(上箭头符号 ^ 表示选取的字母)
```go
rabbbit
^^^^ ^^
rabbbit
^^ ^^^^
rabbbit
^^^ ^^^
```

示例 2:
```go
输入: S = "babgbag", T = "bag"
输出: 5
```

解释:

如下图所示, 有 5 种可以从 S 中得到 "bag" 的方案。 
(上箭头符号 ^ 表示选取的字母)
```go
babgbag
^^ ^
babgbag
^^    ^
babgbag
^    ^^
babgbag
  ^  ^^
babgbag
    ^^^
```

题解：

首先根据输入 S = "rabbbit", T = "rabbit"，画出正确结果的状态图来：

![状态图](https://cnymw.github.io/GolangStudy/docs/img/算法-动态规划/算法-动态规划-leetcode115状态图.png)

```go
	j	1
i	dp	r
1	r	1
```
以上表格表示，当输入 S = r , T = r 时，只有一种子序列方案，可以分析出，当 S[j] == T[i] 时, dp[i][j] = 1

```go
	j	1
i	dp	r
1	r	1
2	a	0
```
以上表格表示，当输入 S = r , T = ra 时，只有零种子序列方案，可以分析出，当 len(T) > len(S) 时, dp[i][j] = 0

```go
	j	1	2
i	dp	r	a
1	r	1	1
```
以上表格表示，当输入 S = ra , T = r 时，只有一种子序列方案，可以分析出，当 S[j] != T[i], dp[i][j] = dp[i][j-1]

```go
	j	1	2	3	4
i	dp	r	a	b	b
1	r	1	1	1	1
2	a	0	1	1	1
3	b	0	0	1	2
```
以上表格表示，当输入 S = rabb , T = rab 时，有两种子序列方案，可以分析出，当 S[j] == T[i] && i>1 && j>1 时, dp[i-1][j-1] + dp[i][j-1]

再根据输入 S = "babgbag" , T = "bag"，可以得出以下状态图

![状态图](https://cnymw.github.io/GolangStudy/docs/img/算法-动态规划/算法-动态规划-leetcode115状态图2.png)

可以发现，当 i==1 时，因为左上方不存在，所以 dp[i][j] = dp[i][j-1] + 1

同理，当 j==1 时，因为左方不存在，所以 dp[i][j] = dp[i-1][j] + 1

当 i==1 and j==1 时，dp[i][j] = 1

处理好边界问题后，可以总结出:

1. if len(T) > len(S) , dp[i][j] = 0
2. if S[j] == T[i]
    1. if i==1 && j==1 , dp[i][j] = 1
    2. if i==1 , dp[i][j] = dp[i][j-1] + 1
    3. if j==1 , dp[i][j] = dp[i-1][j] + 1
    4. else dp[i][j] = dp[i][j-1] + dp[i-1][j-1]
3. if S[j] != T[i]
    1. if j == 1 , dp[i][j] = 0
    2. else dp[i][j] = dp[i][j-1]

根据以上分析可以得出：
```go
func numDistinct(s string, t string) int {
    if len(s) < len(t) {
        return 0
    }
    dp := make([][]int, len(t))
    for i := range dp {
        dp[i] = make([]int, len(s))
    }
    
    for i := 0; i < len(t); i++ {
        for j := 0; j < len(s); j++ {
            if i > j {
                dp[i][j] = 0
            } else if s[j] == t[i] {
                if i == 0 && j == 0 {
                    dp[i][j] = 1
                } else if i == 0 {
                    dp[i][j] = dp[i][j-1] + 1
                } else if j == 0 {
                    dp[i][j] = dp[i-1][j] + 1
                } else {
                    dp[i][j] = dp[i][j-1] + dp[i-1][j-1]
                }
            } else if s[j] != t[i] {
                if j == 0 {
                    dp[i][j] = 0
                } else {
                    dp[i][j] = dp[i][j-1]
                }
    
            }
        }
    }
    return dp[len(t)-1][len(s)-1]
}
```