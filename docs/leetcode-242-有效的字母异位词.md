# leetcode 242 有效的字母异位词
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

题解：

该题比较简单，从排序的思维来看的话，将字符串转化为 int 数组之后，排序完成就能得到从小到大排序好的数组。

然后将两个排序好的数组进行一一比较之后便可知道两个字符串是否是字母异位词了。

以下是实现：

```go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false
    }
    sCh, tCh := make([]int, 0), make([]int, 0)
    for _, ch := range s {
        sCh = append(sCh, int(ch))
    }
    for _, ch := range t {
        tCh = append(tCh, int(ch))
    }
    sSort := QuickSort{
        Value: sCh,
    }
    tSort := QuickSort{
        Value: tCh,
    }
    sSort.Sort()
    tSort.Sort()
    for index := 0; index < len(sSort.Value); index++ {
        if sSort.Value[index] != tSort.Value[index] {
            return false
        }
    }
    return true
}

type QuickSort struct {
    Value []int
}

func (q *QuickSort) Sort() () {
    if len(q.Value) <= 1 {
        return
    }
    quickSort(q.Value, 0, len(q.Value)-1)
}

func quickSort(s []int, left, right int) {
    temp, index := s[left], left
    i, j := left, right
    for j >= i {
        // 比temp大的放在右边
        for j >= index && s[j] >= temp {
            j--
        }
        if j >= index {
            s[index] = s[j]
            index = j
        }
    
        // 比temp小的放在左边
        for index >= i && s[i] <= temp {
            i++
        }
        if index >= i {
            s[index] = s[i]
            index = i
        }
    }
    s[index] = temp
    if index-left > 1 {
        quickSort(s, left, index-1)
    }
    if right-index > 1 {
        quickSort(s, index+1, right)
    }
}
```