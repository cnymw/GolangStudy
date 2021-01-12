# zap

zap 是 uber 开源的高性能日志库，zap 表现非常突出，单线程 Qps 也是 logrus、seelog 的数倍

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

