# leetcode 349 两个数组的交集

## 题目

给定两个数组，编写一个函数来计算它们的交集。

示例 1:

```text
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2]
```

示例 2:

```text
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [9,4]
```

说明:

- 输出结果中的每个元素一定是唯一的。
- 我们可以不考虑输出结果的顺序。

## 题解

### 方法1：哈希表

#### 算法

在题目中，取两个数组的交集有以下三点要求：

- 去重
- 取交集
- 返回结果仍然是数组

golang 标准库没有自带 set 数据结构的实现，所以去重，取交集等功能需要 map 来实现。

在这里 nums1Map 记录的是 nums1 中出现数字的频率，nums2Map 记录了 nums1 和 nums2 之间交集，只有当 nums1 和 nums2 同时出现该数字，才会将该数字记录到 nums2Map 中。

在获取结果时，只获取第二个 map 的 key 值，忽略了 map 中 value 的值，也就是忽略了数字出现的频率，这样就能做到去重的效果。

```go
func intersection(nums1 []int, nums2 []int) []int {
	nums1Map, nums2Map := make(map[int]int), make(map[int]int)
	for _, v := range nums1 {
		nums1Map[v]++
	}
	for _, v := range nums2 {
		if nums1Map[v] > 0 {
			nums2Map[v]++
		}
	}
	result := make([]int, 0)
	for k := range nums2Map {
		result = append(result, k)
	}
	return result
}
```

#### 复杂度分析

- 时间复杂度：O(n)，算法核心是三次循环，所以时间复杂度为 O(n)
- 空间复杂度：O(n)，算法空间消耗取决于 map 结构的空间消耗，golang 标准库中的 map 是通过拉链法来实现的，所以空间复杂度为 O(n)