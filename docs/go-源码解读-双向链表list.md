# Go 源码解读-双向链表 list

双向链表也叫双链表，是链表的一种，它的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。

所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点。

Golang sdk 中实现了双向链表，路径为`container/list/list.go`，我们可以学习一下 sdk 中的实现。

## 包 package

```go
// 包 list 实现了双向链表
//
// 以下是遍历双向链表的方法 (在这里 l 是一个 *List):
//	for e := l.Front(); e != nil; e = e.Next() {
//		// do something with e.Value
//	}
//
package list
```

## 结点 Element

```go
// Element 是链表里的结点
type Element struct {
	// next，prev 是双向链表中结点的后继和前驱结点。
	// 为了简化实现，在内部一个链表 l 被实现为一个循环链表，
	// 这样，&l.root 既是最后一个链表元素的后继（l.Back()），
	// 也是第一个链表元素的前驱（l.Front()）
	next, prev *Element

	// 结点所属的链表 list
	list *List

	// 结点里储存的值 Value
	Value interface{}
}
```

结点 Element 的方法只有两个：Next（）和 Prev（）

需要注意的是，Element 的方法都是包外可见的，意味着在使用双向链表时，会用到 Element 的两个方法，在后面的代码里可以看到，在遍历链表的时候，会用到下面两个方法。

```go
// Next 返回当前结点的后继或 nil
func (e *Element) Next() *Element {
	if p := e.next; e.list != nil && p != &e.list.root {
		return p
	}
	return nil
}
```

```go
// Prev 返回当前结点的前驱或 nil
func (e *Element) Prev() *Element {
	if p := e.prev; e.list != nil && p != &e.list.root {
		return p
	}
	return nil
}
```

## 双向链表 List

```go
// List 实现了双向链表
// List 的零值是一个空的链表
type List struct {
	// 链表的哨兵元素，仅会在 &root，root.prev，root.next 这三个地方使用到 
	root Element 
    
	// 不包含哨兵元素在内的当前链表长度
	len  int     
}
```

因为链表的长度不包含 root 元素，所以在初始化链表的时候链表的长度为 0。

```go
// 初始化链表，或清空链表
func (l *List) Init() *List {
	l.root.next = &l.root
	l.root.prev = &l.root
	l.len = 0
	return l
}
```

构造函数 New 会 new 一个 List 对象，并初始化。

```go
// New 返回一个初始化后的链表
func New() *List { return new(List).Init() }
```

```go
// Len 返回链表的结点数量。
// 时间复杂度为 O（1）
func (l *List) Len() int { return l.len }
```

因为 List 的实现是一个循环链表，所以 root 是链表第一个元素的前驱，root 也是链表最后一个元素的后继。

所以链表的 Front 和 Back 方法都是通过分别访问 root 元素的后继和前驱来实现的。

```go
// Front 返回链表的第一个元素，当链表为空时，返回 nil
func (l *List) Front() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.next
}

// Back 返回链表的最后一个元素，当链表为空时，返回 nil
func (l *List) Back() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.prev
}
```

为了防止链表为空，提供了一个内部方法来后续补偿链表初始化过程。

```go
// lazyInit 延迟初始化链表
func (l *List) lazyInit() {
	if l.root.next == nil {
		l.Init()
	}
}
```

在插入结点 e 到双向链表中的结点 at 时，需要注意：

- 插入结点 e 的 prev 和 next 的赋值

- 插入结点 e 的前驱结点 e.prev（实际上是 at）的后继的赋值

- 插入结点 e 的后继结点 e.next 的前驱的赋值

- 插入结点 e 的归属链表设置为 l

- 双向链表的长度加一

```go
// insert 在结点 at 之后插入结点 e，递增 l.len，返回结点 e
func (l *List) insert(e, at *Element) *Element {
	e.prev = at
	e.next = at.next
	e.prev.next = e
	e.next.prev = e
	e.list = l
	l.len++
	return e
}

// insertValue 对 insert(&Element{Value: v}, at) 做了一个较为方便的封装
func (l *List) insertValue(v interface{}, at *Element) *Element {
	return l.insert(&Element{Value: v}, at)
}
```

在移除结点 e 时，需要注意：

- 移除结点 e 的前驱结点 e.prev 的后继的赋值

- 移除结点 e 的后继结点 e.next 的前驱的赋值

- 移除结点 e 的 prev 和 next 要置为 nil，避免内存泄漏

- 双向链表的长度减一

```go
// remove 从链表中移除结点 e，递减 l.len，返回结点 e
func (l *List) remove(e *Element) *Element {
	e.prev.next = e.next
	e.next.prev = e.prev
	e.next = nil // 避免内存泄漏
	e.prev = nil // 避免内存泄漏
	e.list = nil
	l.len--
	return e
}
```

在移动结点 e 时，需要注意：

- 移动结点 e 的当前位置的 prev 和 next 的赋值，需要分别赋值为当前位置的前驱和后继，避免链表断裂

- 移动结点 e 的 prev 和 next 的赋值

- 移动结点 e 的前驱结点 e.prev（实际上是 at）的后继的赋值

- 移动结点 e 的后继结点 e.next 的前驱的赋值

```go
// move 移动结点 e 到结点 at 的位置，并返回结点 e
func (l *List) move(e, at *Element) *Element {
	if e == at {
		return e
	}
	e.prev.next = e.next
	e.next.prev = e.prev

	e.prev = at
	e.next = at.next
	e.prev.next = e
	e.next.prev = e

	return e
}
```

Remove 方法调用内部方法 remove 移除结点 e，理论上来说，结点 e 之后会被垃圾回收。

```go
// 如果 e 是链表 l 的结点，Remove 方法从链表 l 中移除结点 e
// 方法返回结点值 e.Value
// 结点不能为空
func (l *List) Remove(e *Element) interface{} {
	if e.list == l {
		// if e.list == l，当结点 e 插入到链表 l 时，l 必须已经初始化，或者 l == nil（e 是零值元素），l.remove 将会崩溃
		l.remove(e)
	}
	return e.Value
}
```

因为链表 l 是一个循环链表，所以在链表 l 的前端或后端插入结点，都是围绕结点 root 进行操作。

```go
// PushFront 在链表 l 的前端插入结点 e，并返回结点 e
func (l *List) PushFront(v interface{}) *Element {
	l.lazyInit()
	return l.insertValue(v, &l.root)
}

// PushBack 在链表 l 的后端插入结点 e，并返回结点 e
func (l *List) PushBack(v interface{}) *Element {
	l.lazyInit()
	return l.insertValue(v, l.root.prev)
}
```

在获取到结点 mark，同时 mark 是链表 l 里的结点，可以使用 InsertBefore 和 InsertAfter 在 mark 之前或之后插入一个新的结点。

```go
// InsertBefore 方法在结点 mark 之前插入一个新的结点 e，并返回结点 e
// 如果结点 mark 不是链表 l 的结点，那么链表 l 没有任何改变
// 结点 mark 不能为空
func (l *List) InsertBefore(v interface{}, mark *Element) *Element {
	if mark.list != l {
		return nil
	}
	return l.insertValue(v, mark.prev)
}

// InsertAfter 方法在结点 mark 之前插入一个新的结点 e，并返回结点 e
// 如果结点 mark 不是链表 l 的结点，那么链表 l 没有任何改变
// 结点 mark 不能为空
func (l *List) InsertAfter(v interface{}, mark *Element) *Element {
	if mark.list != l {
		return nil
	}
	return l.insertValue(v, mark)
}
```

MoveToFront，MoveToBack 和 PushFront，PushBack 类似。

```go
// MoveToFront 将结点 e 移动到链表 l 的前端。
// 如果 e 不是链表 l 的结点，那么链表没有任何改变。
// 结点 e 不能为空
func (l *List) MoveToFront(e *Element) {
	if e.list != l || l.root.next == e {
		return
	}
	l.move(e, &l.root)
}

// MoveToBack 将结点 e 移动到链表 l 的末端
// 如果 e 不是链表 l 的结点，那么链表没有任何改变。
// 结点 e 不能为空
func (l *List) MoveToBack(e *Element) {
	if e.list != l || l.root.prev == e {
		return
	}
	l.move(e, l.root.prev)
}
```

MoveBefore，MoveAfter 和 InsertBefore，InsertAfter 类似。

```go
// MoveBefore 移动结点 e 到结点 mark 之前
// 如果 e 或 mark 都不是 l 的结点，或者 e == mark，那么链表 l 没有任何改变
// 结点 e 和 mark 不能为空
func (l *List) MoveBefore(e, mark *Element) {
	if e.list != l || e == mark || mark.list != l {
		return
	}
	l.move(e, mark.prev)
}

// MoveAfter 移动结点 e 到结点 mark 之后
// 如果 e 或 mark 都不是 l 的结点，或者 e == mark，那么链表 l 没有任何改变
// 结点 e 和 mark 不能为空
func (l *List) MoveAfter(e, mark *Element) {
	if e.list != l || e == mark || mark.list != l {
		return
	}
	l.move(e, mark)
}
```

PushBackList，PushFrontList 用到了链表的遍历以及链表的插入。插入的实现在前面的代码里面已经实现过了，这里可以看一下链表的遍历。

```go
// PushBackList 在链表 l 的末尾插入另一个链表的拷贝
// 链表 l 和另一个链表可能相同。他们必须不为空。
func (l *List) PushBackList(other *List) {
	l.lazyInit()
	for i, e := other.Len(), other.Front(); i > 0; i, e = i-1, e.Next() {
		l.insertValue(e.Value, l.root.prev)
	}
}

// PushFrontList 在链表 l 的开头插入另一个链表的拷贝
// 链表 l 和另一个链表可能相同。他们必须不为空。
func (l *List) PushFrontList(other *List) {
	l.lazyInit()
	for i, e := other.Len(), other.Back(); i > 0; i, e = i-1, e.Prev() {
		l.insertValue(e.Value, &l.root)
	}
}
```

# 思维导图

![go-源码解读-双向链表list.png](https://cnymw.github.io/GolangStudy/docs/img/go-源码解读-双向链表list.png)
