# leetcode 164 最大间距

## 题目

给定一个无序的数组，找出数组在排序之后，相邻元素之间最大的差值。

如果数组元素个数小于 2，则返回 0。

示例 1:

```text
输入: [3,6,9,1]
输出: 3
解释: 排序后的数组是 [1,3,6,9], 其中相邻元素 (3,6) 和 (6,9) 之间都存在最大差值 3
```

示例 2:
```text
输入: [10]
输出: 0
解释: 数组元素个数小于 2,因此返回 0
```

## 题解

找出相邻元素的差值，首先要将所有元素排好序，然后从按序计算差值。

然后将差值集合再按照大小进行排序，最后取最大的差值返回。

以下是 go 实现：
```go
func maximumGap(nums []int) int {
    if len(nums) < 2 {
        return 0
    }
    sort := &QuickSort{
        Value: nums,
    }
    sort.Sort()
    max := 0
    for index := 1; index < len(sort.Value); index++ {
        gap := sort.Value[index] - sort.Value[index-1]
        if gap > max {
            max = gap
        }
    }
    return max
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