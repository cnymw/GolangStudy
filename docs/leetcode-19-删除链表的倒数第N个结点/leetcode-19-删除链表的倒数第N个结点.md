## 题目

给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

示例 1：

```text
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

示例 2：

```text
输入：head = [1], n = 1
输出：[]
```

示例 3：

```text
输入：head = [1,2], n = 1
输出：[1]
```

## 题解

### 解法 1：使用 Golang SDK 链表

> [GolangStudy Online Courses：Go 源码解读-双向链表 list](https://golangstudy.tech/course/golang/go-源码解读-双向链表list)
> 
> [GolangStudy Online Courses：数据结构-链表](https://golangstudy.tech/course/golang/数据结构-链表)

根据链表分类，题目给到的 `ListNode` 数据结构属于单向链表，根据单向链表性质，只能从首节点单向向后 `next` 遍历到尾节点。

根据对双向链表性质的了解，可以知道双向链表支持从尾节点向前遍历 `prev` 到首节点。

那么题目中的删除链表倒数第 `n` 个节点，可以转化为从双向链表尾节点开始向前遍历 `prev` 遍历第 `n` 个节点。

同时，根据对 Golang SDK 的了解，可以知道 Golang SDK 原生支持双向链表实现，于是可以通过以下步骤实现题目：

1. 将 `ListNode` 同步到双向链表 `list`
2. 从 `list` 尾节点向前遍历 `n` 个节点
3. 调用 `list` 接口 `Remove`，移除第 2 步中取得的节点
4. 将 `list` 同步到 `ListNode`
5. 返回 `ListNode head` 节点

```go
import "container/list"

func removeNthFromEnd(head *ListNode, n int) *ListNode {
	// 通过使用 Golang SDK 新建一个双向链表
	l := list.New()
	// 将 ListNode 结构复制到链表里面
	for head != nil {
		l.PushBack(head.Val)
		head = head.Next
	}
	remove := l.Back()

	// 从双向链表的末尾向前数 n 个节点就是想要删除的节点
	for i := 1; i < n; i++ {
		remove = remove.Prev()
	}
	// 调用 list SDK 接口实现删除逻辑
	l.Remove(remove)

	// 将双向链表数据重新复制到 ListNode 结构
	if l.Len() > 0 {
		var newHead, tail *ListNode
		element := l.Front()
		newHead = &ListNode{Val: element.Value.(int)}
		tail = newHead
		// 从双向链表的首节点开始遍历
		for i := 1; i < l.Len(); i++ {
			element = element.Next()
			tail.Next = &ListNode{Val: element.Value.(int)}
			tail = tail.Next
		}
		return newHead
	} else {
		return nil
	}
}
```

### 解法 2：计算链表的长度

如果我们知道链表的长度 `length` 的话，我们就可以根据倒数第 `n` 个节点的信息，推出正数第 `m` 个节点。

也就是我们遍历到 `m=length-n+1` 个节点时，它就是我们需要删除的节点。

![leetcode-19-删除链表的倒数第N个结点-图示.png](https://cnymw.github.io/GolangStudy/docs/leetcode-19-删除链表的倒数第N个结点/leetcode-19-删除链表的倒数第N个结点-图示.png)

所以我们可以以下步骤删除倒数：

1. 遍历链表获取链表长度 `length`
2. 遍历到待删除节点的前一个节点，也就是 `cur=length-n` 个节点
3. 将 `cur.Next` 指向 `cur.Next.Next`，即可删除待删除的节点

```go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
	length := 0
	cur := head
	// 遍历 ListNode，获得链表长度 length
	for cur != nil {
		length++
		cur = cur.Next
	}
	dummy := &ListNode{Val: 0, Next: head}
	cur = dummy
	// 遍历到待删除节点的前一个节点
	for i := 1; i < length-n+1; i++ {
		cur = cur.Next
	}
	// 将待删除的节点的前一个节点的 Next 指向待删除的节点的 Next
	// 这样就将待删除节点给删除了
	// 例如 a->b-c 链表，
	// cur 为 a，
	// 将 a.Next 指向 c，
	// 也就是 cur.Next = b.next = cur.Next.Next
	cur.Next = cur.Next.Next
	return dummy.Next
}
```