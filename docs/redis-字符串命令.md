# redis 字符串命令

Redis 字符串数据类型的相关命令用于管理 redis 字符串值。

## SET key value[EX seconds][PX milliseconds][NX|XX]

> 可用版本：>=1.0.0

将字符串值 value 关联到 key 。

如果 key 已经持有其他值， set 就覆盖旧值，无视类型。

当 set 命令对一个带有生存时间（TTL）的 key 进行设置之后，该 key 原有的 TTL 将被清除。

### 可选参数

- EX seconds：将键的过期时间设置为`seconds`秒。
- PX milliseconds：将键的过期时间设置为`milliseconds`毫秒。
- NX：只在键不存在时，才对键进行设置操作。
- XX：只在键已经存在时，才对键进行设置操作。

### 返回值

在 Redis 2.6.12 版本以前， SET 命令总是返回 OK 。

从 Redis 2.6.12 版本开始， SET 命令只在设置操作成功完成时才返回 OK ； 如果命令使用了 NX 或者 XX 选项， 但是因为条件没达到而造成设置操作未执行， 那么命令将返回空批量回复（NULL Bulk Reply）。

### 示例

对不存在的键进行设置：

```bash
redis> SET key "value"
OK

redis> GET key
"value"
```

使用 EX 选项：

```bash
redis> SET key-with-expire-time "hello" EX 10086
OK

redis> GET key-with-expire-time
"hello"

redis> TTL key-with-expire-time
(integer) 10069
```

使用 NX 选项：

```bash
redis> SET not-exists-key "value" NX
OK      # 键不存在，设置成功

redis> GET not-exists-key
"value"

redis> SET not-exists-key "new-value" NX
(nil)   # 键已经存在，设置失败

redis> GEt not-exists-key
"value" # 维持原值不变
```

## SETNX key value

> 可用版本：>=1.0.0

只在键 key 不存在的情况下，将键 key 的值设置为 value 。

若键 key 已经存在，则 SETNX 命令不做任何动作。

### 返回值

命令在设置成功时返回 1 ， 设置失败时返回 0 。

### 示例

```bash
redis> EXISTS job                # job 不存在
(integer) 0

redis> SETNX job "programmer"    # job 设置成功
(integer) 1

redis> SETNX job "code-farmer"   # 尝试覆盖 job ，失败
(integer) 0

redis> GET job                   # 没有被覆盖
"programmer"
```

## SETEX keys seconds value

> 可用版本：>=2.0.0

将键 key 的值设置为 value ，并将键 key 的生存时间设置为 seconds 秒钟。

如果键 key 已经存在，那么 SETEX 命令将覆盖已有的值。

 SETEX 和 SET 的不同之处在于 SETEX 是一个原子（atomic）操作， 它可以在同一时间内完成设置值和设置过期时间这两个操作， 因此 SETEX 命令在储存缓存的时候非常实用。

### 返回值

命令在设置成功时返回 OK 。 当 seconds 参数不合法时， 命令将返回一个错误。

### 示例

键 key 已经存在， 使用 SETEX 覆盖旧值：

```bash
redis> SET cd "timeless"
OK

redis> SETEX cd 3000 "goodbye my love"
OK

redis> GET cd
"goodbye my love"

redis> TTL cd
(integer) 2997
```

## PSETEX key milliseconds value

> 可用版本：>=2.6.0

这个命令和 SETEX 命令相似， 但它以毫秒为单位设置 key 的生存时间， 而不是像 SETEX 命令那样以秒为单位进行设置。

### 返回值

命令在设置成功时返回 OK 。

### 示例

```bash
redis> PSETEX mykey 1000 "Hello"
OK

redis> PTTL mykey
(integer) 999

redis> GET mykey
"Hello"
```

## GET key

> 可用版本：>=1.0.0

返回与键 key 相关联的字符串值。

### 返回值

如果键 key 不存在， 那么返回特殊值 nil ； 否则， 返回键 key 的值。

如果键 key 的值并非字符串类型， 那么返回一个错误， 因为 GET 命令只能用于字符串值。

### 示例

```bash
redis> GET db
(nil)

redis> SET db redis
OK

redis> GET db
"redis"
```

## GETSET key value

> 可用版本：>=1.0.0

将键 key 的值设为 value ， 并返回键 key 在被设置之前的旧值。

### 返回值

返回给定键 key 的旧值。

如果键 key 没有旧值， 也即是说， 键 key 在被设置之前并不存在， 那么命令返回 nil 。

当键 key 存在但不是字符串类型时， 命令返回一个错误。

### 示例

```bash
redis> GETSET db mongodb    # 没有旧值，返回 nil
(nil)

redis> GET db
"mongodb"

redis> GETSET db redis      # 返回旧值 mongodb
"mongodb"

redis> GET db
"redis"
```

## STRLEN key

> 可用版本：>=2.2.0

返回键 key 储存的字符串值的长度。

### 返回值

STRLEN 命令返回字符串值的长度。

当键 key 不存在时， 命令返回 0 。

当 key 储存的不是字符串值时， 返回一个错误。

### 示例

获取字符串值的长度：

```bash
redis> SET mykey "Hello world"
OK

redis> STRLEN mykey
(integer) 11
```

## APPEND key value

> 可用版本：>=2.0.0

如果键 key 已经存在并且它的值是一个字符串， APPEND 命令将把 value 追加到键 key 现有值的末尾。

如果 key 不存在， APPEND 就简单地将键 key 的值设为 value ， 就像执行 SET key value 一样。

### 返回值

追加 value 之后， 键 key 的值的长度。

### 示例

对已存在的字符串进行`APPEND`：


```bash
redis> APPEND myphone " - 1110"     # 长度从 5 个字符增加到 12 个字符
(integer) 12

redis> GET myphone
"nokia - 1110"
```

## SETRANGE key offset value

> 可用版本：>=2.2.0

从偏移量 offset 开始， 用 value 参数覆写(overwrite)键 key 储存的字符串值。

 SETRANGE 命令会确保字符串足够长以便将 value 设置到指定的偏移量上， 如果键 key 原来储存的字符串长度比偏移量小(比如字符串只有 5 个字符长，但你设置的 offset 是 10 )， 那么原字符和偏移量之间的空白将用零字节(zerobytes, "\x00" )进行填充。

当生成一个很长的字符串时，Redis 需要分配内存空间，该操作有时候可能会造成服务器阻塞(block)。

### 返回值

SETRANGE 命令会返回被修改之后， 字符串值的长度。

### 示例

对非空字符串执行 SETRANGE 命令：

```bash
redis> SET greeting "hello world"
OK

redis> SETRANGE greeting 6 "Redis"
(integer) 11

redis> GET greeting
"hello Redis"
```

## GETRANGE key start end 

> 可用版本：>=2.4.0

返回键 key 储存的字符串值的指定部分，字符串的截取范围由 start 和 end 两个偏移量决定 (包括 start 和 end 在内)。

负数偏移量表示从字符串的末尾开始计数， -1 表示最后一个字符， -2 表示倒数第二个字符，以此类推。

### 返回值

GETRANGE 命令会返回字符串值的指定部分。

### 示例

```bash
redis> SET greeting "hello, my friend"
OK

redis> GETRANGE greeting 0 4          # 返回索引0-4的字符，包括4。
"hello"

redis> GETRANGE greeting -1 -5        # 不支持回绕操作
""

redis> GETRANGE greeting -3 -1        # 负数索引
"end"

redis> GETRANGE greeting 0 -1         # 从第一个到最后一个
"hello, my friend"

redis> GETRANGE greeting 0 1008611    # 值域范围不超过实际字符串，超过部分自动被符略
"hello, my friend"
```

## INCR key

> 可用版本：>=1.0.0

为键 key 储存的数字值加上一。

如果键 key 不存在， 那么它的值会先被初始化为 0 ， 然后再执行 INCR 命令。

如果键 key 储存的值不能被解释为数字， 那么 INCR 命令将返回一个错误。

### 返回值

INCR 命令会返回键 key 在执行加一操作之后的值。

### 示例

```bash
redis> SET page_view 20
OK

redis> INCR page_view
(integer) 21

redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
"21"
```

## INCRBY key increment

> 可用版本：>=1.0.0

为键 key 储存的数字值加上增量 increment 。

如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 INCRBY 命令。

如果键 key 储存的值不能被解释为数字， 那么 INCRBY 命令将返回一个错误。

### 返回值

在加上增量 increment 之后， 键 key 当前的值。

### 示例

```bash
redis> SET rank 50
OK

redis> INCRBY rank 20
(integer) 70

redis> GET rank
"70"
```

## INCRBYFLOAT key increment

> 可用版本：>=2.6.0

为键 key 储存的值加上浮点数增量 increment 。

如果键 key 不存在， 那么 INCRBYFLOAT 会先将键 key 的值设为 0 ， 然后再执行加法操作。

如果命令执行成功， 那么键 key 的值会被更新为执行加法计算之后的新值， 并且新值会以字符串的形式返回给调用者。

### 返回值

在加上增量 increment 之后， 键 key 的值。

### 示例

```bash
redis> GET decimal
"3.0"

redis> INCRBYFLOAT decimal 2.56
"5.56"

redis> GET decimal
"5.56"
```

## DECR key

> 可用版本：>=1.0.0

为键 key 储存的数字值减去一。

如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 DECR 操作。

如果键 key 储存的值不能被解释为数字， 那么 DECR 命令将返回一个错误。

### 返回值

DECR 命令会返回键 key 在执行减一操作之后的值。

### 示例

```bash
redis> SET failure_times 10
OK

redis> DECR failure_times
(integer) 9
```

## DECRBY key decrement

> 可用版本：>=1.0.0

将键 key 储存的整数值减去减量 decrement 。

如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 DECRBY 命令。

如果键 key 储存的值不能被解释为数字， 那么 DECRBY 命令将返回一个错误。

### 返回值

DECRBY 命令会返回键在执行减法操作之后的值。

### 示例

```bash
redis> SET count 100
OK

redis> DECRBY count 20
(integer) 80
```

## MSET key value [key value …]

> 可用版本：>=1.0.1

同时为多个键设置值。

如果某个给定键已经存在， 那么 MSET 将使用新值去覆盖旧值， 如果这不是你所希望的效果， 请考虑使用 MSETNX 命令， 这个命令只会在所有给定键都不存在的情况下进行设置。

MSET 是一个原子性(atomic)操作， 所有给定键都会在同一时间内被设置， 不会出现某些键被设置了但是另一些键没有被设置的情况。

### 返回值

MSET 命令总是返回 OK 。

### 示例

```bash
redis> MSET date "2012.3.30" time "11:00 a.m." weather "sunny"
OK

redis> MGET date time weather
1) "2012.3.30"
2) "11:00 a.m."
3) "sunny"
```

## MSETNX key value [key value …]

> 可用版本：>=1.0.1

当且仅当所有给定键都不存在时， 为所有给定键设置值。

即使只有一个给定键已经存在， MSETNX 命令也会拒绝执行对所有键的设置操作。

MSETNX 是一个原子性(atomic)操作， 所有给定键要么就全部都被设置， 要么就全部都不设置， 不可能出现第三种状态。

### 返回值

当所有给定键都设置成功时， 命令返回 1 ； 如果因为某个给定键已经存在而导致设置未能成功执行， 那么命令返回 0 。

### 示例

```bash
redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
(integer) 1

redis> MGET rmdbs nosql key-value-store
1) "MySQL"
2) "MongoDB"
3) "redis"
```

## MGET key [key …]

> 可用版本：>=1.0.0

返回给定的一个或多个字符串键的值。

如果给定的字符串键里面， 有某个键不存在， 那么这个键的值将以特殊值 nil 表示。

### 返回值

MGET 命令将返回一个列表， 列表中包含了所有给定键的值。

### 示例

```bash
redis> SET redis redis.com
OK

redis> SET mongodb mongodb.org
OK

redis> MGET redis mongodb
1) "redis.com"
2) "mongodb.org"

redis> MGET redis mongodb mysql     # 不存在的 mysql 返回 nil
1) "redis.com"
2) "mongodb.org"
3) (nil)
```

# 思维导图

![redis-redis字符串命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-redis字符串命令.png)



