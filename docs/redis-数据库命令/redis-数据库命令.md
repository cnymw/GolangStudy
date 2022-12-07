# redis 数据库命令

## DBSIZE

> 可用版本： >= 1.0.0

返回当前数据库的 key 的数量。

### 返回值

当前数据库的 key 的数量。

### 示例

```bash
redis> DBSIZE
(integer) 5

redis> SET new_key "hello_moto"     # 增加一个 key 试试
OK

redis> DBSIZE
(integer) 6
```

## FLUSHDB

> 可用版本： >= 1.0.0

清空当前数据库中的所有 key。

此命令从不失败。

### 返回值

总是返回 OK 。

### 示例

```bash
redis> DBSIZE    # 清空前的 key 数量
(integer) 4

redis> FLUSHDB
OK

redis> DBSIZE    # 清空后的 key 数量
(integer) 0
```

## FLUSHALL

> 可用版本： >= 1.0.0

清空整个 Redis 服务器的数据(删除所有数据库的所有 key )。

此命令从不失败。

### 返回值

总是返回 OK 。

### 示例

```bash
redis> DBSIZE            # 0 号数据库的 key 数量
(integer) 9

redis> SELECT 1          # 切换到 1 号数据库
OK

redis[1]> DBSIZE         # 1 号数据库的 key 数量
(integer) 6

redis[1]> flushall       # 清空所有数据库的所有 key
OK

redis[1]> DBSIZE         # 不但 1 号数据库被清空了
(integer) 0

redis[1]> SELECT 0       # 0 号数据库(以及其他所有数据库)也一样
OK

redis> DBSIZE
(integer) 0
```

## SELECT index

> 可用版本： >= 1.0.0

切换到指定的数据库，数据库索引号 index 用数字值指定，以 0 作为起始索引值。

默认使用 0 号数据库。

### 返回值

OK

### 示例

```bash
redis> SET db_number 0         # 默认使用 0 号数据库
OK

redis> SELECT 1                # 使用 1 号数据库
OK

redis[1]> GET db_number        # 已经切换到 1 号数据库，注意 Redis 现在的命令提示符多了个 [1]
(nil)

redis[1]> SET db_number 1
OK

redis[1]> GET db_number
"1"

redis[1]> SELECT 3             # 再切换到 3 号数据库
OK

redis[3]>                      # 提示符从 [1] 改变成了 [3]
```

## SWAPDB db1 db2

> 可用版本： >= 4.0.0

对换指定的两个数据库， 使得两个数据库的数据立即互换。

### 返回值

OK

### 示例

```bash
# 对换数据库 0 和数据库 1
redis> SWAPDB 0 1
OK
```

# 思维导图

![redis-数据库命令.png](https://cnymw.github.io/GolangStudy/docs/img/redis-数据库命令.png)
