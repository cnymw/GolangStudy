# redis 哈希表命令

## HSET hash field value 

> 可用版本：>= 2.0.0

将哈希表 hash 中域 field 的值设置为 value 。

如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 HSET 操作。

如果域 field 已经存在于哈希表中， 那么它的旧值将被新值 value 覆盖。

### 返回值

当 HSET 命令在哈希表中新创建 field 域并成功为它设置值时， 命令返回 1 ； 如果域 field 已经存在于哈希表， 并且 HSET 命令成功使用新值覆盖了它的旧值， 那么命令返回 0 。

### 代码示例

```bash
redis> HSET website google "www.g.cn"
(integer) 1

redis> HGET website google
"www.g.cn"
```

## HSETNX hash field value

> 可用版本：>=2.0.0

当且仅当域 field 尚未存在于哈希表的情况下， 将它的值设置为 value 。

如果给定域已经存在于哈希表当中， 那么命令将放弃执行设置操作。

如果哈希表 hash 不存在， 那么一个新的哈希表将被创建并执行 HSETNX 命令。

### 返回值

HSETNX 命令在设置成功时返回 1 ， 在给定域已经存在而放弃执行设置操作时返回 0 。

### 代码示例

```bash
redis> HSETNX database key-value-store Redis
(integer) 1

redis> HGET database key-value-store
"Redis"
```

## HGET hash field

> 可用版本：>= 2.0.0

返回哈希表中给定域的值。

### 返回值

HGET 命令在默认情况下返回给定域的值。

如果给定域不存在于哈希表中， 又或者给定的哈希表并不存在， 那么命令返回 nil 。

### 代码示例

```bash
redis> HSET homepage redis redis.com
(integer) 1

redis> HGET homepage redis
"redis.com"
```

## HEXISTS hash field

> 可用版本：>=2.0.0

检查给定域 field 是否存在于哈希表 hash 当中。

### 返回值

HEXISTS 命令在给定域存在时返回 1 ， 在给定域不存在时返回 0 。

### 代码示例

```bash
redis> HEXISTS phone myphone
(integer) 0
```

## HDEL key field [field …]

> 可用版本：>=2.0.0

删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。

### 返回值

被成功移除的域的数量，不包括被忽略的域。

### 代码示例

```bash
redis> HDEL abbr a
(integer) 1
```

## HLEN key

> 可用版本：>=2.0.0

返回哈希表 key 中域的数量。

### 返回值

哈希表中域的数量。

当 key 不存在时，返回 0 。

### 代码示例

```bash
redis> HSET db redis redis.com
(integer) 1

redis> HSET db mysql mysql.com
(integer) 1

redis> HLEN db
(integer) 2

redis> HSET db mongodb mongodb.org
(integer) 1

redis> HLEN db
(integer) 3
```

## HSTRLEN key field

> 可用版本：>=3.2.0

返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度（string length）。

如果给定的键或者域不存在， 那么命令返回 0 。

### 返回值

字符串长度，整数。

### 代码示例

```bash
redis> HMSET myhash f1 "HelloWorld" f2 "99" f3 "-256"
OK

redis> HSTRLEN myhash f1
(integer) 10

redis> HSTRLEN myhash f2
(integer) 2

redis> HSTRLEN myhash f3
(integer) 4
```

## HINCRBY key field increment

> 可用版本：>=2.0.0

为哈希表 key 中的域 field 的值加上增量 increment 。

增量也可以为负数，相当于对给定域进行减法操作。

如果 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令。

如果域 field 不存在，那么在执行命令前，域的值被初始化为 0 。

### 返回值

执行 HINCRBY 命令之后，哈希表 key 中域 field 的值。

### 代码示例

```bash
# increment 为正数

redis> HEXISTS counter page_view    # 对空域进行设置
(integer) 0

redis> HINCRBY counter page_view 200
(integer) 200

redis> HGET counter page_view
"200"
```

## HINCRBYFLOAT key field increment

> 可用版本：>=2.6.0

为哈希表 key 中的域 field 加上浮点数增量 increment 。

如果哈希表中没有域 field ，那么 HINCRBYFLOAT 会先将域 field 的值设为 0 ，然后再执行加法操作。

如果键 key 不存在，那么 HINCRBYFLOAT 会先创建一个哈希表，再创建域 field ，最后再执行加法操作。

### 返回值

执行加法操作之后 field 域的值。

### 代码示例

```bash
# 值和增量都是普通小数

redis> HSET mykey field 10.50
(integer) 1
redis> HINCRBYFLOAT mykey field 0.1
"10.6"
```

## HMSET key field value [field value …]

> 可用版本：>=2.0.0

同时将多个 field-value (域-值)对设置到哈希表 key 中。

此命令会覆盖哈希表中已存在的域。

如果 key 不存在，一个空哈希表被创建并执行 HMSET 操作。

### 返回值

如果命令执行成功，返回 OK 。

当 key 不是哈希表(hash)类型时，返回一个错误。

### 代码示例

```bash
redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HGET website google
"www.google.com"

redis> HGET website yahoo
"www.yahoo.com"
```

## HMGET key field [field …]

> 可用版本：>=2.0.0

返回哈希表 key 中，一个或多个给定域的值。

如果给定的域不存在于哈希表，那么返回一个 nil 值。

因为不存在的 key 被当作一个空哈希表来处理，所以对一个不存在的 key 进行 HMGET 操作将返回一个只带有 nil 值的表。

### 返回值

一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样。

### 代码示例

```bash
redis> HMSET pet dog "doudou" cat "nounou"    # 一次设置多个域
OK

redis> HMGET pet dog cat fake_pet             # 返回值的顺序和传入参数的顺序一样
1) "doudou"
2) "nounou"
3) (nil)                                      # 不存在的域返回nil值
```

## HKEYS key

> 可用版本：>=2.0.0

返回哈希表 key 中的所有域。

### 返回值

一个包含哈希表中所有域的表。

当 key 不存在时，返回一个空表。

### 代码示例

```bash
# 哈希表非空

redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HKEYS website
1) "google"
2) "yahoo"
```

## HVALS key

> 可用版本：>=2.0.0

返回哈希表 key 中所有域的值。

### 返回值

一个包含哈希表中所有值的表。

当 key 不存在时，返回一个空表。

### 代码示例

```bash
# 非空哈希表

redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HVALS website
1) "www.google.com"
2) "www.yahoo.com"
```

## HGETALL key

> 可用版本：>=2.0.0

返回哈希表 key 中，所有的域和值。

在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

### 返回值

以列表形式返回哈希表的域和域的值。

若 key 不存在，返回空列表。

### 代码示例

```bash
redis> HSET people jack "Jack Sparrow"
(integer) 1

redis> HSET people gump "Forrest Gump"
(integer) 1

redis> HGETALL people
1) "jack"          # 域
2) "Jack Sparrow"  # 值
3) "gump"
4) "Forrest Gump"
```

## HSCAN key cursor [MATCH pattern] [COUNT count]

> 可用版本：>=2.8.0

HSCAN 迭代散列类型的字段及其关联值。

HSCAN 是一个基于光标的迭代器。这意味着在每次调用命令时，服务器都会返回一个更新的游标，用户需要在下一次调用中将其用作游标参数。

### 返回值

返回散列表的所有元素列表，每个元素包含两个参数，字段 field 和值 value。

### 代码示例

```bash
redis 127.0.0.1:6379> hmset hash name Jack age 33
OK
redis 127.0.0.1:6379> hscan hash 0
1) "0"
2) 1) "name"
   2) "Jack"
   3) "age"
   4) "33"
```

> 参考资料：[redis命令：SCAN](https://redis.io/commands/scan)

# 思维导图

![redis-redis哈希表命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-redis哈希表命令.png)

