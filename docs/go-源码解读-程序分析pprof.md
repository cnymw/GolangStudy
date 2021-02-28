# Go 源码解读 程序分析模块 pprof

## pprof 包概览

pprof 包以 pprof 可视化工具所期望的格式写入运行时 profiling data。

下面会简单的介绍如何通过 Go SDK 自带的包 pprof 来分析 Go 程序。

### 如何 profiling 一个 Go 程序

profiling Go 程序的第一步是启用 profiling。

对于用标准测试包构建的评测基准的支持已经内置在 Go test 模块中。

例如，以下命令是在当前目录中运行基准测试（benchmark），并将 CPU 和内存配置文件写入 cpu.prof 和 mem.prof。

```go
go test -cpuprofile cpu.prof -memprofile mem.prof -bench .
```

要向独立的程序来添加相同的 profiling 能力，需要向 main 函数添加如下代码：

```go
var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to `file`")
var memprofile = flag.String("memprofile", "", "write memory profile to `file`")

func main() {
    flag.Parse()
    if *cpuprofile != "" {
        f, err := os.Create(*cpuprofile)
        if err != nil {
            log.Fatal("could not create CPU profile: ", err)
        }
        defer f.Close() // error handling omitted for example
        if err := pprof.StartCPUProfile(f); err != nil {
            log.Fatal("could not start CPU profile: ", err)
        }
        defer pprof.StopCPUProfile()
    }

    // ... rest of the program ...

    if *memprofile != "" {
        f, err := os.Create(*memprofile)
        if err != nil {
            log.Fatal("could not create memory profile: ", err)
        }
        defer f.Close() // error handling omitted for example
        runtime.GC() // get up-to-date statistics
        if err := pprof.WriteHeapProfile(f); err != nil {
            log.Fatal("could not write memory profile: ", err)
        }
    }
}
```

还有一个用于 profiling 数据的标准 HTTP 接口。添加以下代码将在`/debug/pprof/`请求中安装 handlers 用于下载实时的 profiling 文件。

```go
import _ "net/http/pprof"
```

可以查看包`net/http/pprof`来获取更多的信息。

然后可以使用 pprof 工具来可视化分析数据。

```go
go tool pprof cpu.prof
```

pprof 命令行提供了许多命令。

常用的命令包括`top`，用于打印性能消耗排名靠前的操作。

还有`web`，用于打开热点操作及其调用图的交互式图形。

有关所有命令的信息，请使用`help`。

更多关于 pprof 的信息，可以查看[ google / pprof ](https://github.com/google/pprof/blob/master/doc/README.md)

## CPU profiling

对于 CPU 的 profiling 是从调用 `StartCPUProfile` 开始，到 `StopCPUProfile` 结束，下面就详细的介绍下这两个函数逻辑以及实现。

### StartCPUProfile / StopCPUProfile

```go
// StartCPUProfile 为当前的进程开启 CPU profiling。
// 当在 profiling 时，分析文件将被缓冲写入参数指定的 io.Writer w。
// 当已经开启 profiling 时，StartCPUProfile将会返回 error。
//
// 在类 Unix 系统上，StartCPUProfile 在默认情况下不适用于使用 -buildmode=c-archive 或者 -buildmode=c-shared。
// StartCPUProfile 依赖于 SIGPROF 信号，但该信号将被传递到 main 程序的 SIGPROF 信号处理程序（如果有的话），而不是 Go 使用的处理程序。
// 要让其工作的话，针对于 syscall.SIGPROF 来调用 os/signal.Notify，但请注意，这样做可能会中断 main 程序正在执行的任何 profiling。
func StartCPUProfile(w io.Writer) error {
    // 运行时程序允许可变的分析速率，但实际上操作系统不能以超过 500Hz 
    // 的频率触发信号，而且我们对信号的处理也代价不菲（主要是获取堆栈跟踪复杂）。
    // 100Hz 是一个合理的选择：它的频率足以产生有用的数据，同时也可以不使系统
    // 陷入崩溃，它也是一个很好的整数可以使采样计数很容易转换为秒。
    // 我们没有要求每个 client 指定频率，我们硬编码了频率为 100Hz。
    const hz = 100
    
    cpu.Lock()
    defer cpu.Unlock()
    if cpu.done == nil {
        cpu.done = make(chan bool)
    }
    // 双重检查，不允许开启两个 profiling
    if cpu.profiling {
        return fmt.Errorf("cpu profiling already in use")
    }
    cpu.profiling = true
    runtime.SetCPUProfileRate(hz)
    go profileWriter(w)
    return nil
}
```

在这里，`runtime.SetCPUProfileRate(hz)`用于设置 CPU 的 profiling 速率。

`profileWriter(w)`在这里是写向`w io.Writer`写 profiling 数据的核心逻辑。

```go
func profileWriter(w io.Writer) {
	// newProfileBuilder 返回一个新的 profileBuilder。
	// 可以通过调用 b.addCPUData 添加从运行时获得的 CPU profiling 数据，然后通过调用 b.finish 获得最终的 profiling 文件。
	b := newProfileBuilder(w)
	var err error
	for {
		time.Sleep(100 * time.Millisecond)
		
		// runtime 提供的 readProfile 会持续返回一个二进制 CPU profiling 堆栈跟踪数据块，直到数据不可用为止。
		// 如果打开 profiling 时返回了累积的所有 profile 数据，当关闭 profiling 时，readProfile 会返回 eof=true。
		// 在再次调用 readProfile 之前，调用者必须保存返回的数据和标记。
		data, tags, eof := readProfile()
		if e := b.addCPUData(data, tags); e != nil && err == nil {
			err = e
		}
		if eof {
			break
		}
	}
	if err != nil {
		// 运行时不应生成无效的或截断的 profile。
		// 它会删除无法放入日志缓冲区的记录。
		panic("runtime/pprof: converting profile: " + err.Error())
	}
	b.build()
	cpu.done <- true
}
```

`readProfile()`函数和`runtime_pprof_readProfile()`函数通过`go:linkname`链接。

> go:linkname 引导编译器将当前(私有)方法或者变量在编译时链接到指定的位置的方法或者变量，第一个参数表示当前方法或变量，第二个参数表示目标方法或变量，因为这关指令会破坏系统和包的模块化，因此在使用时必须导入unsafe。

也就是说，当你使用`go:linkname`链接时，不管你用没用到`unsafe`包，你都必须`import "unsafe"`。

```go
// pprof.go
func readProfile() (data []uint64, tags []unsafe.Pointer, eof bool)

// cpuprof.go
//go:linkname runtime_pprof_readProfile runtime/pprof.readProfile
func runtime_pprof_readProfile() ([]uint64, []unsafe.Pointer, bool) {
	lock(&cpuprof.lock)
	log := cpuprof.log
	unlock(&cpuprof.lock)
	data, tags, eof := log.read(profBufBlocking)
	if len(data) == 0 && eof {
		lock(&cpuprof.lock)
		cpuprof.log = nil
		unlock(&cpuprof.lock)
	}
	return data, tags, eof
}
```

`b.addCPUData(data, tags)`用于将`data`写入到 profile 文件中。

```go
// addCPUData 将 CPU profiling 数据添加到 profile 文件中。
// 数据必须是 runtime 交付的完整记录数。
func (b *profileBuilder) addCPUData(data []uint64, tags []unsafe.Pointer) error {
	if !b.havePeriod {
		// 第一条记录是一个阶段（因为无法开启两个 profiling，所以要通过）
		if len(data) < 3 {
			return fmt.Errorf("truncated profile")
		}
		if data[0] != 3 || data[2] == 0 {
			return fmt.Errorf("malformed profile")
		}
		// data[2]是以 Hz 为单位的采样率。
		// 转换为采样周期(纳秒)。
		b.period = 1e9 / int64(data[2])
		b.havePeriod = true
		data = data[3:]
	}
	
	// 分析 profile 文件中的 CPU 采样数据。
	// 每个采样数据为 3+n 个 uint64：
	// data[0] = 3+n
	// data[1] = time stamp (可以忽略)
	// data[2] = count
	// data[3:3+n] = stack
	// 如果 count 为0，stack 长度为 ，那这是运行时插入的溢出记录，表示 stack[0] 样本丢失。
	// 否则，计数通常为 1，但在一些特殊情况下（如丢失的 non-Go 样本），计数可能更大。
	// 因为有许多具有相同堆栈的样本到达，所以我们希望立即进行重复数据消除，这是使用 b.m.profMap 完成的。
	for len(data) > 0 {
		// 如果第一条数据不完整，那么表示数据是被截断的
		if len(data) < 3 || data[0] > uint64(len(data)) {
			return fmt.Errorf("truncated profile")
		}
		if data[0] < 3 || tags != nil && len(tags) < 1 {
			return fmt.Errorf("malformed profile")
		}
		count := data[2]
		stk := data[3:data[0]]
		data = data[data[0]:]
		var tag unsafe.Pointer
		if tags != nil {
			tag = tags[0]
			tags = tags[1:]
		}

		if count == 0 && len(stk) == 1 {
			// 溢出记录
			count = uint64(stk[0])
			stk = []uint64{
				// gentraceback 保证堆栈中的 pc 可以无条件的递减并且仍然有效，所以我们必须这样做。
				uint64(funcPC(lostProfileEvent) + 1),
			}
		}
		b.m.lookup(stk, tag).count += int64(count)
	}
	return nil
}
```

`b.m.lookup` 会将样本数据存入到`map`结构的对象中，同时能够保证存储的数据不会重复。

在结束了 profiling 数据的记录之后，`eof=true` 时，调用`b.build()`便可生成最终的 profile 文件。

调用 StopCPUProfile 可以结束 CPU profiling 过程，如果不结束的话，后台会持续的进行 profiling 过程，所以务必要调用该接口。

```go
// StopCPUProfile 停止当前的 CPU profile（如果有的话）。
// StopCPUProfile 仅在写入所有的 profile 完成后返回。
func StopCPUProfile() {
	cpu.Lock()
	defer cpu.Unlock()

	if !cpu.profiling {
		return
	}
	cpu.profiling = false
	runtime.SetCPUProfileRate(0)
	<-cpu.done
}
```

### readProfile

`readProfile()`所链接的`runtime_pprof_readProfile()`核心逻辑是从`profBuf`对象中读取数据。

`profBuf`的数据是来自于程序接受了操作系统发来的 SIGPROF 信号之后，进行解析信号之后所读取的数据。

以 Linux 操作系统为例，SIGPROF 是 Linux 的信号机制中的一种信号，是用于 profiling 的定时报警器（profiling time alarm），在这里的 profiling 指的是操作系统提供的 profiling 能力，而不是单纯指的 Go SDK 所提供的 profiling 能力了。

> 在计算机科学中，信号是Unix、类Unix以及其他POSIX兼容的操作系统中进程间通讯的一种有限制的方式。它是一种异步的通知机制，用来提醒进程一个事件已经发生。当一个信号发送给一个进程，操作系统中断了进程正常的控制流程，此时，任何非原子操作都将被中断。如果进程定义了信号的处理函数，那么它将被执行，否则就执行默认的处理函数。

