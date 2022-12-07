# redis 集合命令

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

集合对象的编码可以是 intset 或者 hashtable。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)。

## SADD key member [member …]

> 可用版本：>=1.0.0

将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。

假如 key 不存在，则创建一个只包含 member 元素作成员的集合。

当 key 不是集合类型时，返回一个错误。

### 返回值

被添加到集合中的新元素的数量，不包括被忽略的元素。

### 示例

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

### 示例

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

### 示例

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

### 示例

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

### 示例

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

### 示例

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

### 示例

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

### 示例

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

## SINTER key [key …]

> 可用版本：>=1.0.0

返回一个集合的全部成员，该集合是所有给定集合的交集。

不存在的 key 被视为空集。

当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。

### 返回值

交集成员的列表。

### 示例

```bash
redis> SMEMBERS group_1
1) "LI LEI"
2) "TOM"
3) "JACK"

redis> SMEMBERS group_2
1) "HAN MEIMEI"
2) "JACK"

redis> SINTER group_1 group_2
1) "JACK"
```

## SINTERSTORE destination key [key …]

> 可用版本：>=1.0.0

这个命令类似于 SINTER key [key …] 命令，但它将结果保存到 destination 集合，而不是简单地返回结果集。

如果 destination 集合已经存在，则将其覆盖。

destination 可以是 key 本身。

### 返回值

结果集中的成员数量。

### 示例

```bash
redis> SMEMBERS songs
1) "good bye joe"
2) "hello,peter"

redis> SMEMBERS my_songs
1) "good bye joe"
2) "falling"

redis> SINTERSTORE song_interset songs my_songs
(integer) 1

redis> SMEMBERS song_interset
1) "good bye joe"
```

## SUNION key [key …]

> 可用版本：>=1.0.0

返回一个集合的全部成员，该集合是所有给定集合的并集。

不存在的 key 被视为空集。

### 返回值

并集成员的列表。

### 示例

```bash
redis> SMEMBERS songs
1) "Billie Jean"

redis> SMEMBERS my_songs
1) "Believe Me"

redis> SUNION songs my_songs
1) "Billie Jean"
2) "Believe Me"
```

## SUNIONSTORE destination key [key …]

> 可用版本：>=1.0.0

这个命令类似于 SUNION key [key …] 命令，但它将结果保存到 destination 集合，而不是简单地返回结果集。

如果 destination 已经存在，则将其覆盖。

destination 可以是 key 本身。

### 返回值

结果集中的元素数量。

### 示例

```bash
redis> SMEMBERS NoSQL
1) "MongoDB"
2) "Redis"

redis> SMEMBERS SQL
1) "sqlite"
2) "MySQL"

redis> SUNIONSTORE db NoSQL SQL
(integer) 4

redis> SMEMBERS db
1) "MySQL"
2) "sqlite"
3) "MongoDB"
4) "Redis"
```

## SDIFF key [key …]

> 可用版本：>=1.0.0

返回一个集合的全部成员，该集合是所有给定集合之间的差集。

不存在的 key 被视为空集。

### 返回值

一个包含差集成员的列表。

### 示例

```bash
redis> SMEMBERS peter's_movies
1) "bet man"
2) "start war"
3) "2012"

redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SDIFF peter's_movies joe's_movies
1) "bet man"
2) "start war"
```

## SDIFFSTORE destination key [key …]

> 可用版本：>=1.0.0

这个命令的作用和 SDIFF key [key …] 类似，但它将结果保存到 destination 集合，而不是简单地返回结果集。

如果 destination 集合已经存在，则将其覆盖。

destination 可以是 key 本身。

### 返回值

结果集中的元素数量。

### 示例

```bash
redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SMEMBERS peter's_movies
1) "bet man"
2) "start war"
3) "2012"

redis> SDIFFSTORE joe_diff_peter joe's_movies peter's_movies
(integer) 2

redis> SMEMBERS joe_diff_peter
1) "hi, lady"
2) "Fast Five"
```

# 思维导图

![redis-集合命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-集合命令.png)

