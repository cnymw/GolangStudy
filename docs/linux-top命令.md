# Linux top 命令详解

在类 Unix 操作系统上，top 命令提供运行系统的动态实时视图。它可以显示系统摘要信息，以及当前由内核管理的进程或线程的列表。显示的信息类型，顺序和大小都是用户可配置的。

## 语法

```bash
top -hv | -bcHisS -d delay -n limit -u|U user | -p pid -w [cols]
```

## 参数

