# leetcode 1038 从二叉搜索树到更大和树

给出二叉搜索树的根节点，该二叉树的节点值各不相同，修改二叉树，使每个节点 node 的新值等于原树中大于或等于 node.val 的值之和。

提醒一下，二叉搜索树满足下列约束条件：

- 节点的左子树仅包含键小于节点键的节点。
- 节点的右子树仅包含键大于节点键的节点。
- 左右子树也必须是二叉搜索树。

示例：

![示例](https://cnymw.github.io/go-study/docs/img/数据结构-二叉查找树/数据结构-二叉查找树-示例1.png)

```go
输入:[4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]
输出:[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]
```

提示：

1. 树中的节点数介于 1 和 100 之间。
2. 每个节点的值介于 0 和 100 之间。
3. 给定的树为二叉搜索树。

题解：

根据示例不难发现，该题目的核心是遍历树，但是不是常见的前中后序遍历，而是一种改造过后的遍历方式，伪代码表示如下：

```go
INORDER-TREE-WALK(x,sum)
    if x!=nil{
    	INORDER-TREE-WALK(x.right,sum)
    	x.val += sum
    	sum = x.val
    	INORDER-TREE-WALK(x.left,sum)
    }
```

在这里 sum 变量记录了之前遍历结点的值的总和，然后加上当前结点的值便是题目所需要的和。

```go
func bstToGst(root *TreeNode) *TreeNode {
    sum := 0
    inorderTreeWalk(root, &sum)
    return root
}

func inorderTreeWalk(node *TreeNode, sum *int) {
    if node != nil {
        inorderTreeWalk(node.Right, sum)
        if *sum != 0 {
            node.Val += *sum
        }
        *sum = node.Val
        inorderTreeWalk(node.Left, sum)
    }
}
```
