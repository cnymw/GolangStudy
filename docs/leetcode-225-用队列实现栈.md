# leetcode 225 用队列实现栈

使用队列实现栈的下列操作：

push(x) -- 元素 x 入栈

pop() -- 移除栈顶元素

top() -- 获取栈顶元素

empty() -- 返回栈是否为空

题解：

这道题目非常有意思，同时考到了两个考点，队列和栈，有关队列的笔记可以看下 [队列](https://cnymw.github.io/GolangStudy/docs/数据结构-队列.html)

设原始序列为={1,2,3,4,5}，按顺序入栈后得到的栈如下所示:

![stack](https://gitee.com/GolangStudy_1/AliGolangStudy/raw/master/docs/img/数据结构-栈/数据结构-栈-临时栈1.png)

出栈的顺序为={5,4,3,2,1}

将原始序列插入到队列 A 中，得到的队列如下所示：

![queueA](https://gitee.com/GolangStudy_1/AliGolangStudy/raw/master/docs/img/数据结构-栈/数据结构-栈-临时队列1.png)

出队列的顺序为={1,2,3,4,5}，与预期的不符合，所以需要另外一个队列 B 来记录序列{1,2,3,4}，然后队列 A 才能弹出 5。

所以为了弹出 5，还需要一个队列 B，也就是 A 按顺序出{1,2,3,4}，然后 B 按顺序进{1,2,3,4}，最终弹出 5 ，B的数据如下所示:

![queueB](https://gitee.com/GolangStudy_1/AliGolangStudy/raw/master/docs/img/数据结构-栈/数据结构-栈-临时栈2.png)

go 实现如下：
```go
type Element interface{}

type Queue interface {
    Offer(e Element)
    Poll() Element
    Clear() bool
    Size() int
    IsEmpty() bool
}

type sliceEntry struct {
    element []Element
}

func NewQueue() *sliceEntry {
    return &sliceEntry{}
}

func (entry *sliceEntry) Offer(e Element) {
    entry.element = append(entry.element, e)
}

func (entry *sliceEntry) Poll() Element {
    if entry.IsEmpty() {
        return nil
    }
    first := entry.element[0]
    entry.element = entry.element[1:]
    return first
}

func (entry *sliceEntry) Clear() bool {
    if entry.IsEmpty() {
        return false
    }
    for i := 0; i < len(entry.element); i++ {
        entry.element[i] = nil
    }
    entry.element = nil
    return true
}
func (entry *sliceEntry) Size() int {
    return len(entry.element)
}

func (entry *sliceEntry) IsEmpty() bool {
    return entry.Size() == 0
}

type MyStack struct {
    queueA *sliceEntry
    queueB *sliceEntry
}

/** Initialize your data structure here. */
func Constructor() MyStack {
    qa, qb := NewQueue(), NewQueue()
    return MyStack{queueA: qa, queueB: qb}
}

/** Push element x onto stack. */
func (this *MyStack) Push(x int) {
    if this.queueA.Size() != 0 {
        this.queueA.Offer(x)
    } else if this.queueB.Size() != 0 {
        this.queueB.Offer(x)
    } else {
        this.queueA.Offer(x)
    }
}

/** Removes the element on top of the stack and returns that element. */
func (this *MyStack) Pop() int {
    if this.queueA.Size() == 0 && this.queueB.Size() == 0 {
        return 0
    }
    result := 0
    if this.queueA.Size() != 0 {
        for this.queueA.Size() > 0 {
            tmp := this.queueA.Poll()
            if this.queueA.Size() != 0 {
                this.queueB.Offer(tmp)
            } else {
                result = tmp.(int)
            }
        }
    } else {
        for this.queueB.Size() > 0 {
            tmp := this.queueB.Poll()
            if this.queueB.Size() != 0 {
                this.queueA.Offer(tmp)
            } else {
                result = tmp.(int)
            }
        }
    }
    return result
}

/** Get the top element. */
func (this *MyStack) Top() int {
    if this.queueA.Size() != 0 {
        for this.queueA.Size() > 0 {
            tmp := this.queueA.Poll()
            if this.queueA.Size() != 0 {
                this.queueB.Offer(tmp)
            } else {
                this.queueB.Offer(tmp)
                return tmp.(int)
            }
        }
    } else {
        for this.queueB.Size() > 0 {
            tmp := this.queueB.Poll()
            if this.queueB.Size() != 0 {
                this.queueA.Offer(tmp)
            } else {
                this.queueA.Offer(tmp)
                return tmp.(int)
            }
        }
    }
    return 0
}

/** Returns whether the stack is empty. */
func (this *MyStack) Empty() bool {
    if this.queueA.Size() != 0 || this.queueB.Size() != 0 {
        return false
    }
    return true
}
```