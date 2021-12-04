# leetcode 53 最大子序和

## 题目

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:

输入: 
```text
[-2,1,-3,4,-1,2,1,-5,4]
```

输出: 
```text
6
```
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。

## 题解

下面讲两种动规思路:

第一种是从动规概念上面去分析
- 动态规划的是首先对数组进行遍历，当前最大连续子序列和为 sum，结果为 ans
- 如果 sum > 0，则说明 sum 对结果有增益效果，则 sum 保留并加上当前遍历数字
- 如果 sum <= 0，则说明 sum 对结果无增益效果，需要舍弃，则 sum 直接更新为当前遍历数字
- 每次比较 sum 和 ans的大小，将最大值置为ans，遍历结束返回结果
- 时间复杂度：O(n)

第二种是从状态转移方程角度去分析

用 dp[i] 表示以i结尾的最大子序列和。初始值 dp[0] = nums[0]，然后从第二个数开始遍历

- if 当前数加上前一个最大序列和大于当前数，则将当前数加到序列和中，nums[i] + dp[i-1] > nums[i]，则 dp[i] = nums[i] + dp[i-1];
- else 以当前数结尾的最大序列和即为当前数本身 dp[i] = nums[i]

```go
func maxSubArray(nums []int) int {
    dp := make([]int, len(nums))
    dp[0] = nums[0]

    for index := 1; index < len(nums); index++ {
        dp[index] = int(math.Max(float64(dp[index-1]+nums[index]), float64(nums[index])))
    }
    
    max := dp[0]
    for index := 1; index < len(nums); index++ {
        if dp[index] > max {
            max = dp[index]
        }
    }
    return max
}
```