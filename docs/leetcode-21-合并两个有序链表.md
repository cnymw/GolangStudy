# leetcode 21 合并两个有序链表

## 题目

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

示例：

```text
输入: 1->2->4, 1->3->4
输出: 1->1->2->3->4->4
```

## 题解

该题目主要考察的是链表的遍历，首先构造一个新的链表作为返回的结果，然后遍历输入的两个链表，将数字较小的值添加到新的链表中。

这里需要注意的是需要定义一个指针用于遍历新的链表，最终返回的是伪造的 head 的 next 指针。

![数据结构-链表-leetcode21.png](https://cnymw.github.io/GolangStudy/docs/img/数据结构-链表-leetcode21.png)

实现如下：

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    cur1, cur2 := l1, l2
    result := &ListNode{}
    cur := result
    for cur1 != nil && cur2 != nil {
        if cur1.Val >= cur2.Val {
            cur.Next = cur2
            cur = cur.Next
            cur2 = cur2.Next
        } else {
            cur.Next = cur1
            cur = cur.Next
            cur1 = cur1.Next
        }
    }
    if cur1 != nil {
        cur.Next = cur1
    } else if cur2 != nil {
        cur.Next = cur2
    }
    return result.Next
}
```