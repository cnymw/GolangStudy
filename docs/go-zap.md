# zap

zap 是 uber 开源的高性能日志库，zap 表现非常突出，单线程 Qps 也是 logrus、seelog 的数倍。

zap 是一个轻量级的库，代码量大概 1.6W 行，可以认真的阅读，提高对于开发工具库的熟悉程度。

## 性能对比

下面简单的总结了常用的日志组件的特点：

- seelog: 最早的日志组件之一，功能强大但是性能不佳，不过给社区后来的日志库在设计上提供了很多的启发。
- logrus: 代码清晰简单，同时提供结构化的日志，性能较好。
- zap: uber 开源的高性能日志库，面向高性能并且也确实做到了高性能。

| Package | Time | Time % to zap | Objects Allocated |
| :------ | :--: | :-----------: | :---------------: |
| zap | 862 ns/op | +0% | 5 allocs/op
| zap (sugared) | 1250 ns/op | +45% | 11 allocs/op
| zerolog | 4021 ns/op | +366% | 76 allocs/op
| go-kit | 4542 ns/op | +427% | 105 allocs/op
| apex/log | 26785 ns/op | +3007% | 115 allocs/op
| logrus | 29501 ns/op | +3322% | 125 allocs/op
| log15 | 29906 ns/op | +3369% | 122 allocs/op

## 代码分析

### 初始化对象

要使用 zap 来记录日志需要先 new 一个 Logger 对象，用于后续的操作

```go
logger, _ := zap.NewProduction()
```

因为一个 Logger 对象具有很多参数，同时都有默认值，所以 zap 内部初始化对象用到了建造者（Builder）模式，来快速的构造一个立马可以上手使用的 Logger 对象。

在这里是构造了一个生产环境 Logger，对应也有 NewDevelopment 方法来构造开发环境 Logger

```go
// NewProduction builds a sensible production Logger that writes InfoLevel and
// above logs to standard error as JSON.
//
// It's a shortcut for NewProductionConfig().Build(...Option).
//
// NewProduction 构造了一个实用的生产环境 Logger ，它将 InfoLevel 和更高级别的日志用 JSON 字符串写入标准错误。
// 这是个 NewProductionConfig().Build(...Option) 快速函数。
func NewProduction(options ...Option) (*Logger, error) {
	return NewProductionConfig().Build(options...)
}
```

Build 函数使用了一系列参数，并返回了一个 Logger 指针。

从 zapcore.NewCore(enc, sink, cfg.Level) 可以分析出，NewCore 是构造了一个 io 对象，enc 指定字符编码，sink 指定输出地址，cfg.Level 指定日志级别。

```go
// Build constructs a logger from the Config and Options.
//
// Build 函数通过 Config 和 Options 构造了一个 Logger。
func (cfg Config) Build(opts ...Option) (*Logger, error) {
	enc, err := cfg.buildEncoder()
	if err != nil {
		return nil, err
	}

	sink, errSink, err := cfg.openSinks()
	if err != nil {
		return nil, err
	}

	if cfg.Level == (AtomicLevel{}) {
		return nil, fmt.Errorf("missing Level")
	}

	log := New(
		zapcore.NewCore(enc, sink, cfg.Level),
		cfg.buildOptions(errSink)...,
	)
	if len(opts) > 0 {
		log = log.WithOptions(opts...)
	}
	return log, nil
}
```

对于日志没有任何场景的需求的话，使用 NewProduction 等默认的构造器即可满足，但是一旦有复杂的日志场景，便需要使用 New 方法来构造出适用于多场景的 zapcore.Core 对象：multiCore

在 New 构造器中:

- core：指定底层输出日志的 io 对象 
- errorOutput：指定如果 zap 内部报错的时候，输出错误信息的地址
- addStack：指定输出日志的级别，FatalLevel + 1 代表所有级别（包括 FatalLevel）的日志都会输出

```go
// New constructs a new Logger from the provided zapcore.Core and Options. If
// the passed zapcore.Core is nil, it falls back to using a no-op
// implementation.
//
// This is the most flexible way to construct a Logger, but also the most
// verbose. For typical use cases, the highly-opinionated presets
// (NewProduction, NewDevelopment, and NewExample) or the Config struct are
// more convenient.
//
// For sample code, see the package-level AdvancedConfiguration example.
//
// 通过提供的 zapcore.Core 和 Options，New 方法构造一个新的 Logger。
// 如果传的 zapcore.Core 是空，则返回一个空实现。
//
// 这是最灵活的方式去构造一个 Logger，也是最冗长的。
// 典型的用法是使用默认的构造方法（例如 NewProduction，NewDevelopment，NewExample），或者默认的 Config 类型更加方便。
//
// 有关示例代码，请参阅包级别的 AdvancedConfiguration 示例。
func New(core zapcore.Core, options ...Option) *Logger {
	if core == nil {
		return NewNop()
	}
	log := &Logger{
		core:        core,
		errorOutput: zapcore.Lock(os.Stderr),
		addStack:    zapcore.FatalLevel + 1,
	}
	return log.WithOptions(options...)
}
```

以下是在复杂场景下使用 New 构造 multiCore 的示例。

```go
func Example_advancedConfiguration() {
	// The bundled Config struct only supports the most common configuration
	// options. More complex needs, like splitting logs between multiple files
	// or writing to non-file outputs, require use of the zapcore package.
	//
	// In this example, imagine we're both sending our logs to Kafka and writing
	// them to the console. We'd like to encode the console output and the Kafka
	// topics differently, and we'd also like special treatment for
	// high-priority logs.

	// First, define our level-handling logic.
	//
	// 捆绑好的配置只支持最常见的场景。
	// 更加复杂的场景时，例如分割日志到多个文件或者向非文件输出日志的时候，需要用到 zapcore 包了。
	//
	// 在下面这个例子中， 假设我们要同时向 Kafka 和控制台输出日志。
	// 我们希望对控制台输出和 Kafka topics 进行不同的编码，还希望对高级别的日志进行特殊处理。
	//
	// 首先，定义我们的级别处理逻辑。
	highPriority := zap.LevelEnablerFunc(func(lvl zapcore.Level) bool {
		return lvl >= zapcore.ErrorLevel
	})
	lowPriority := zap.LevelEnablerFunc(func(lvl zapcore.Level) bool {
		return lvl < zapcore.ErrorLevel
	})

	// Assume that we have clients for two Kafka topics. The clients implement
	// zapcore.WriteSyncer and are safe for concurrent use. (If they only
	// implement io.Writer, we can use zapcore.AddSync to add a no-op Sync
	// method. If they're not safe for concurrent use, we can add a protecting
	// mutex with zapcore.Lock.)
	//
	// 假设我们有针对两个 Kafka topics 有客户端。
	// 客户端实现了 zapcore.WriteSyncer 接口，并且在并发场景下是安全的。
	//（如果他们只实现 io.Writer 接口，我们可以使用 zapcore.AddSync 来添加一个空实现的 Sync 方法。） 
	//（如果他们对于并发场景不是安全的，那么我们可以使用 zapcore.Lock 添加一个受保护的 mutex 互斥锁。）
	topicDebugging := zapcore.AddSync(ioutil.Discard)
	topicErrors := zapcore.AddSync(ioutil.Discard)

	// High-priority output should also go to standard error, and low-priority
	// output should also go to standard out.
	//
	// 高优先级的应该输出到 Stdout，低优先级的应该输出到 Stderr。
	consoleDebugging := zapcore.Lock(os.Stdout)
	consoleErrors := zapcore.Lock(os.Stderr)

	// Optimize the Kafka output for machine consumption and the console output
	// for human operators.
	//
	// 针对 Kafka 输出优化了机器消耗，同时针对人工操作优化了控制台输出。
	kafkaEncoder := zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
	consoleEncoder := zapcore.NewConsoleEncoder(zap.NewDevelopmentEncoderConfig())

	// Join the outputs, encoders, and level-handling functions into
	// zapcore.Cores, then tee the four cores together.
	//
	// 将输出，编码和级别控制函数封装到 zapcore.Cores 里，然后将四个 cores 合并在一起。
	core := zapcore.NewTee(
		zapcore.NewCore(kafkaEncoder, topicErrors, highPriority),
		zapcore.NewCore(consoleEncoder, consoleErrors, highPriority),
		zapcore.NewCore(kafkaEncoder, topicDebugging, lowPriority),
		zapcore.NewCore(consoleEncoder, consoleDebugging, lowPriority),
	)

	// From a zapcore.Core, it's easy to construct a Logger.
	//
	// 通过 zapcore.Core，可以很轻松地构造一个 Logger。
	logger := zap.New(core)
	defer logger.Sync()
	logger.Info("constructed a logger")
}
```