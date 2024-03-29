# 数据结构面试题：B 树和 B+ 树的区别

## B 树

B 树被称为自平衡树，它的节点按中序遍历进行排序。在 B 树中，一个节点可以有两个以上的子节点。

B 树必须满足以下条件：

- B 树的所有叶节点必须处于同一级别

- 在 B 树的叶节点上方，不应该有空的子树

- 树的高度应该尽可能的低

## B+ 树

B+ 树仅在树的叶节点处存储数据指针，这样消除了 B 树的缺点，可以用于数据库索引。也因此，B+ 树的叶节点结构和 B 树的内部节点结构完全不同。

B+ 树由于数据指针仅存储于叶节点，因此叶节点必须存储所有的键值以及其到磁盘文件块对应的数据指针。此外，叶节点之间也有一条链表来提供对记录的有序访问。

因此，B+ 树的叶节点构成索引的第一级，内部节点构成多级索引的其他级别。叶节点的键值也出现在内部节点中，但是只作为用于搜索记录的索引。

![数据结构-面试题-B树和B+树的区别-B+树.jpg](https://cnymw.github.io/GolangStudy/docs/img/数据结构-面试题-B树和B+树的区别-B+树.jpg)

## B 树和 B+ 树的区别

B树|B+树
---|:--:
所有的内部节点和叶节点都有数据指针|只有叶节点有数据指针
由于叶节点并非拥有所有的键，搜索通常需要更多时间|所有键都位于叶节点，因此搜索速度更快更准确
B树不保留键的副本|所有节点的副本都被储存在叶节点中
插入需要更多的时间，有时是不可预测的|插入更容易，结果总是一样的
内部节点的删除是非常复杂的，树必须经历很多旋转|删除任何节点都很容易，因为所有节点都位于叶节点
叶节点不会存储为链表结构|叶节点存储为链表结构
没有冗余的键|可能存在冗余的键