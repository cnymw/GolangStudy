# Go 检测竞态条件

## 竞态条件

竞态条件（race condition）是两个或多个线程访问同一资源，如变量或结构体，并尝试对该资源进行读写而不考虑其他线程。这种类型代码会产生各种各样的 bug，通常需要大量的日志才能找到这些类型的 bug。

在 Go 版本 1.1 中，Go 工具库就引入了竞态检测器（race detector）。竞态检测器是在构建过程中内置到程序中的代码，一旦你的程序开始运行，就能够检测出竞态，并报告它发现的任何竞争状况。

## 竞态程序分析1

```go
package main

import (
    "fmt"
    "sync"
)

var Wait sync.WaitGroup
var Counter int = 0

func main() {

    for routine := 1; routine <= 2; routine++ {

        Wait.Add(1)
        go Routine(routine)
    }

    Wait.Wait()
    fmt.Printf("Final Counter: %d\n", Counter)
}

func Routine(id int) {

	for count := 0; count < 2; count++ {

		value := Counter
		value++
		Counter = value
	}

	Wait.Done()
}
```

该程序启动两个线程，每个线程将全局计数器变量递增两次，当两个线程运行完之后，程序将显示全局计数器变量的值。当运行程序时，它会显示数字 4。

通过竞态检测器运行代码，看看它找到了什么。打开源代码所在的 Terminal 会话，并使用 -race 选项构建代码。

```bash
go build -race
```

运行编译好的程序：

```bash
==================
WARNING: DATA RACE
Read at 0x000001277d08 by goroutine 8:
  main.Routine()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/goroutines/race/main.go:27 +0x47

Previous write at 0x000001277d08 by goroutine 7:
  main.Routine()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/goroutines/race/main.go:29 +0x64

Goroutine 8 (running) created at:
  main.main()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/goroutines/race/main.go:16 +0x75

Goroutine 7 (finished) created at:
  main.main()
      /Users/benjamin/Documents/golangworkspace/gopath/src/learnGo/src/goroutines/race/main.go:16 +0x75
==================
Final Counter: 4
Found 1 data race(s)
```

看样子工具检测到了代码的竞争条件，在竞态条件报告下面查看信息，可以看到程序的输出。全局计数器变量的值是 4，这就是这些类型的错误问题，代码可能大部分时间都在正常工作，然后随机发生一些不符合预期的事情。

## 竞态程序分析2

让我们对程序进行更改，来使得竞态条件变得更加的苛刻：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var Wait sync.WaitGroup
var Counter int = 0

func main() {

	for routine := 1; routine <= 2; routine++ {

		Wait.Add(1)
		go Routine(routine)
	}

	Wait.Wait()
	fmt.Printf("Final Counter: %d\n", Counter)
}

func Routine(id int) {

	for count := 0; count < 2; count++ {

		value := Counter
		time.Sleep(1 * time.Nanosecond)
		value++
		Counter = value
	}

	Wait.Done()
}
```

在循环中加入了 1ns 的停顿，再来看看全局计数器变量的值是多少：

```bash
Final Counter: 2
```

循环中的暂停导致程序失败，计数器变量的值现在是 2，不再是 4，那么发生了什么？让我们分析一下代码，并理解为什么暂停会暴露出错误。

在不暂停的情况下，程序运行如下所示：

![go-检测竞态条件-不暂停运行.png](https://cnymw.github.io/GolangStudy/docs/img/go-检测竞态条件-不暂停运行.png)

如果不暂停，生成的第一个线程将运行到完成，然后第二个线程开始运行，这就是程序运行正常的原因。代码本身是顺序执行的，因为它在机器上运行的速度足够快。

让我们看看程序在暂停时是如何运行的：

![go-检测竞态条件-暂停运行.png](https://cnymw.github.io/GolangStudy/docs/img/go-检测竞态条件-暂停运行.png)

暂停导致正在运行的两个线程之间发生上下文切换，这次我们有了一个完全不同的故事，让我们看看图表中正在运行的代码：

```go
value := Counter

time.Sleep(1 * time.Nanosecond)

value++

Counter = value
```

在循环的每次迭代中，全局计数器变量的值被本地捕获，然后本地副本被递增，最后回写全局计数器变量。如果这三行代码没有立即运行，我们就会出现问题。该图显示了全局计数器变量的读取以及上下文切换是如何导致所有的初始化问题。

在图中，在线程 1 的增量值被回写全局计数器变量之前，线程 2 会唤醒并读取全局计数器变量。基本上，这两个线程对全局计数器变量执行完全相同的读写操作，因此最终的值为 2。

## 竞态程序分析3

要解决该问题，你可能认为我们只需要将全局计数器变量的增量从三行代码减少到一行代码：

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

var Wait sync.WaitGroup
var Counter int = 0

func main() {

    for routine := 1; routine <= 2; routine++ {

        Wait.Add(1)
        go Routine(routine)
    }

    Wait.Wait()
    fmt.Printf("Final Counter: %d\n", Counter)
}

func Routine(id int) {

    for count := 0; count < 2; count++ {

        Counter = Counter + 1
        time.Sleep(1 * time.Nanosecond)
    }

    Wait.Done()
}
```

当我们运行这个版本的程序时，我们再次得到正确答案：

```bash
Final Counter: 4
```

如果我们通过竞态检测器运行这个代码，我们的问题应该会消失：

```bash
==================
WARNING: DATA RACE
Write by goroutine 5:
  main.Routine()
      /Users/bill/Spaces/Test/src/test/main.go:30 +0x44
  gosched0()
      /usr/local/go/src/pkg/runtime/proc.c:1218 +0x9f

Previous write by goroutine 4:
  main.Routine()
      /Users/bill/Spaces/Test/src/test/main.go:30 +0x44
  gosched0()
      /usr/local/go/src/pkg/runtime/proc.c:1218 +0x9f

Goroutine 5 (running) created at:
  main.main()
      /Users/bill/Spaces/Test/src/test/main.go:18 +0x66
  runtime.main()
      /usr/local/go/src/pkg/runtime/proc.c:182 +0x91

Goroutine 4 (running) created at:
  main.main()
      /Users/bill/Spaces/Test/src/test/main.go:18 +0x66
  runtime.main()
      /usr/local/go/src/pkg/runtime/proc.c:182 +0x91

==================
Final Counter: 4
Found 1 data race(s)
```

我们仍然存在竞态条件。

使用一行代码执行增量，程序将正确运行，为什么我们还有竞态条件？不要被我们用来增加计数器的一行代码所欺骗，让我们看看为这一行代码生成的汇编代码：

```bash
0064 (./main.go:30) MOVQ Counter+0(SB),BX ; Copy the value of Counter to BX
0065 (./main.go:30) INCQ ,BX              ; Increment the value of BX
0066 (./main.go:30) MOVQ BX,Counter+0(SB) ; Move the new value to Counter
```

实际上有三行汇编代码正在执行来递增计数器，这三行汇编代码看起来很像原始的 Go 代码。这三行汇编代码中的任意一行之后都可能有上下文切换。即使这个程序现在能够正常运行，但从技术上讲，这个 bug 仍然存在。

尽管我使用的事例很简单，但它向你展示了查找这些 bug 是多么复杂。对于上下文切换，Go 编译器生成的任何一行汇编代码都可以暂停，我们的 Go 代码看起来像是在安全地访问资源，而实际上底层的汇编代码根本就不安全。

## 竞态程序分析4

为了修复这个程序，我们需要保证对全局计数器变量的读写总是在任何其他线程可以访问该变量之前完成。channel 是序列化访问资源的好方法。在这个案例里，我建议使用互斥锁 Mutex。

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

var Wait sync.WaitGroup
var Counter int = 0
var Lock sync.Mutex

func main() {

    for routine := 1; routine <= 2; routine++ {

        Wait.Add(1)
        go Routine(routine)
    }

    Wait.Wait()
    fmt.Printf("Final Counter: %d\n", Counter)
}

func Routine(id int) {

    for count := 0; count < 2; count++ {

        Lock.Lock()

        value := Counter
        time.Sleep(1 * time.Nanosecond)
        value++
        Counter = value

        Lock.Unlock()
    }

    Wait.Done()
}
```

我们用竞态检测器构建程序并查看结果：

```bash
go build -race
./test

Final Counter: 4
```

这一次，我们得到了正确的结果，不存在竞态条件了。Mutex 保护锁和解锁之间的所有代码，确保一次只能有一个线程执行该代码。

# 参考资料

- [Detecting Race Conditions With Go](https://www.ardanlabs.com/blog/2013/09/detecting-race-conditions-with-go.html)