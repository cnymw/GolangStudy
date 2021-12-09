# redis Hyperloglog 命令

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

## PFADD key element [element …]

> 可用版本： >= 2.8.9

将任意数量的元素添加到指定的 HyperLogLog 里面。

作为这个命令的副作用， HyperLogLog 内部可能会被更新， 以便反映一个不同的唯一元素估计数量（也即是集合的基数）。

如果 HyperLogLog 估计的近似基数（approximated cardinality）在命令执行之后出现了变化， 那么命令返回 1 ， 否则返回 0 。 如果命令执行时给定的键不存在， 那么程序将先创建一个空的
HyperLogLog 结构， 然后再执行命令。

调用 PFADD key element [element …] 命令时可以只给定键名而不给定元素：

- 如果给定键已经是一个 HyperLogLog ， 那么这种调用不会产生任何效果；
- 但如果给定的键不存在， 那么命令会创建一个空的 HyperLogLog ， 并向客户端返回 1 。

### 返回值

如果 HyperLogLog 的内部储存被修改了， 那么返回 1 ， 否则返回 0 。

### 示例

```bash
redis> PFADD  databases  "Redis"  "MongoDB"  "MySQL"
(integer) 1

redis> PFCOUNT  databases
(integer) 3

redis> PFADD  databases  "Redis"    # Redis 已经存在，不必对估计数量进行更新
(integer) 0

redis> PFCOUNT  databases    # 元素估计数量没有变化
(integer) 3

redis> PFADD  databases  "PostgreSQL"    # 添加一个不存在的元素
(integer) 1

redis> PFCOUNT  databases    # 估计数量增一
4
```

## PFCOUNT key [key …]

> 可用版本： >= 2.8.9

当 PFCOUNT key [key …] 命令作用于单个键时， 返回储存在给定键的 HyperLogLog 的近似基数， 如果键不存在， 那么返回 0 。

当 PFCOUNT key [key …] 命令作用于多个键时， 返回所有给定 HyperLogLog 的并集的近似基数， 这个近似基数是通过将所有给定 HyperLogLog 合并至一个临时 HyperLogLog 来计算得出的。

通过 HyperLogLog 数据结构， 用户可以使用少量固定大小的内存， 来储存集合中的唯一元素 （每个 HyperLogLog 只需使用 12k 字节内存，以及几个字节的内存来储存键本身）。

命令返回的可见集合（observed set）基数并不是精确值， 而是一个带有 0.81% 标准错误（standard error）的近似值。

### 返回值

给定 HyperLogLog 包含的唯一元素的近似数量。

### 示例

```bash
redis> PFADD  databases  "Redis"  "MongoDB"  "MySQL"
(integer) 1

redis> PFCOUNT  databases
(integer) 3

redis> PFADD  databases  "Redis"    # Redis 已经存在，不必对估计数量进行更新
(integer) 0

redis> PFCOUNT  databases    # 元素估计数量没有变化
(integer) 3

redis> PFADD  databases  "PostgreSQL"    # 添加一个不存在的元素
(integer) 1

redis> PFCOUNT  databases    # 估计数量增一
4
```

## PFMERGE destkey sourcekey [sourcekey …]

> 可用版本： >= 2.8.9

将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。

合并得出的 HyperLogLog 会被储存在 destkey 键里面， 如果该键并不存在， 那么命令在执行之前， 会先为该键创建一个空的 HyperLogLog 。

### 返回值

返回 OK 。

### 示例

```bash
redis> PFADD  nosql  "Redis"  "MongoDB"  "Memcached"
(integer) 1

redis> PFADD  RDBMS  "MySQL" "MSSQL" "PostgreSQL"
(integer) 1

redis> PFMERGE  databases  nosql  RDBMS
OK

redis> PFCOUNT  databases
(integer) 6
```

# 思维导图

![redis-Hyperloglog命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-Hyperloglog命令.png)

