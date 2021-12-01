# Go 源码解读-二叉堆 heap

（二叉）堆是一个数组，它可以被看成是一个近似的完全二叉树。

## 包 package

```go
// 包 heap 为实现 heap.Interface 接口的所有类型提供堆操作。
// 堆是一棵树，特性是每个节点都是其子树中的最小值。
//
// 树中最小的元素是根节点，索引为 0。
//
// 堆是实现优先队列的常用方法。
// 要创建一个优先队列，实现一个具有使用（负的）优先级作为比较的依据的 Less 方法的 Heap 接口，如此一来可用 Push 添加项目而用 Pop 取出队列最高优先级的项目。
// 例子里包含如下实现：文件 example_pq_test.go 包含完整的源代码。
package heap
```

## 接口 Interface

```go
// Interface 描述了使用此包中的一系列方法所需要的实现的类型。
// 实现 Interface 的任何类型都可以称为一个最小堆（堆在调用 Init 后或数据为空或排序时建立），这个堆具有如下的特性
//
//	!h.Less(j, i) for 0 <= i < h.Len() and 2*i+1 <= j <= 2*i+2 and j < h.Len()
//
// 请注意，此接口中的 Push 和 Pop 用于被堆的实现来调用。
// 要从堆中添加和删除数据，请使用 heap.Push 和 heap.Pop
type Interface interface {
	sort.Interface
	Push(x interface{}) // add x as element Len()
	Pop() interface{}   // remove and return element Len() - 1.
}
```