# redis 集合命令

## SADD key member [member …]

> 可用版本：>=1.0.0

将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。

假如 key 不存在，则创建一个只包含 member 元素作成员的集合。

当 key 不是集合类型时，返回一个错误。

### 返回值

被添加到集合中的新元素的数量，不包括被忽略的元素。

### 代码示例

```bash
# 添加单个元素

redis> SADD bbs "discuz.net"
(integer) 1


# 添加重复元素

redis> SADD bbs "discuz.net"
(integer) 0
```

## SISMEMBER key member

> 可用版本：>=1.0.0

判断 member 元素是否集合 key 的成员。

### 返回值

如果 member 元素是集合的成员，返回 1 。 如果 member 元素不是集合的成员，或 key 不存在，返回 0 。

### 代码示例

```bash
redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SISMEMBER joe's_movies "bet man"
(integer) 0

redis> SISMEMBER joe's_movies "Fast Five"
(integer) 1
```

## SPOP key

> 可用版本：>=1.0.0

移除并返回集合中的一个随机元素。

### 返回值

被移除的随机元素。 当 key 不存在或 key 是空集时，返回 nil 。

### 代码示例

```bash
redis> SMEMBERS db
1) "MySQL"
2) "MongoDB"
3) "Redis"

redis> SPOP db
"Redis"
```

## SRANDMEMBER key [count]

> 可用版本：>=1.0.0

如果命令执行时，只提供了 key 参数，那么返回集合中的一个随机元素。

该操作和 SPOP key 相似，但 SPOP key 将随机元素从集合中移除并返回，而 SRANDMEMBER 则仅仅返回随机元素，而不对集合进行任何改动。

### 返回值

只提供 key 参数时，返回一个元素；如果集合为空，返回 nil 。 如果提供了 count 参数，那么返回一个数组；如果集合为空，返回空数组。

### 代码示例

```bash
# 添加元素

redis> SADD fruit apple banana cherry
(integer) 3

# 只给定 key 参数，返回一个随机元素

redis> SRANDMEMBER fruit
"cherry"

redis> SRANDMEMBER fruit
"apple"
```

## SREM key member [member …]

> 可用版本：>=1.0.0

移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。

### 返回值

被成功移除的元素的数量，不包括被忽略的元素。

### 代码示例

```bash
# 测试数据

redis> SMEMBERS languages
1) "c"
2) "lisp"
3) "python"
4) "ruby"


# 移除单个元素

redis> SREM languages ruby
(integer) 1
```

## SMOVE source destination member

> 可用版本：>=1.0.0

将 member 元素从 source 集合移动到 destination 集合。

### 返回值

如果 member 元素被成功移除，返回 1 。 如果 member 元素不是 source 集合的成员，并且没有任何操作对 destination 集合执行，那么返回 0 。

### 代码示例

```bash
redis> SMEMBERS songs
1) "Billie Jean"
2) "Believe Me"

redis> SMEMBERS my_songs
(empty list or set)

redis> SMOVE songs my_songs "Believe Me"
(integer) 1

redis> SMEMBERS songs
1) "Billie Jean"

redis> SMEMBERS my_songs
1) "Believe Me"
```

## SCARD key

> 可用版本：>=1.0.0

返回集合 key 的基数(集合中元素的数量)。

### 返回值

集合的基数。 当 key 不存在时，返回 0 。

### 代码示例

```bash
redis> SADD tool pc printer phone
(integer) 3

redis> SCARD tool   # 非空集合
(integer) 3

redis> DEL tool
(integer) 1

redis> SCARD tool   # 空集合
(integer) 0
```

## SMEMBERS key

> 可用版本：>=1.0.0

返回集合 key 中的所有成员。

### 返回值

集合中的所有成员。

### 代码示例

```bash
# 非空集合

redis> SADD language Ruby Python Clojure
(integer) 3

redis> SMEMBERS language
1) "Python"
2) "Ruby"
3) "Clojure"
```

## SSCAN key cursor [MATCH pattern] [COUNT count]

> 参考资料：[redis命令：SCAN](https://redis.io/commands/scan)

