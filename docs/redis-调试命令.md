# redis 调试命令

## PING

> 可用版本： >= 1.0.0

使用客户端向 Redis 服务器发送一个 PING ，如果服务器运作正常的话，会返回一个 PONG 。

通常用于测试与服务器的连接是否仍然生效，或者用于测量延迟值。

### 返回值

如果连接正常就返回一个 PONG ，否则返回一个连接错误。

### 示例

```bash
# 客户端和服务器连接正常

redis> PING
PONG

# 客户端和服务器连接不正常(网络不正常或服务器未能正常运行)

redis 127.0.0.1:6379> PING
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```

## ECHO message

> 可用版本： >= 1.0.0

打印一个特定的信息 message ，测试时使用。

### 返回值

message 自身。

### 示例

```bash
redis> ECHO "Hello Moto"
"Hello Moto"

redis> ECHO "Goodbye Moto"
"Goodbye Moto"
```

## OBJECT subcommand [arguments [arguments]]

> 可用版本： >= 2.2.3

OBJECT 命令允许从内部察看给定 key 的 Redis 对象， 它通常用在除错(debugging)或者了解为了节省空间而对 key 使用特殊编码的情况。 当将 Redis 用作缓存程序时，你也可以通过 OBJECT
命令中的信息，决定 key 的驱逐策略(eviction policies)。

OBJECT 命令有多个子命令：

- OBJECT REFCOUNT <key> 返回给定 key 引用所储存的值的次数。此命令主要用于除错。

- OBJECT ENCODING <key> 返回给定 key 锁储存的值所使用的内部表示(representation)。

- OBJECT IDLETIME <key> 返回给定 key 自储存以来的空闲时间(idle， 没有被读取也没有被写入)，以秒为单位。

对象可以以多种方式编码：

- 字符串可以被编码为 raw (一般字符串)或 int (为了节约内存，Redis 会将字符串表示的 64 位有符号整数编码为整数来进行储存）。

- 列表可以被编码为 ziplist 或 linkedlist 。 ziplist 是为节约大小较小的列表空间而作的特殊表示。

- 集合可以被编码为 intset 或者 hashtable 。 intset 是只储存数字的小集合的特殊表示。

- 哈希表可以编码为 zipmap 或者 hashtable 。 zipmap 是小哈希表的特殊表示。

- 有序集合可以被编码为 ziplist 或者 skiplist 格式。 ziplist 用于表示小的有序集合，而 skiplist 则用于表示任何大小的有序集合。

假如你做了什么让 Redis 没办法再使用节省空间的编码时(比如将一个只有 1 个元素的集合扩展为一个有 100 万个元素的集合)，特殊编码类型(specially encoded types)会自动转换成通用类型(general
type)。

### 返回值

REFCOUNT 和 IDLETIME 返回数字。 ENCODING 返回相应的编码类型。

### 示例

```bash
redis> SET game "COD"           # 设置一个字符串
OK

redis> OBJECT REFCOUNT game     # 只有一个引用
(integer) 1

redis> OBJECT IDLETIME game     # 等待一阵。。。然后查看空闲时间
(integer) 90

redis> GET game                 # 提取game， 让它处于活跃(active)状态
"COD"

redis> OBJECT IDLETIME game     # 不再处于空闲状态
(integer) 0

redis> OBJECT ENCODING game     # 字符串的编码方式
"raw"

redis> SET big-number 23102930128301091820391092019203810281029831092  # 非常长的数字会被编码为字符串
OK

redis> OBJECT ENCODING big-number
"raw"

redis> SET small-number 12345  # 而短的数字则会被编码为整数
OK

redis> OBJECT ENCODING small-number
"int"
```

## SLOWLOG subcommand [argument]

> 可用版本： >= 2.2.12

### 什么是 SLOWLOG

Slow log 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

### 设置 SLOWLOG

Slow log 的行为由两个配置参数(configuration parameter)指定，可以通过改写 redis.conf 文件或者用 CONFIG GET 和 CONFIG SET 命令对它们动态地进行修改。

第一个选项是 slowlog-log-slower-than ，它决定要对执行时间大于多少微秒(microsecond，1秒 = 1,000,000 微秒)的查询进行记录。

比如执行以下命令将让 slow log 记录所有查询时间大于等于 100 微秒的查询：

CONFIG SET slowlog-log-slower-than 100

而以下命令记录所有查询时间大于 1000 微秒的查询：

CONFIG SET slowlog-log-slower-than 1000

另一个选项是 slowlog-max-len ，它决定 slow log 最多能保存多少条日志， slow log 本身是一个 FIFO 队列，当队列大小超过 slowlog-max-len
时，最旧的一条日志将被删除，而最新的一条日志加入到 slow log ，以此类推。

以下命令让 slow log 最多保存 1000 条日志：

CONFIG SET slowlog-max-len 1000

使用 CONFIG GET 命令可以查询两个选项的当前值：

```bash
redis> CONFIG GET slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "1000"

redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "1000"
```

### 查看 slow log

要查看 slow log ，可以使用 SLOWLOG GET 或者 SLOWLOG GET number 命令，前者打印所有 slow log ，最大长度取决于 slowlog-max-len 选项的值，而 SLOWLOG GET
number 则只打印指定数量的日志。

最新的日志会最先被打印：

```bash
# 为测试需要，将 slowlog-log-slower-than 设成了 10 微秒

redis> SLOWLOG GET
1) 1) (integer) 12                      # 唯一性(unique)的日志标识符
   2) (integer) 1324097834              # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
   3) (integer) 16                      # 查询执行时间，以微秒为单位
   4) 1) "CONFIG"                       # 执行的命令，以数组的形式排列
      2) "GET"                          # 这里完整的命令是 CONFIG GET slowlog-log-slower-than
      3) "slowlog-log-slower-than"

2) 1) (integer) 11
   2) (integer) 1324097825
   3) (integer) 42
   4) 1) "CONFIG"
      2) "GET"
      3) "*"

3) 1) (integer) 10
   2) (integer) 1324097820
   3) (integer) 11
   4) 1) "CONFIG"
      2) "GET"
      3) "slowlog-log-slower-than"

```

日志的唯一 id 只有在 Redis 服务器重启的时候才会重置，这样可以避免对日志的重复处理(比如你可能会想在每次发现新的慢查询时发邮件通知你)。

### 查看当前日志的数量

使用命令 SLOWLOG LEN 可以查看当前日志的数量。

请注意这个值和 slower-max-len 的区别，它们一个是当前日志的数量，一个是允许记录的最大日志的数量。

```bash
redis> SLOWLOG LEN
(integer) 14
```

### 清空日志

使用命令 SLOWLOG RESET 可以清空 slow log 。

```bash
redis> SLOWLOG LEN
(integer) 14

redis> SLOWLOG RESET
OK

redis> SLOWLOG LEN
(integer) 0
```

### 返回值

取决于不同命令，返回不同的值。

## MONITOR

> 可用版本： >= 1.0.0

实时打印出 Redis 服务器接收到的命令，调试用。

### 返回值

总是返回 OK 。

### 示例

```bash
127.0.0.1:6379> MONITOR
OK
# 以第一个打印值为例
# 1378822099.421623 是时间戳
# [0 127.0.0.1:56604] 中的 0 是数据库号码， 127... 是 IP 地址和端口
# "PING" 是被执行的命令
1378822099.421623 [0 127.0.0.1:56604] "PING"
1378822105.089572 [0 127.0.0.1:56604] "SET" "msg" "hello world"
1378822109.036925 [0 127.0.0.1:56604] "SET" "number" "123"
1378822140.649496 [0 127.0.0.1:56604] "SADD" "fruits" "Apple" "Banana" "Cherry"
1378822154.117160 [0 127.0.0.1:56604] "EXPIRE" "msg" "10086"
1378822257.329412 [0 127.0.0.1:56604] "KEYS" "*"
1378822258.690131 [0 127.0.0.1:56604] "DBSIZE"
```

## DEBUG OBJECT key

> 可用版本： >= 1.0.0

DEBUG OBJECT 是一个调试命令，它不应被客户端所使用。

### 返回值

当 key 存在时，返回有关信息。 当 key 不存在时，返回一个错误。

### 示例

```bash
redis> DEBUG OBJECT my_pc
Value at:0xb6838d20 refcount:1 encoding:raw serializedlength:9 lru:283790 lru_seconds_idle:150

redis> DEBUG OBJECT your_mac
(error) ERR no such key
```

## DEBUG SEGFAULT

> 可用版本： >= 1.0.0

执行一个不合法的内存访问从而让 Redis 崩溃，仅在开发时用于 BUG 模拟。

### 返回值

无

### 示例

```bash
redis> DEBUG SEGFAULT
Could not connect to Redis at: Connection refused

not connected>
```

# 思维导图

![redis-调试命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-调试命令.png)

