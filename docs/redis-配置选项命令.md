# redis 配置选项命令

## CONFIG SET parameter value

> 可用版本： >= 2.0.0

CONFIG SET 命令可以动态地调整 Redis 服务器的配置(configuration)而无须重启。

你可以使用它修改配置参数，或者改变 Redis 的持久化(Persistence)方式。

CONFIG SET 可以修改的配置参数可以使用命令 CONFIG GET * 来列出，所有被 CONFIG SET 修改的配置参数都会立即生效。

### 返回值

当设置成功时返回 OK ，否则返回一个错误。

### 示例

```bash
redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "1024"

redis> CONFIG SET slowlog-max-len 10086
OK

redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "10086"
```

## CONFIG GET parameter

> 可用版本： >= 2.0.0

CONFIG GET 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters)，在 Redis 2.4 版本中， 有部分参数没有办法用 CONFIG GET 访问，但是在最新的 Redis 2.6 版本中，所有配置参数都已经可以用 CONFIG GET 访问了。

CONFIG GET 接受单个参数 parameter 作为搜索关键字，查找所有匹配的配置参数，其中参数和值以“键-值对”(key-value pairs)的方式排列。

比如执行 CONFIG GET s* 命令，服务器就会返回所有以 s 开头的配置参数及参数的值。

使用命令 CONFIG GET * ，可以列出 CONFIG GET 命令支持的所有参数。

所有被 CONFIG SET 所支持的配置参数都可以在配置文件 redis.conf 中找到，不过 CONFIG GET 和 CONFIG SET 使用的格式和 redis.conf 文件所使用的格式有以下两点不同：

- 10kb 、 2gb 这些在配置文件中所使用的储存单位缩写，不可以用在 CONFIG 命令中， CONFIG SET 的值只能通过数字值显式地设定。

像 CONFIG SET xxx 1k 这样的命令是错误的，正确的格式是 CONFIG SET xxx 1000 。

- save 选项在 redis.conf 中是用多行文字储存的，但在 CONFIG GET 命令中，它只打印一行文字。

以下是 save 选项在 redis.conf 文件中的表示：

```bash
save 900 1
save 300 10
save 60 10000
```


但是 CONFIG GET 命令的输出只有一行：

```bash
redis> CONFIG GET save
1) "save"
2) "900 1 300 10 60 10000"
```

上面 save 参数的三个值表示：在 900 秒内最少有 1 个 key 被改动，或者 300 秒内最少有 10 个 key 被改动，又或者 60 秒内最少有 1000 个 key 被改动，以上三个条件随便满足一个，就触发一次保存操作。

### 返回值

给定配置参数的值。

## CONFIG RESETSTAT

> 可用版本： >= 2.0.0

重置 INFO 命令中的某些统计数据，包括：

- Keyspace hits (键空间命中次数)

- Keyspace misses (键空间不命中次数)

- Number of commands processed (执行命令的次数)

- Number of connections received (连接服务器的次数)

- Number of expired keys (过期key的数量)

- Number of rejected connections (被拒绝的连接数量)

- Latest fork(2) time(最后执行 fork(2) 的时间)

- The aof_delayed_fsync counter(aof_delayed_fsync 计数器的值)

### 返回值

总是返回 OK 。

## CONFIG REWRITE

> 可用版本： >= 2.8.0

CONFIG REWRITE 命令对启动 Redis 服务器时所指定的 redis.conf 文件进行改写： 因为 CONFIG_SET 命令可以对服务器的当前配置进行修改， 而修改后的配置可能和 redis.conf 文件中所描述的配置不一样， CONFIG REWRITE 的作用就是通过尽可能少的修改， 将服务器当前所使用的配置记录到 redis.conf 文件中。

重写会以非常保守的方式进行：

- 原有 redis.conf 文件的整体结构和注释会被尽可能地保留。

- 如果一个选项已经存在于原有 redis.conf 文件中 ， 那么对该选项的重写会在选项原本所在的位置（行号）上进行。

- 如果一个选项不存在于原有 redis.conf 文件中， 并且该选项被设置为默认值， 那么重写程序不会将这个选项添加到重写后的 redis.conf 文件中。

- 如果一个选项不存在于原有 redis.conf 文件中， 并且该选项被设置为非默认值， 那么这个选项将被添加到重写后的 redis.conf 文件的末尾。

- 未使用的行会被留白。 比如说， 如果你在原有 redis.conf 文件上设置了数个关于 save 选项的参数， 但现在你将这些 save 参数的一个或全部都关闭了， 那么这些不再使用的参数原本所在的行就会变成空白的。

即使启动服务器时所指定的 redis.conf 文件已经不再存在， CONFIG REWRITE 命令也可以重新构建并生成出一个新的 redis.conf 文件。

另一方面， 如果启动服务器时没有载入 redis.conf 文件， 那么执行 CONFIG REWRITE 命令将引发一个错误。

### 原子性重写

对 redis.conf 文件的重写是原子性的， 并且是一致的： 如果重写出错或重写期间服务器崩溃， 那么重写失败， 原有 redis.conf 文件不会被修改。 如果重写成功， 那么 redis.conf 文件为重写后的新文件。

### 返回值

一个状态值：如果配置重写成功则返回 OK ，失败则返回一个错误。

### 示例

以下是执行 CONFIG REWRITE 前， 被载入到 Redis 服务器的 redis.conf 文件中关于 appendonly 选项的设置：

```bash
# ... 其他选项

appendonly no

# ... 其他选项
```

在执行以下命令之后：

```bash
redis> CONFIG GET appendonly           # appendonly 处于关闭状态
1) "appendonly"
2) "no"

redis> CONFIG SET appendonly yes       # 打开 appendonly
OK

redis> CONFIG GET appendonly
1) "appendonly"
2) "yes"

redis> CONFIG REWRITE                  # 将 appendonly 的修改写入到 redis.conf 中
OK
```

重写后的 redis.conf 文件中的 appendonly 选项将被改写：

```bash
# ... 其他选项

appendonly yes

# ... 其他选项
```

# 思维导图

![redis-配置选项命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-配置选项命令.png)

