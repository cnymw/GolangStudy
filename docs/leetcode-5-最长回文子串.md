# leetcode 5 最长回文子串

## 题目

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

```text
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案
```

示例 2：

```text
输入: "cbbd"
输出: "bb"
```

## 题解

以输入"babad"为例，画出状态图:

![状态图](https://cnymw.github.io/GolangStudy/docs/img/算法-动态规划-leetcode5状态图.png)

由此总结出以下状态转移方程(伪代码):

```go
if i == j
    dp[i,j]=1
else if s[i]==s[j] && i==j-1
    dp[i,j]=1
else if s[i]==s[j] && i<=j-2 && dp[i+1][j-1]
    dp[j,i]=1
```

最后 go 实现为:

```go
func longestPalindrome(s string) string {
	sLen := len(s)
	if sLen == 0 {
		return ""
	}
	left, maxlen := 0, 1

	dp := make([][]int, sLen)
	for i := 0; i < sLen; i++ {
		dp[i] = make([]int, sLen)
	}

	for j := 0; j < sLen; j++ {
		dp[j][j] = 1
		for i := 0; i < j; i++ {
			if s[i] == s[j] && (j-i <= 2 || dp[i+1][j-1] == 1) {
				dp[i][j] = 1
			}
			if dp[i][j] == 1 && maxlen < j-i+1 {
				left = i
				maxlen = j - i + 1
			}
		}
	}

	return s[left : left+maxlen]
}

```