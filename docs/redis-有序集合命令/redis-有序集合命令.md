# redis 有序集合命令

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)。

## ZADD key score member [[score member] [score member] …]

> 可用版本： >= 1.2.0

将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

如果某个 member 已经是有序集的成员，那么更新这个 member 的 score 值，并通过重新插入这个 member 元素，来保证该 member 在正确的位置上。

score 值可以是整数值或双精度浮点数。

### 返回值

被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

### 示例

```bash
# 添加单个元素
redis> ZADD page_rank 10 google.com
(integer) 1

# 添加多个元素
redis> ZADD page_rank 9 baidu.com 8 bing.com
(integer) 2

redis> ZRANGE page_rank 0 -1 WITHSCORES
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"
```

---

## ZSCORE key member

> 可用版本： >= 1.2.0

返回有序集 key 中，成员 member 的 score 值。

### 返回值

member 成员的 score 值，以字符串形式表示。

### 示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
1) "tom"
2) "2000"
3) "peter"
4) "3500"
5) "jack"
6) "5000"

redis> ZSCORE salary peter              # 注意返回值是字符串
"3500"
```

---

## ZINCRBY key increment member

> 可用版本： >= 1.2.0

为有序集 key 的成员 member 的 score 值加上增量 increment 。

### 返回值

member 成员的新 score 值，以字符串形式表示。

### 示例

```bash
redis> ZSCORE salary tom
"2000"

redis> ZINCRBY salary 2000 tom   # tom 加薪啦！
"4000"
```

---

## ZCARD key

> 可用版本： >= 1.2.0

返回有序集 key 的基数。

### 返回值

当 key 存在且是有序集类型时，返回有序集的基数。

### 示例

```bash
redis > ZADD salary 2000 tom    # 添加一个成员
(integer) 1

redis > ZCARD salary
(integer) 1
```

---

## ZCOUNT key min max

> 可用版本： >= 2.0.0

返回有序集 key 中，score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max)的成员的数量。

### 返回值

score 值在 min 和 max 之间的成员的数量。

### 示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
1) "jack"
2) "2000"
3) "peter"
4) "3500"
5) "tom"
6) "5000"

redis> ZCOUNT salary 2000 5000          # 计算薪水在 2000-5000 之间的人数
(integer) 3
```

---

## ZRANGE key start stop [WITHSCORES]

> 可用版本： >= 1.2.0

返回有序集 key 中，指定区间内的成员。

其中成员的位置按 score 值递增(从小到大)来排序。

具有相同 score 值的成员按字典序(lexicographical order)来排列。

可以通过使用 WITHSCORES 选项，来让成员和它的 score 值一并返回，返回列表以 value1,score1, ..., valueN,scoreN 的格式表示。客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

### 返回值

指定区间内，带有 score 值(可选)的有序集成员的列表。

### 示例

```bash
redis > ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"
```

---

## ZREVRANGE key start stop [WITHSCORES]

> 可用版本： >= 1.2.0

返回有序集 key 中，指定区间内的成员。

其中成员的位置按 score 值递减(从大到小)来排列。具有相同 score 值的成员按字典序的逆序(reverse lexicographical order)排列。

### 返回值

指定区间内，带有 score 值(可选)的有序集成员的列表。

### 示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES        # 递增排列
1) "peter"
2) "3500"
3) "tom"
4) "4000"
5) "jack"
6) "5000"

redis> ZREVRANGE salary 0 -1 WITHSCORES     # 递减排列
1) "jack"
2) "5000"
3) "tom"
4) "4000"
5) "peter"
6) "3500"
```

---

## ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

> 可用版本： >= 1.0.5

返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。

具有相同 score 值的成员按字典序(lexicographical order)来排列(该属性是有序集提供的，不需要额外的计算)。

可选的 LIMIT 参数指定返回结果的数量及区间(就像SQL中的 SELECT LIMIT offset, count )，注意当 offset 很大时，定位 offset 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。

### 区间及无限

min 和 max 可以是 -inf 和 +inf，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用 ZRANGEBYSCORE 这类命令。

默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 ( 符号来使用可选的开区间 (小于或大于)。

举个例子：

```bash
ZRANGEBYSCORE zset (1 5
```

返回所有符合条件 1 < score <= 5 的成员，而

```bash
ZRANGEBYSCORE zset (5 (10
```

则返回所有符合条件 5 < score < 10 的成员。

### 返回值

指定区间内，带有 score 值(可选)的有序集成员的列表。

### 示例

```bash
redis> ZADD salary 2500 jack                        # 测试数据
(integer) 0
redis> ZADD salary 5000 tom
(integer) 0
redis> ZADD salary 12000 peter
(integer) 0

redis> ZRANGEBYSCORE salary -inf +inf               # 显示整个有序集
1) "jack"
2) "tom"
3) "peter"
```

---

## ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

> 可用版本：>= 2.2.0

返回有序集 key 中，score 值介于 max 和 min 之间(默认包括等于 max 或 min)的所有的成员。有序集成员按 score 值递减(从大到小)的次序排列。

具有相同 score 值的成员按字典序的逆序(reverse lexicographical order)排列。

### 返回值

指定区间内，带有 score 值(可选)的有序集成员的列表。

### 示例

```bash
redis > ZADD salary 10086 jack
(integer) 1
redis > ZADD salary 5000 tom
(integer) 1
redis > ZADD salary 7500 peter
(integer) 1
redis > ZADD salary 3500 joe
(integer) 1

redis > ZREVRANGEBYSCORE salary +inf -inf   # 逆序排列所有成员
1) "jack"
2) "peter"
3) "tom"
4) "joe"

redis > ZREVRANGEBYSCORE salary 10000 2000  # 逆序排列薪水介于 10000 和 2000 之间的成员
1) "peter"
2) "tom"
3) "joe"
```

---

## ZRANK key member

> 可用版本：>= 2.0.0

返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列。

### 返回值

如果 member 是有序集 key 的成员，返回 member 的排名。

### 示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES        # 显示所有成员及其 score 值
1) "peter"
2) "3500"
3) "tom"
4) "4000"
5) "jack"
6) "5000"

redis> ZRANK salary tom                     # 显示 tom 的薪水排名，第二
(integer) 1
```

---

## ZREVRANK key member

> 可用版本：>= 2.0.0

返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从大到小)排序。

### 返回值

如果 member 是有序集 key 的成员，返回 member 的排名。

### 示例

```bash
redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES     # 测试数据
1) "jack"
2) "2000"
3) "peter"
4) "3500"
5) "tom"
6) "5000"

redis> ZREVRANK salary peter     # peter 的工资排第二
(integer) 1

redis> ZREVRANK salary tom       # tom 的工资最高
(integer) 0
```

---

## ZREM key member [member …]

> 可用版本：>= 1.2.0

移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。

### 返回值

被成功移除的成员的数量，不包括被忽略的成员。

### 示例

```bash
redis> ZRANGE page_rank 0 -1 WITHSCORES
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 移除单个元素
redis> ZREM page_rank google.com
(integer) 1
```

---

## ZREMRANGEBYRANK key start stop

> 可用版本：>= 2.0.0

移除有序集 key 中，指定排名(rank)区间内的所有成员。

区间分别以下标参数 start 和 stop 指出，包含 start 和 stop 在内。

### 返回值

被移除成员的数量。

### 示例

```bash
redis> ZADD salary 2000 jack
(integer) 1
redis> ZADD salary 5000 tom
(integer) 1
redis> ZADD salary 3500 peter
(integer) 1

redis> ZREMRANGEBYRANK salary 0 1       # 移除下标 0 至 1 区间内的成员
(integer) 2
```

---

## ZREMRANGEBYSCORE key min max

> 可用版本：>= 1.2.0

移除有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max)的成员。

### 返回值

被移除成员的数量。

### 示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES          # 显示有序集内所有成员及其 score 值
1) "tom"
2) "2000"
3) "peter"
4) "3500"
5) "jack"
6) "5000"

redis> ZREMRANGEBYSCORE salary 1500 3500      # 移除所有薪水在 1500 到 3500 内的员工
(integer) 2
```

---

## ZRANGEBYLEX key min max [LIMIT offset count]

> 可用版本： >= 2.8.9

当有序集合的所有成员都具有相同的分值时，有序集合的元素会根据成员的字典序（lexicographical ordering）来进行排序，而这个命令则可以返回给定的有序集合键 key 中，值介于 min 和 max 之间的成员。

如果有序集合里面的成员带有不同的分值，那么命令返回的结果是未指定的（unspecified）。

可选的 LIMIT offset count 参数用于获取指定范围内的匹配元素（就像 SQL 中的 SELECT LIMIT offset count 语句）。需要注意的一点是，如果 offset 参数的值非常大的话，那么命令在返回结果之前，需要先遍历至 offset 所指定的位置，这个操作会为命令加上最多 O(N) 复杂度。

### 如何指定范围区间

合法的 min 和 max 参数必须包含 ( 或者 [ ，其中 ( 表示开区间（指定的值不会被包含在范围之内），而 [ 则表示闭区间（指定的值会被包含在范围之内）。

特殊值 + 和 - 在 min 参数以及 max 参数中具有特殊的意义，其中 + 表示正无限，而 - 表示负无限。因此，向一个所有成员的分值都相同的有序集合发送命令 ZRANGEBYLEX <zset> - + ，命令将返回有序集合中的所有元素。

### 返回值

返回一个列表，列表里面包含了有序集合在指定范围内的成员。

### 示例

```bash
redis> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
(integer) 7

redis> ZRANGEBYLEX myzset - [c
1) "a"
2) "b"
3) "c"

redis> ZRANGEBYLEX myzset - (c
1) "a"
2) "b"

redis> ZRANGEBYLEX myzset [aaa (g
1) "b"
2) "c"
3) "d"
4) "e"
5) "f"
```

---

## ZLEXCOUNT key min max

> 可用版本： >= 2.8.9

对于一个所有成员的分值都相同的有序集合键 key 来说，这个命令会返回该集合中，成员介于 min 和 max 范围内的元素数量。

### 返回值

返回指定范围内的元素数量。

### 示例

```bash
redis> ZADD myzset 0 a 0 b 0 c 0 d 0 e
(integer) 5

redis> ZADD myzset 0 f 0 g
(integer) 2

redis> ZLEXCOUNT myzset - +
(integer) 7
```

---

## ZREMRANGEBYLEX key min max

> 可用版本：>= 2.8.9

对于一个所有成员的分值都相同的有序集合键 key 来说，这个命令会移除该集合中，成员介于 min 和 max 范围内的所有元素。

### 返回值

被移除的元素数量。

### 示例

```bash
redis> ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
(integer) 5

redis> ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
(integer) 5

redis> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "alpha"
4) "b"
5) "c"
6) "d"
7) "e"
8) "foo"
9) "zap"
10) "zip"

redis> ZREMRANGEBYLEX myzset [alpha [omega
(integer) 6

redis> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "zap"
4) "zip"
```

---

## ZUNIONSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]

> 可用版本：>= 2.0.0

计算给定的一个或多个有序集的并集，其中给定 key 的数量必须以 numkeys 参数指定，并将该并集(结果集)储存到 destination。

默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之和 。

### WEIGHTS

使用 WEIGHTS 选项，你可以为每个给定有序集分别指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 score 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。

如果没有指定 WEIGHTS 选项，乘法因子默认设置为 1 。

### AGGREGATE

使用 AGGREGATE 选项，你可以指定并集的结果集的聚合方式。

默认使用的参数 SUM，可以将所有集合中某个成员的 score 值之和作为结果集中该成员的 score 值；

使用参数 MIN，可以将所有集合中某个成员的最小 score 值作为结果集中该成员的 score 值；而参数 MAX 则是将所有集合中某个成员的最大 score 值作为结果集中该成员的 score 值。

### 返回值

保存到 destination 的结果集的基数。

### 示例

```bash
redis> ZRANGE programmer 0 -1 WITHSCORES
1) "peter"
2) "2000"
3) "jack"
4) "3500"
5) "tom"
6) "5000"

redis> ZRANGE manager 0 -1 WITHSCORES
1) "herry"
2) "2000"
3) "mary"
4) "3500"
5) "bob"
6) "4000"

redis> ZUNIONSTORE salary 2 programmer manager WEIGHTS 1 3   # 公司决定加薪。。。除了程序员。。。
(integer) 6
```

---

## ZINTERSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]

> 可用版本： >= 2.0.0

计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination 。

默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之和.

### 返回值

保存到 destination 的结果集的基数。

### 示例

```bash
redis > ZADD mid_test 70 "Li Lei"
(integer) 1
redis > ZADD mid_test 70 "Han Meimei"
(integer) 1
redis > ZADD mid_test 99.5 "Tom"
(integer) 1

redis > ZADD fin_test 88 "Li Lei"
(integer) 1
redis > ZADD fin_test 75 "Han Meimei"
(integer) 1
redis > ZADD fin_test 99.5 "Tom"
(integer) 1

redis > ZINTERSTORE sum_point 2 mid_test fin_test
(integer) 3

redis > ZRANGE sum_point 0 -1 WITHSCORES     # 显示有序集内所有成员及其 score 值
1) "Han Meimei"
2) "145"
3) "Li Lei"
4) "158"
5) "Tom"
6) "199"
```

---

## 思维导图

![redis-有序集合命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-有序集合命令.png)

