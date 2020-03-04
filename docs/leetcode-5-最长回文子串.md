# leetcode 5 最长回文子串
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：
```go
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案
```


示例 2：
```go
输入: "cbbd"
输出: "bb"
```

题解

以输入"babad"为例，画出状态图:

![状态图](https://cnymw.github.io/go-study/docs/img/算法-动态规划/算法-动态规划-leetcode5状态图.png)

容易看出，当子串长为 1，也就是当 i == j 时，dp[j,i]=1

当相邻的(满足条件: i == j+1)两个字符相等(满足条件: s[i] == s[j])时，也就是当 s[i] == s[j] && i == j+1 时，dp[j,i]=1

当不相邻(满足条件: i-j > 2 )的两个字符相等(满足条件: s[i] == s[j])，同时之间的子串满足回文条件(满足条件: dp[j+1][i-1] == 1)，也就是当 s[i] == s[j] && i-j > 2 && dp[j+1][i-1] == 1 时，dp[j][i]=1

由此总结出以下状态转移方程(伪代码):

```go
if i == j
    dp[i,j] = 1
else if s[i]==s[j] && j == i + 1
    dp[i,j] = 1
else if s[i]==s[j] && i-j > 2 && dp[j+1][i-1]
    dp[j,i] = 1
```

其中后面两个条件可以合为一个条件:

```go
if (s[i] == s[j]) && (i-j < 2 || dp[j+1][i-1] == 1) {
    dp[j][i] = 1
}
```

最后 go 实现为:
```go
func longestPalindrome(s string) string {
    sLen := len(s)
    if sLen == 0 {
        return ""
    }
    left, len := 0, 1
    dp := make([][]int, sLen)
    for i := range dp {
        dp[i] = make([]int, sLen)
    }
    for i := 0; i < sLen; i++ {
        dp[i][i] = 1
        for j := 0; j < i; j++ {
            if (s[i] == s[j]) && (i-j < 2 || dp[j+1][i-1] == 1) {
                dp[j][i] = 1
            }
            if dp[j][i] == 1 && len < i-j+1 {
                len = i - j + 1
                left = j
            }
        }
    }
    return s[left : left+len]
}
```