# redis 列表命令

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

一个列表最多可以包含 2^32 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

## LPUSH key value [value …]

> 可用版本：>=1.0.0

将一个或多个值 value 插入到列表 key 的表头。

如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表头： 比如说，对空列表 mylist 执行命令 LPUSH mylist a b c ，列表的值将是 c b a ，这等同于原子性地执行 LPUSH
mylist a 、 LPUSH mylist b 和 LPUSH mylist c 三个命令。

如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。

当 key 存在但不是列表类型时，返回一个错误。

### 返回值

执行 LPUSH 命令后，列表的长度。

### 示例

```bash
# 加入多个元素

redis> LPUSH mylist a b c
(integer) 3

redis> LRANGE mylist 0 -1
1) "c"
2) "b"
3) "a"
```

## LPUSHX key value

> 可用版本：>=2.2.0

将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表。

和 LPUSH key value [value …] 命令相反，当 key 不存在时， LPUSHX 命令什么也不做。

### 返回值

LPUSHX 命令执行之后，表的长度。

### 示例

```bash
# 对空列表执行 LPUSHX

redis> LLEN greet                       # greet 是一个空列表
(integer) 0

redis> LPUSHX greet "hello"             # 尝试 LPUSHX，失败，因为列表为空
(integer) 0


# 对非空列表执行 LPUSHX

redis> LPUSH greet "hello"              # 先用 LPUSH 创建一个有一个元素的列表
(integer) 1

redis> LPUSHX greet "good morning"      # 这次 LPUSHX 执行成功
(integer) 2

redis> LRANGE greet 0 -1
1) "good morning"
2) "hello"
```

## RPUSH key value [value …]

> 可用版本：>=1.0.0

将一个或多个值 value 插入到列表 key 的表尾(最右边)。

如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表尾：比如对一个空列表 mylist 执行 RPUSH mylist a b c ，得出的结果列表为 a b c ，等同于执行命令 RPUSH mylist a
、 RPUSH mylist b 、 RPUSH mylist c 。

如果 key 不存在，一个空列表会被创建并执行 RPUSH 操作。

当 key 存在但不是列表类型时，返回一个错误。

### 返回值

执行 RPUSH 操作后，表的长度。

### 示例

```bash
# 添加多个元素

redis> RPUSH mylist a b c
(integer) 3

redis> LRANGE mylist 0 -1
1) "a"
2) "b"
3) "c"
```

## RPUSHX key value

> 可用版本：>=2.2.0

将值 value 插入到列表 key 的表尾，当且仅当 key 存在并且是一个列表。

和 RPUSH key value [value …] 命令相反，当 key 不存在时， RPUSHX 命令什么也不做。

### 返回值

RPUSHX 命令执行之后，表的长度。

### 示例

```bash
# key不存在

redis> LLEN greet
(integer) 0

redis> RPUSHX greet "hello"     # 对不存在的 key 进行 RPUSHX，PUSH 失败。
(integer) 0


# key 存在且是一个非空列表

redis> RPUSH greet "hi"         # 先用 RPUSH 插入一个元素
(integer) 1

redis> RPUSHX greet "hello"     # greet 现在是一个列表类型，RPUSHX 操作成功。
(integer) 2

redis> LRANGE greet 0 -1
1) "hi"
2) "hello"
```

## LPOP key

> 可用版本：>=1.0.0

移除并返回列表 key 的头元素。

### 返回值

列表的头元素。 当 key 不存在时，返回 nil 。

### 示例

```bash
redis> RPUSH course algorithm001
(integer) 1

redis> RPUSH course c++101
(integer) 2

redis> LPOP course  # 移除头元素
"algorithm001"
```

## RPOP key

> 可用版本：>=1.0.0

移除并返回列表 key 的尾元素。

### 返回值

列表的尾元素。 当 key 不存在时，返回 nil 。

### 示例

```bash
redis> RPUSH mylist "one"
(integer) 1

redis> RPUSH mylist "two"
(integer) 2

redis> RPUSH mylist "three"
(integer) 3

redis> RPOP mylist           # 返回被弹出的元素
"three"

redis> LRANGE mylist 0 -1    # 列表剩下的元素
1) "one"
2) "two"
```

## RPOPLPUSH source destination

> 可用版本：>=1.2.0

命令 RPOPLPUSH 在一个原子时间内，执行以下两个动作：

- 将列表 source 中的最后一个元素(尾元素)弹出，并返回给客户端。
- 将 source 弹出的元素插入到列表 destination ，作为 destination 列表的的头元素。

### 返回值

被弹出的元素。

### 示例

```bash
# source 和 destination 不同

redis> LRANGE alpha 0 -1         # 查看所有元素
1) "a"
2) "b"
3) "c"
4) "d"

redis> RPOPLPUSH alpha reciver   # 执行一次 RPOPLPUSH 看看
"d"

redis> LRANGE alpha 0 -1
1) "a"
2) "b"
3) "c"

redis> LRANGE reciver 0 -1
1) "d"
```

## LREM key count value

> 可用版本：>=1.0.0

根据参数 count 的值，移除列表中与参数 value 相等的元素。

count 的值可以是以下几种：

- count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
- count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
- count = 0 : 移除表中所有与 value 相等的值。

### 返回值

被移除元素的数量。 因为不存在的 key 被视作空表(empty list)，所以当 key 不存在时， LREM 命令总是返回 0 。

### 示例

```bash
redis> LPUSH greet "morning"
(integer) 1
redis> LPUSH greet "hello"
(integer) 2
redis> LPUSH greet "morning"
(integer) 3
redis> LPUSH greet "hello"
(integer) 4
redis> LPUSH greet "morning"
(integer) 5

redis> LRANGE greet 0 4         # 查看所有元素
1) "morning"
2) "hello"
3) "morning"
4) "hello"
5) "morning"

redis> LREM greet 2 morning     # 移除从表头到表尾，最先发现的两个 morning
(integer) 2                     # 两个元素被移除

redis> LRANGE greet 0 2
1) "hello"
2) "hello"
3) "morning"
```

## LLEN key

> 可用版本：>=1.0.0

返回列表 key 的长度。

如果 key 不存在，则 key 被解释为一个空列表，返回 0 .

如果 key 不是列表类型，返回一个错误。

### 返回值

列表 key 的长度。

### 示例

```bash
# 空列表
redis> LLEN job
(integer) 0


# 非空列表
redis> LPUSH job "cook food"
(integer) 1

redis> LPUSH job "have lunch"
(integer) 2

redis> LLEN job
(integer) 2
```

## LINDEX key index

> 可用版本：>=1.0.0

返回列表 key 中，下标为 index 的元素。

### 返回值

列表中下标为 index 的元素。 如果 index 参数的值不在列表的区间范围内(out of range)，返回 nil 。

### 示例

```bash
redis> LPUSH mylist "World"
(integer) 1

redis> LPUSH mylist "Hello"
(integer) 2

redis> LINDEX mylist 0
"Hello"

redis> LINDEX mylist -1
"World"

redis> LINDEX mylist 3        # index不在 mylist 的区间范围内
(nil)
```

## LINSERT key BEFORE|AFTER pivot value

> 可用版本：>=2.2.0

将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。

当 pivot 不存在于列表 key 时，不执行任何操作。

当 key 不存在时， key 被视为空列表，不执行任何操作。

### 返回值

如果命令执行成功，返回插入操作完成之后，列表的长度。 如果没有找到 pivot ，返回 -1 。 如果 key 不存在或为空列表，返回 0 。

### 示例

```bash
redis> RPUSH mylist "Hello"
(integer) 1

redis> RPUSH mylist "World"
(integer) 2

redis> LINSERT mylist BEFORE "World" "There"
(integer) 3

redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
```

## LSET key index value

> 可用版本：>=1.0.0

将列表 key 下标为 index 的元素的值设置为 value 。

当 index 参数超出范围，或对一个空列表( key 不存在)进行 LSET 时，返回一个错误。

### 返回值

操作成功返回 ok ，否则返回错误信息。

### 示例

```bash
# 对非空列表进行 LSET

redis> LPUSH job "cook food"
(integer) 1

redis> LRANGE job 0 0
1) "cook food"

redis> LSET job 0 "play game"
OK

redis> LRANGE job  0 0
1) "play game"
```

## LRANGE key start stop

> 可用版本：>=1.0.0

返回列表 key 中指定区间内的元素，区间以偏移量 start 和 stop 指定。

下标(index)参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。

### 返回值

一个列表，包含指定区间内的元素。

### 示例

```bash
redis> RPUSH fp-language lisp
(integer) 1

redis> LRANGE fp-language 0 0
1) "lisp"

redis> RPUSH fp-language scheme
(integer) 2

redis> LRANGE fp-language 0 1
1) "lisp"
2) "scheme"
```

## LTRIM key start stop

> 可用版本：>=1.0.0

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

### 返回值

命令执行成功时，返回 ok 。

### 示例

```bash
redis> LRANGE alpha 0 -1       # alpha 是一个包含 5 个字符串的列表
1) "h"
2) "e"
3) "l"
4) "l"
5) "o"

redis> LTRIM alpha 1 -1        # 删除 alpha 列表索引为 0 的元素
OK

redis> LRANGE alpha 0 -1       # "h" 被删除了
1) "e"
2) "l"
3) "l"
4) "o"
```

## BLPOP key [key …] timeout

> 可用版本：>=2.0.0

BLPOP 是列表的阻塞式(blocking)弹出原语。

它是 LPOP key 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BLPOP 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。

### 返回值

如果列表为空，返回一个 nil 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。

### 示例

```bash
redis> EXISTS job                # 确保两个 key 都不存在
(integer) 0
redis> EXISTS command
(integer) 0

redis> BLPOP job command 300     # 因为key一开始不存在，所以操作会被阻塞，直到另一客户端对 job 或者 command 列表进行 PUSH 操作。
1) "job"                         # 这里被 push 的是 job
2) "do my home work"             # 被弹出的值
(26.26s)                         # 等待的秒数

redis> BLPOP job command 5       # 等待超时的情况
(nil)
(5.66s)                          # 等待的秒数
```

## BRPOP key [key …] timeout

> 可用版本：>=2.0.0

BRPOP 是列表的阻塞式(blocking)弹出原语。

它是 RPOP key 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BRPOP 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。

### 返回值

假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。

### 示例

```bash
redis> LLEN course
(integer) 0

redis> RPUSH course algorithm001
(integer) 1

redis> RPUSH course c++101
(integer) 2

redis> BRPOP course 30
1) "course"             # 被弹出元素所属的列表键
2) "c++101"             # 被弹出的元素
```

## BRPOPLPUSH source destination timeout

> 可用版本：>=2.2.0

BRPOPLPUSH 是 RPOPLPUSH source destination 的阻塞版本，当给定列表 source 不为空时， BRPOPLPUSH 的表现和 RPOPLPUSH source destination 一样。

当列表 source 为空时， BRPOPLPUSH 命令将阻塞连接，直到等待超时，或有另一个客户端对 source 执行 LPUSH key value [value …] 或 RPUSH key value [value …]
命令为止。

超时参数 timeout 接受一个以秒为单位的数字作为值。超时参数设为 0 表示阻塞时间可以无限期延长(block indefinitely) 。

### 返回值

假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。

### 示例

```bash
# 非空列表

redis> BRPOPLPUSH msg reciver 500
"hello moto"                        # 弹出元素的值
(3.38s)                             # 等待时长

redis> LLEN reciver
(integer) 1

redis> LRANGE reciver 0 0
1) "hello moto"

# 空列表

redis> BRPOPLPUSH msg reciver 1
(nil)
(1.34s)
```

# 思维导图

![redis-列表命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-列表命令.png)

