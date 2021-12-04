# leetcode 2 两数相加

## 题目

给你两个非空的链表，表示两个非负的整数。它们每位数字都是按照逆序的方式存储的，并且每个节点只能存储一位数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例 1：

```text
输入: l1 = [2,4,3], l2 = [5,6,4]
输出: [7,0,8]
解释: 342 + 465 = 807.
```

示例 2：

```text
输入: l1 = [0], l2 = [0]
输出: [0]
```

示例 3：

```text
输入: l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出: [8,9,9,9,0,0,0,1]
```

## 题解

### 方法1：链表

传入的是两个链表，需要得出链表所对应的数值需要遍历链表，不过这个遍历可以同时进行，也就是一次循环可以同时取两个链表的节点。

同时遍历两个链表，取出节点的值，进行相加，如果结果大于等于 10，那么需要进位，加到下一次遍历的结果中去。

相加结果 answer=(n1+n2+carry)mod10，进位 carry=answer/10

如果两个链表长度不同，可以认为较短的链表后面有若干个 0。

此外，遍历结束后，如果 carry>0，还需要在结果后面添加个值，值为 carry。

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) (head *ListNode) {
	var tail *ListNode
	carry := 0

	for l1 != nil || l2 != nil {
		n1, n2 := 0, 0
		if l1 != nil {
			n1 = l1.Val
			l1 = l1.Next
		}
		if l2 != nil {
			n2 = l2.Val
			l2 = l2.Next
		}

		sum := n1 + n2 + carry
		sum, carry = sum%10, sum/10

		if head == nil {
			head = &ListNode{Val: sum}
			tail = head
		} else {
			tail.Next = &ListNode{Val: sum}
			tail = tail.Next
		}
	}
	if carry != 0 {
		tail.Next = &ListNode{Val: carry}
	}
	return
}
```