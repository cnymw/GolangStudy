# leetcode 3 无重复字符的最长子串

## 题目 

给定一个字符串，请你找出其中不含有重复字符的最长子串的长度。

示例 1:

```go
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc",所以其长度为 3
```

示例 2:

```go
输入: "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b",所以其长度为 1
```

示例 3:

```go
输入: "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke",所以其长度为 3
```

## 题解

因为需要判断是否有重复的字符，所以需要将字符放入哈希表中，以提升判断重复的效率。

其次需要知道子串的长度，所以需要有一个子串来记录满足条件的子串。

所以大致的逻辑是：
```go
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