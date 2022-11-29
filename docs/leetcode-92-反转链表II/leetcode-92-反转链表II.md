## 题目

给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置 `left` 到位置 `right` 的链表节点，返回反转后的链表 。

示例 1：

```text
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```

示例 2：

```text
输入：head = [5], left = 1, right = 1
输出：[5]
```

## 题解

### 解法 1：利用数组特性

> [GolangStudy Online Courses：数据结构-链表](https://golangstudy.tech/course/golang/数据结构-链表)

通过链表的特性我们可以知道，链表不支持随机读写，所以没办法将指定位置的两个节点进行对调。

但是我们了解数组或者称作顺序表，是支持随机读写的，可以根据索引位置将两个节点进行对调。

于是，我们可以用如下步骤来翻转链表：

1. 创建数组，将链表数据同步到数组
2. 遍历数组，从索引位置 `left` 遍历到 `(right-left+1)/2`，对调 `left+i-1` 和 `right-i-1` 的数据，实现局部反转的效果
3. 将链表数据同步回到链表

```go
func reverseBetween(head *ListNode, left int, right int) *ListNode {
	// 创建临时数组，用于链表数据同步到数组
	array := make([]int, 0)
	cur := head

	// 通过 cur 遍历链表，将链表数据同步到数组
	for cur != nil {
		array = append(array, cur.Val)
		cur = cur.Next
	}

	// 利用数组特性：随机读取，通过该特性可以将指定的两个数组元素进行调换
	for i := 0; i < (right-left+1)/2; i++ {
		array[left+i-1], array[right-i-1] = array[right-i-1], array[left+i-1]
	}

	cur = head
	// 通过 cur 遍历链表，将数组元素同步到链表
	for i := 0; i < len(array); i++ {
		cur.Val = array[i]
		cur = cur.Next
	}
	return head
}
```