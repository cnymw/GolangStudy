# leetcode 242 有效的字母异位词

## 题目

给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。

示例 1:
```go
输入: s = "anagram", t = "nagaram"
输出: true
```


示例 2:
```go
输入: s = "rat", t = "car"
输出: false
```

说明:你可以假设字符串只包含小写字母。

## 题解

### 方法1：排序

#### 算法

从排序的思维来看的话，将字符串转化为 byte 数组之后，排序完成就能得到排序好的数组。

然后将两个排序好的数组进行一一比较之后便可知道两个字符串是否是字母异位词了。

这里用到了 Go 标准库中的 sort，bytes 库，sort.Sort 用于排序两个 byte 数组，bytes.Equal 用于判断两个数组是否相等

```go
import (
	"bytes"
	"sort"
)

type Bytes []byte

func isAnagram(s string, t string) bool {
	if len(s) != len(t) {
		return false
	}
	sBytes, tBytes := Bytes(s), Bytes(t)
	sort.Sort(sBytes)
	sort.Sort(tBytes)
	return bytes.Equal(sBytes, tBytes)
}

func (s Bytes) Len() int {
	return len(s)
}

func (s Bytes) Less(i, j int) bool {
	return s[i] < s[j]
}

func (s Bytes) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}
```

#### 复杂度分析

- 时间复杂度：O(nlgn)，该算法的时间复杂度取决于 sort 库的排序算法效率，排序算法根据不同状况会使用最优的算法，总体的效率为 O(nlgn)
- 空间复杂度：>O(n)，因为会新建 byte 数组，所以至少会有 O(n) 空间的损耗，总的空间复杂度取决于 sort 库所使用的排序算法，例如使用了堆排序，那么空间复杂度为 O(n)+O(1)。

### 方法2：哈希表

#### 算法

如果两者所有字符的出现频率都是相等的，也能判断出两个字符串是有效的字母异位词。

因为字符串只包含 26 个字母，所以定义一个大小为 26 的哈希表来存放出现字母的频率，如果在 s 中出现了单词，那么单词序列对应的频率加一，在 t 中出现的单词序列对应频率减一。

最后统计哈希表所有字母序列对应频率为 0 即可。

```go
func isAnagram(s string, t string) bool {
	if len(s) != len(t) {
		return false
	}
	counter := make([]int, 26)
	for i := 0; i < len(s); i++ {
		counter[s[i]-'a']++
		counter[t[i]-'a']--
	}
	for i := range counter {
		if counter[i] != 0 {
			return false
		}
	}
	return true
}
```

#### 复杂度分析

- 时间复杂度：O(n)，遍历字符串需要消耗 O(n) 时间，最后遍历哈希表 26 个字母的频率消耗 O(1) 时间，总体消耗 O(n)
- 空间复杂度：O(1)，存放哈希表的空间只需要一个长度 26 的数组即可，所以复杂度为 O(1)