# leetcode 3 无重复字符的最长子串

## 题目 

给定一个字符串，请你找出其中不含有重复字符的最长子串的长度。

示例 1:

```text
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc",所以其长度为 3
```

示例 2:

```text
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b",所以其长度为 1
```

示例 3:

```text
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke",所以其长度为 3
```

## 题解

### 方法1：哈希表

因为需要判断是否有重复的字符，所以需要将字符放入哈希表中，以提升判断重复的效率。

其次需要知道子串的长度，所以需要有一个子串来记录满足条件的子串。

所以大致的逻辑是：

```text
len = 1
foreach char in s {
    if char in map{
        // sub 减去之前重复的 char
        sub -= map[char]
        // sub 加上新的 char
        sub += char
        // 重新给哈希表赋值，sub 里的字符全部需要添加到 map 中去
        refresh(map)
    }else{
        // sub 加上新的 char
        sub += char
        // 给哈希表赋值
        map[char]
        if len(sub)>len{
            len = len(sub)
        }
    }
}
```

go 的实现如下：

```go
// 哈希表
func lengthOfLongestSubstring(s string) int {
    if len(s) == 0 {
        return 0
    }
    m, length := make(map[rune]int, 0), 1
    sub := make([]int32, 0)
    runes := []rune(s)
    for index, r := range s {
        if i, ok := m[r]; ok {
            m = make(map[rune]int, 0)
            sub = runes[i+1 : index]
            sub = append(sub, r)
    
            for j, k := index-len(sub)+1, 0; j <= index && k < len(sub); {
                m[sub[k]] = j
                k++
                j++
            }
        } else {
            sub = append(sub, r)
            if len(sub) > length {
                length = len(sub)
            }
            m[r] = index
        }
    }
    return length
}
```

### 方法2：滑动窗口

> 算法-滑动窗口介绍：[滑动窗口](/docs/算法-滑动窗口.md)

这个问题可以使用滑动窗口技术：

1. 我们使用线性循环计算 n 项中前 k 个元素的和，并将其存储在变量 window_sum 中

2. 然后我们将在数组上线性求和，直到子窗口达到末端，同时随时监控最大和

3. 为了获得 k 个元素块的当前和，只需从上一个块中减去第一个元素，然后添加当前块的最后一个元素

根据滑动窗口技术，我们可以定义两个指针 i，rk 表示窗口的左右边界，其中左指针代表线性循环的起点，右指针为满足条件的 k 个元素的末尾。

在每次遍历时，我们会将左指针向右移动一格，表示我们开始枚举下一个字符作为起始位置，然后我们可以不断向右移动右指针，直到子串中存在重复的字符。在移动结束后，这个子串对应满足条件的子串。我们记下这个子串的长度。

在枚举结束后，我们找到的最长的子串即为答案。

至此，我们能够根据以上逻辑实现代码：

```go
// 滑动窗口
func lengthOfLongestSubstringByWindowSliding(s string) int {
	n := len(s)
	m := map[byte]int{}

	rk, ans := -1, 0
	for i := 0; i < n; i++ {
		// 滑动到下一个节点，将前一个节点删除
		if i > 0 {
			delete(m, s[i-1])
		}

		// 如果满足条件(无重复字符)，那么会持续往后面滑动
		for rk+1 < n && m[s[rk+1]] == 0 {
			m[s[rk+1]]++
			rk++
		}

		ans = int(math.Max(float64(ans), float64(rk+1-i)))
	}

	return ans
}
```