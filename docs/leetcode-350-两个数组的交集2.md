# leetcode 350 两个数组的交集 II

## 题目

给定两个数组，编写一个函数来计算它们的交集。

示例 1:

```text
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2,2]
```

示例 2:

```text
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [4,9]
```

说明：

- 输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
- 我们可以不考虑输出结果的顺序。

## 题解

### 方法1：哈希表

#### 算法

在题目中，取两个数组的交集有以下两点要求：

- 取交集
- 返回结果仍然是数组

在这里 nums1Map 记录的是 nums1 中出现数字的频率，nums2Map 记录了 nums1 和 nums2 之间交集，只有当 nums1 和 nums2 同时出现该数字，才会将该数字记录到 nums2Map 中。

在获取结果时，取 nums1Map 和 nums2Map 中频率最小的值 v，来往结果数组中添加 v 次交集值。

```go
func intersect(nums1 []int, nums2 []int) []int {
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
	for k, v := range nums2Map {
		if nums1Map[k] < v {
			v = nums1Map[k]
		}
		for i := v; i > 0; i-- {
			result = append(result, k)
		}

	}
	return result
}

```

#### 复杂度分析

- 时间复杂度：O(n^2)，算法核心是三次循环，最后取结果时需要两层循环，所以时间复杂度为 O(n^2)
- 空间复杂度：O(n)，算法空间消耗取决于 map 结构的空间消耗，golang 标准库中的 map 是通过拉链法来实现的，所以空间复杂度为 O(n)