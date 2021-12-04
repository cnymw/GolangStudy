# leetcode 15 三数之和

## 题目

给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]

## 题解 

- 标签：数组遍历
- 首先对数组进行排序，排序后固定一个数 nums[i]，再使用左右指针指向 nums[i] 后面的两端，数字分别为 nums[L] 和 nums[R]，计算三个数的和 sum 判断是否满足为 0，满足则添加进结果集
- 如果 nums[i] 大于 0，则三数之和必然无法等于 0，结束循环
- 如果 nums[i] == nums[i-1]，则说明该数字重复，会导致结果重复，所以应该跳过
- 当 sum == 0 时，nums[L] == nums[L+1] 则会导致结果重复，应该跳过，L++
- 当 sum == 0 时，nums[R] == nums[R-1] 则会导致结果重复，应该跳过，R--
- 时间复杂度：O(n^2)，n 为数组长度

在这里用到了 go sdk 中自带的 sort 包，用于实现对数组的排序。

```go
import "sort"

type Interval struct {
    Value []int
}

func threeSum(nums []int) [][]int {
    result := make([][]int, 0)
    if nums == nil || len(nums) < 3 {
        return result
    }
    interval := new(Interval)
    interval.Value = nums
    sort.Sort(interval)
    
    for index := 0; index < interval.Len(); index++ {
        if interval.Value[index] > 0 {
            break
        }
        if index >= 1 && interval.Value[index] == interval.Value[index-1] {
            continue
        }
        for L, R := index+1, interval.Len()-1; L < R; {
            sum := interval.Value[index] + interval.Value[L] + interval.Value[R]
            if sum == 0 {
                tmp := []int{interval.Value[index], interval.Value[L], interval.Value[R]}
                result = append(result, tmp)
                for ; L+1 < interval.Len() && interval.Value[L] == interval.Value[L+1]; L++ {
    
                }
                for ; R-1 >= 0 && interval.Value[R] == interval.Value[R-1]; R-- {
    
                }
                L++
                R--
                continue
            } else if sum > 0 {
                R--
                continue
            } else {
                L++
                continue
            }
        }
    }
    return result
}

func (interval *Interval) Len() int {
    return len(interval.Value)
}

func (interval *Interval) Less(i, j int) bool {
    return interval.Value[i] < interval.Value[j]
}

func (interval *Interval) Swap(i, j int) {
    interval.Value[i], interval.Value[j] = interval.Value[j], interval.Value[i]
}
```