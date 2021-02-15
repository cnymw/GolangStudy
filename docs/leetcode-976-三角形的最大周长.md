# leetcode 976 三角形的最大周长

## 题目

给定由一些正数（代表长度）组成的数组 A，返回由其中三个长度组成的、面积不为零的三角形的最大周长。

如果不能形成任何面积不为零的三角形，返回 0。


示例 1：
```text
输入：[2,1,2]
输出：5
```

示例 2：
```text
输入：[1,2,1]
输出：0
```

示例 3：
```text
输入：[3,2,3,4]
输出：10
```

示例 4：
```text
输入：[3,6,2,3]
输出：8
```

## 题解

### 方法1：排序

#### 算法

首先，复习一下能够组成三角形的三边特性：两边之和大于第三边，两边之差小于第三边。

两边之和大于第三边重点在于找到最大的第三边，两边之差也是需要找到最大的第三边来执行相减操作。

设边 b<a,c<a,所以 b-c 是一定小于 a 的，不需要做判断。

将序列按照倒序排列，每三个数做一次判断，从大到小进行遍历，可以找到三角形的最大周长。

```go
import (
	"sort"
)

type Arr []int

func largestPerimeter(A []int) int {
	s := Arr(A)

	sort.Sort(s)

	for i := 0; i <= len(s)-3; i++ {
		if judgeTriangle(s[i], s[i+1], s[i+2]) {
			return s[i] + s[i+1] + s[i+2]
		}
	}
	return 0
}

func judgeTriangle(a, b, c int) bool {
	if a < b+c && a-b < c && a-c < b {
		return true
	} else {
		return false
	}
}

func (s Arr) Len() int {
	return len(s)
}

func (s Arr) Less(i, j int) bool {
	return s[i] > s[j]
}

func (s Arr) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}
```

#### 复杂度分析

- 时间复杂度：O(nlgn)，时间复杂度取决于 sort 库选择的排序算法的效率
- 空间复杂度：取决于 sort 库选择排序算法