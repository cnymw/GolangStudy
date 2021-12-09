# redis 持久化命令

## SAVE

> 可用版本： >= 1.0.0

SAVE 命令执行一个同步保存操作，将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。

一般来说，在生产环境很少执行 SAVE 操作，因为它会阻塞所有客户端，保存数据库的任务通常由 BGSAVE 命令异步地执行。然而，如果负责保存数据的后台子进程不幸出现问题时， SAVE 可以作为保存数据的最后手段来使用。

### 返回值

保存成功时返回 OK 。

### 示例

```bash
redis> SAVE
OK
```

## BGSAVE

> 可用版本： >= 1.0.0

在后台异步(Asynchronously)保存当前数据库的数据到磁盘。

BGSAVE 命令执行之后立即返回 OK ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

客户端可以通过 LASTSAVE 命令查看相关信息，判断 BGSAVE 命令是否执行成功。

### 返回值

反馈信息。

### 示例

```bash
redis> BGSAVE
Background saving started
```

## BGREWRITEAOF

> 可用版本： >= 1.0.0

执行一个 AOF文件重写操作。重写会创建一个当前 AOF 文件的体积优化版本。

即使 BGREWRITEAOF 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 BGREWRITEAOF 成功之前不会被修改。

重写操作只会在没有其他持久化工作在后台执行时被触发，也就是说：

- 如果 Redis 的子进程正在执行快照的保存工作，那么 AOF 重写的操作会被预定(scheduled)，等到保存工作完成之后再执行 AOF 重写。在这种情况下， BGREWRITEAOF 的返回值仍然是 OK
  ，但还会加上一条额外的信息，说明 BGREWRITEAOF 要等到保存操作完成之后才能执行。在 Redis 2.6 或以上的版本，可以使用 INFO [section] 命令查看 BGREWRITEAOF 是否被预定。

- 如果已经有别的 AOF 文件重写在执行，那么 BGREWRITEAOF 返回一个错误，并且这个新的 BGREWRITEAOF 请求也不会被预定到下次执行。

从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， BGREWRITEAOF 仅仅用于手动触发重写操作。

### 返回值

反馈信息。

### 示例

```bash
redis> BGREWRITEAOF
Background append only file rewriting started
```

## LASTSAVE

> 可用版本： >= 1.0.0

返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示。

### 返回值

一个 UNIX 时间戳。

### 示例

```bash
redis> LASTSAVE
(integer) 1324043588
```

# 思维导图

![redis-持久化命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-持久化命令.png)

