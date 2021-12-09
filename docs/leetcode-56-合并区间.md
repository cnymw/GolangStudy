# leetcode 56 合并区间

## 题目

给出一个区间的集合，请合并所有重叠的区间。

示例 1:

```text
输入: [[1,3],[2,6],[8,10],[15,18]]
输出: [[1,6],[8,10],[15,18]]
解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

示例 2:

```text
输入: [[1,4],[4,5]]
输出: [[1,5]]
解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间
```

## 题解

针对于输入 s = [1,3], t = [2,6]，可以看出 s[1] > t[0]，存在重叠区间，所以 s 和 t 合并之后，左边界取s[0],t[0]中最小的为1，右边界取 s[1] , t[1] 中最大的为6，所以结果为 [1,6]

针对于输入 s = [2,3], t = [1,10]，可以看出 s[1] > t[0]，存在重叠区间，所以 s 和 t 合并之后，左边界取 1，右边界取 10，所以结果为 [1,10]

可以看出，当 s[1] > t[0] 时，即存在重叠区间，左边界取最小值，右边界取最大值。

所以最快的方式是将所有输入按首个数字进行排序，然后再比较 s[1],t[0] 的大小，这样能够保证 s[0] < t[0] 为起始条件，省去了一步判断最小的过程。

以下是实现：

```go
import (
    "math"
    "sort"
)

type Interval struct {
    Value [][]int
}

func merge(intervals [][]int) [][]int {
    if len(intervals) <= 1 {
        return intervals
    }
    interval := new(Interval)
    interval.Value = intervals
    sort.Sort(interval)
    
    result := make([][]int, 0)
    result = append(result, interval.Value[0])
    for i := 1; i < len(interval.Value); i++ {
        if result[end(result)][1] < interval.Value[i][0] {
            result = append(result, interval.Value[i])
        } else {
            result[end(result)][1] = int(math.Max(float64(result[end(result)][1]), float64(interval.Value[i][1])))
        }
    }
    return result
}

func end(i [][]int) int {
    return len(i) - 1
}

func (interval *Interval) Len() int {
    return len(interval.Value)
}

func (interval *Interval) Less(i, j int) bool {
    return interval.Value[i][0] < interval.Value[j][0]
}

func (interval *Interval) Swap(i, j int) {
    interval.Value[i], interval.Value[j] = interval.Value[j], interval.Value[i]
}
```