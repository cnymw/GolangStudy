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

## 堆操作 up

为了建立一个最小堆，如果一个节点比它的父节点小，那么需要将它同父节点交换位置。

```go
func up(h Interface, j int) {
	for {
		// 取父节点的索引
		i := (j - 1) / 2
		
		// i==j：如果当前是根节点
		// !h.Less(j, i)：如果子节点的值大于父节点的值
		// 如果当前是根节点或者子节点的值大于父节点的值，那么 break
		if i == j || !h.Less(j, i) {
			break
		}
		
		// 交换子节点和父节点的索引
		h.Swap(i, j)
		
		// 继续向上循环，直到根结点
		j = i
	}
}
```

## 堆操作 down

和 up 相对应的，如果三个节点（一个父节点+两个子节点）中，父节点不是最小的，那么需要将最小的值交换到父节点，组成一个由三个节点组成的最小堆。

这个操作也称作堆化（heapify）

```go
func down(h Interface, i0, n int) bool {
	i := i0
	for {
		// 取左子节点的索引
		j1 := 2*i + 1
		
		// 如果左子节点越界了，大于数组的长度或者小于0
		if j1 >= n || j1 < 0 { 
			break
		}
		j := j1
		
		// 取左右子节点中值更小的节点
		if j2 := j1 + 1; j2 < n && h.Less(j2, j1) {
			j = j2
		}
		
		// 如果父节点的值大于最小的子节点的值，那么break
		if !h.Less(j, i) {
			break
		}
		
		// 将父节点和最小的子节点的索引进行替换，保证父节点是三者中最小的值
		h.Swap(i, j)
		
		// 继续向下循环，直到到叶节点
		i = j
	}
	return i > i0
}

```

## 初始化 Init

```go
// Init 建立包中其他方法所需的堆特性：堆中某个结点的值总是不大于或不小于其父结点的值
// Init 对于堆特性是幂等的，并且可以在堆特性失效时调用。
// 复杂度为 O(n)，其中 n=h.Len()
func Init(h Interface) {
	// 将具有子节点的节点依次堆化，建立最小堆
	n := h.Len()
	for i := n/2 - 1; i >= 0; i-- {
		down(h, i, n)
	}
}
```

## 插入接口 Push

```go
// Push 将元素 x 插入到堆里。
// 复杂度为 O(logn)，其中 n=h.Len()
func Push(h Interface, x interface{}) {
	h.Push(x)
	up(h, h.Len()-1)
}
```

## 弹出接口 Pop

```go
// Pop 从堆中移除并返回最小元素（根据 Less 判断大小）。
// 复杂度为 O(logn)，其中 n=h.Len()
// Pop 相当于 Remove(h,0)
func Pop(h Interface) interface{} {
	n := h.Len() - 1
	h.Swap(0, n)
	down(h, 0, n)
	return h.Pop()
}
```

## 移除接口 Remove

```go
// Remove 从堆中移除并返回索引 i 处的元素。
// 复杂度为 O(logn)，其中 n=h.Len()
func Remove(h Interface, i int) interface{} {
	n := h.Len() - 1
	if n != i {
		h.Swap(i, n)
		if !down(h, i, n) {
			up(h, i)
		}
	}
	return h.Pop()
}
```

## Fix

```go
// Fix 在索引 i 处的元素更改其值后重新建立堆顺序。
// 更改索引 i 处的值，然后调用 Fix 相当于调用 Remove(h,i)，然后 Push 新的值，但成本 Fix 更低。
// 复杂度为 O(logn)，其中 n=h.Len()
func Fix(h Interface, i int) {
	if !down(h, i, h.Len()) {
		up(h, i)
	}
}
```