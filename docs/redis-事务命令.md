# redis 事务命令

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。

- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。

- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
  
- 命令入队。
  
- 执行事务。

## MULTI

> 可用版本：>= 1.2.0

标记一个事务块的开始。

事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令原子性(atomic)地执行。

### 返回值

总是返回 OK 。

### 示例

```bash
redis> MULTI            # 标记事务开始
OK

redis> INCR user_id     # 多条命令按顺序入队
QUEUED

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> PING
QUEUED

redis> EXEC             # 执行
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG
```

## EXEC

> 可用版本：>= 1.2.0

执行所有事务块内的命令。

假如某个(或某些) key 正处于 WATCH 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 EXEC 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。

### 返回值

事务块内所有命令的返回值，按命令执行的先后顺序排列。

当操作被打断时，返回空值 nil 。

### 示例

```bash
# 事务被成功执行

redis> MULTI
OK

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> PING
QUEUED

redis> EXEC
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG
```

## DISCARD

> 可用版本： >= 2.0.0

取消事务，放弃执行事务块内的所有命令。

如果正在使用 WATCH 命令监视某个(或某些) key，那么取消所有监视，等同于执行命令 UNWATCH 。

### 返回值

总是返回 OK 。

### 示例

```bash
redis> MULTI
OK

redis> PING
QUEUED

redis> SET greeting "hello"
QUEUED

redis> DISCARD
OK
```

## WATCH

> 可用版本：>= 2.2.0

监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

### 返回值

总是返回 OK 。

### 示例

```bash
redis> WATCH lock lock_times
OK
```

## UNWATCH

> 可用版本：>= 2.2.0

取消 WATCH 命令对所有 key 的监视。

如果在执行 WATCH 命令之后， EXEC 命令或 DISCARD 命令先被执行了的话，那么就不需要再执行 UNWATCH 了。

因为 EXEC 命令会执行事务，因此 WATCH 命令的效果已经产生了；而 DISCARD 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 UNWATCH 了。

### 返回值

总是 OK 。

### 示例

```bash
redis> WATCH lock lock_times
OK

redis> UNWATCH
OK
```

# 思维导图

![redis-事务命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-事务命令.png)

