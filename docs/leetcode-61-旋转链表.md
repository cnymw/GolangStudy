# leetcode 61 旋转链表

## 题目

给你一个链表的头节点 head ，旋转链表，将链表每个节点向右移动 k 个位置。

示例 1：

![leetcode-61-旋转链表-示例1.jpg](https://cnymw.github.io/GolangStudy/docs/img/leetcode-61-旋转链表-示例1.jpg)

```text
输入：head = [1,2,3,4,5], k = 2
输出：[4,5,1,2,3]
```

## 题解

### 方法1：循环链表

读题后，可以发现有如下几个问题：

1. 如何将尾部的节点和头部的节点连接起来？即如何实现 5->1 的连接？

2. 如何实现头部节点向右移动 k 个位置？

3. 如果 k 的大小比链表大小大该怎么办？

根据链表种类的了解，可以知道问题 1 可以通过循环链表的方式解决，将链表的尾部 tail 的 Next 指向链表的头部 head，即可实现 5->1 的连接了。

换个角度思考问题 2，即可发现头部节点向右移动 k 个位置实际上是找出一个正确的新的头部节点，例如向右移动 1 个位置，即是找到头部为 5，然后将 4 的 Next 置为 nil 即可。

问题 3 是代码实现里需要注意的细节，代码实现如下：

```go
func rotateRight(head *ListNode, k int) *ListNode {
	if head == nil {
		return head
	}

	tail, count := head, 1

	// 遍历链表以指向尾部 tail
	for tail.Next != nil {
		tail = tail.Next
		count++
	}

	// 将尾部 tail 的 Next 指向头部 head，以实现单向循环链表结构
	tail.Next = head

	// 防止移动数大于链表节点数量
	k = k % count

	k = count - 1 - k
	node := head

	// 找到新的头部节点的前一个节点
	for i := 0; i < k; i++ {
		node = node.Next
	}
	// 设置链表的新的头部节点
	head = node.Next
	// 将链表新的尾部节点的 Next 置为空，将单向循环链表重新设置为单向链表
	node.Next = nil
	return head
}
```