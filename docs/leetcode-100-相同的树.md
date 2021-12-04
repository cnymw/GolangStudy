# leetcode 100 相同的树

## 题目

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

示例 1:

```text
输入:       1         1
          / \       / \
         2   3     2   3

        [1,2,3],   [1,2,3]

输出: true
```

示例 2:

```text
输入:      1          1
          /           \
         2             2

        [1,2],     [1,null,2]

输出: false
```

示例 3:

```text
输入:       1         1
          / \       / \
         2   1     1   2

        [1,2,1],   [1,1,2]

输出: false
```

## 题解

这道题考察的仍然是树的前序遍历，树的前序遍历实现方式有两种：递归和栈。因为是两个树的同步遍历，所以在这里使用栈来实现比较方便。实现逻辑如下所示：

1. 将两个树的根结点分别放入到两个栈内
2. 如果两个栈都不为空，遍历栈
    1. 弹出 top 结点，对比两个 top 的值，如果不相等返回 false
    2. 如果 top 结点的两个子结点不相等，返回 false ，也就是 top1.left != top2.left 或者 top1.right != top2.right 的时候，返回false
    3. 将 top 结点的子结点放入栈内
3. 结束遍历栈后，如果还有一个栈不为空的话，那么返回 false

go 实现如下：
```go
func isSameTree(p *TreeNode, q *TreeNode) bool {
    if p == nil && q == nil {
        return true
    }
    pstack, qstack := New(), New()
    
    if p != nil {
        pstack = pstack.Push(p)
    }
    if q != nil {
        qstack = qstack.Push(q)
    }
    
    var ppop, qpop interface{}
    for !pstack.Empty() && !qstack.Empty() {
        pstack, ppop = pstack.Pop()
        qstack, qpop = qstack.Pop()
    
        ptemp, qtemp := ppop.(*TreeNode), qpop.(*TreeNode)
        if ptemp.Val != qtemp.Val {
            return false
        }
    
        if (ptemp.Left == nil && qtemp.Left != nil) || (ptemp.Left != nil && qtemp.Left == nil) {
            return false
        } else {
            if ptemp.Left != nil && qtemp.Left != nil {
                pstack = pstack.Push(ptemp.Left)
                qstack = qstack.Push(qtemp.Left)
            }
        }
    
        if (ptemp.Right == nil && qtemp.Right != nil) || (ptemp.Right != nil && qtemp.Right == nil) {
            return false
        } else {
            if ptemp.Right != nil && qtemp.Right != nil {
                pstack = pstack.Push(ptemp.Right)
                qstack = qstack.Push(qtemp.Right)
            }
        }
    }
    
    if !pstack.Empty() || !qstack.Empty() {
        return false
    }
    return true
}

type stack []interface{}

func New() stack {
    s := make(stack, 0)
    return s
}
func (s stack) Push(v interface{}) stack {
    return append(s, v)
}

func (s stack) Pop() (stack, interface{}) {
    if s.Len() == 0 {
        return s, nil
    }
    return s[:s.Len()-1], s[s.Len()-1]
}

func (s stack) Len() int {
    return len(s)
}

func (s stack) Peek() interface{} {
    return s[s.Len()-1]
}

func (s stack) Empty() bool {
    return s.Len() == 0
}
```