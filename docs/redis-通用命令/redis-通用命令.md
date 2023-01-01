# redis 通用命令

Redis 通用命令用于管理 redis 的键。

## 1. COPY source destination [DB destination-db] [REPLACE]

> 可用版本：>= 6.2.0

用于将存储在源 key 中的值复制到目标 key。

### 返回值

如果复制成功，则返回 1，否则返回 0。

### 示例

```bash
SET dolly "sheep"
COPY dolly clone
```

## 2. DEL KEY_NAME

> 可用版本：>= 1.0.0

用于删除已存在的键。不存在的 key 会被忽略。

### 返回值

被删除 key 的数量。

### 示例

```bash
redis 127.0.0.1:6379> DEL rediskey
(integer) 1
```

---

## 3. DUMP KEY_NAME

> 可用版本：>= 2.6.0

用于序列化给定 key ，并返回被序列化的值。

### 返回值

如果 key 不存在，那么返回 nil 。 否则，返回序列化之后的值。

### 示例

```bash
redis> SET greeting "hello, dumping world!"
OK

redis> DUMP greeting
"\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"
```

---

## 4. EXISTS KEY_NAME

> 可用版本：>= 1.0.0

用于检查给定 key 是否存在。

### 返回值

若 key 存在返回 1 ，否则返回 0 。

### 示例

```bash
redis 127.0.0.1:6379> set rediskey newkey
OK

redis 127.0.0.1:6379> EXISTS rediskey
(integer) 1
```

---

## 5. Expire KEY_NAME TIME_IN_SECONDS

> 可用版本：>= 1.0.0

用于设置 key 的过期时间，key 过期后将不再可用。单位以秒计。

### 返回值

设置成功返回 1 。 当 key 不存在或者不能为 key 设置过期时间时返回 0 。

### 示例

```bash
redis 127.0.0.1:6379> EXPIRE rediskey 60
(integer) 1
```

---

## 6. Expireat KEY_NAME TIME_IN_UNIX_TIMESTAMP

> 可用版本：>= 1.0.0

用于以 UNIX 时间戳(unix timestamp)格式设置 key 的过期时间。key 过期后将不再可用。

### 返回值

设置成功返回 1 。 当 key 不存在或者不能为 key 设置过期时间时返回 0 。

### 示例

```bash
redis 127.0.0.1:6379> EXPIREAT rediskey 1293840000
(integer) 1
```

---

## 7. EXPIRETIME key

> 可用版本：>= 7.0.0

返回给定 key 将过期的 Unix timestamp（自 1970 年 1 月 1 日起）。

### 返回值

返回过期 Unix timestamp。

### 示例

```bash
redis> EXPIREAT mykey 33177117420
(integer) 1
redis> EXPIRETIME mykey
(integer) 33177117420
```

---

## 8. KEYS PATTERN

> 可用版本：>= 1.0.0

用于查找所有符合给定模式 pattern 的 key。

### 返回值

符合给定模式的 key 列表。

### 示例

```bash
redis 127.0.0.1:6379> KEYS rediskey*
1) "rediskey3"
2) "rediskey1"
3) "rediskey2"
```

---

## 6. PEXPIRE key milliseconds

> 可用版本：>= 2.6.0

和 EXPIRE 命令的作用类似，但是它以毫秒为单位设置 key 的生存时间，而不像 EXPIRE 命令那样，以秒为单位。

### 返回值

设置成功，返回 1 。key 不存在或设置失败，返回 0。

### 示例

```bash
redis> SET mykey "Hello"
"OK"

redis> PEXPIRE mykey 1500
(integer) 1
```

---

## 7. PEXPIREAT KEY_NAME TIME_IN_MILLISECONDS_IN_UNIX_TIMESTAMP

用于设置 key 的过期时间，以毫秒计。key 过期后将不再可用。

> 可用版本：>= 1.0.0

### 返回值

设置成功返回 1。当 key 不存在或者不能为 key 设置过期时间时返回 0 。

### 示例

```bash
redis 127.0.0.1:6379> SET rediskey redis
OK

redis 127.0.0.1:6379> PEXPIREAT rediskey 1555555555005
(integer) 1
```

---

## 9. MOVE KEY_NAME DESTINATION_DATABASE

用于将当前数据库的 key 移动到给定的数据库 db 当中。

> 可用版本：>= 1.0.0

### 返回值

移动成功返回 1 ，失败则返回 0 。

### 示例

```bash
redis> SELECT 0                             # redis默认使用数据库 0，为了清晰起见，这里再显式指定一次。
OK

redis> SET song "secret base - Zone"
OK

redis> MOVE song 1                          # 将 song 移动到数据库 1
(integer) 1
```

---

## 10. PERSIST KEY_NAME

用于移除给定 key 的过期时间，使得 key 永不过期。

> 可用版本：>= 2.2.0

### 返回值

当过期时间移除成功时，返回 1 。 如果 key 不存在或 key 没有设置过期时间，返回 0 。

### 示例

```bash
redis> PERSIST mykey    # 移除 key 的生存时间
(integer) 1

redis> TTL mykey
(integer) -1
```

---

## 11. PTTL KEY_NAME

以毫秒为单位返回 key 的剩余过期时间。

> 可用版本：>= 2.6.0

### 返回值

当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。

### 示例

```bash
# key 存在，但没有设置剩余生存时间
redis> SET key value
OK

redis> PTTL key
(integer) -1

# 有剩余生存时间的 key
redis> PEXPIRE key 10086
(integer) 1

redis> PTTL key
(integer) 6179
```

---

## 12. TTL KEY_NAME

以秒为单位返回 key 的剩余过期时间。

> 可用版本：>= 1.0.0

### 返回值

当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。

### 示例

```bash
# key 存在，但没有设置剩余生存时间
redis> SET key value
OK

redis> TTL key
(integer) -1


# 有剩余生存时间的 key
redis> EXPIRE key 10086
(integer) 1

redis> TTL key
(integer) 10084
```

---

## 13. RANDOMKEY

从当前数据库中随机返回一个 key 。

> 可用版本：>= 1.0.0

### 返回值

当数据库不为空时，返回一个 key 。 当数据库为空时，返回 nil （windows 系统返回 null）。

### 示例

```bash
redis> MSET fruit "apple" drink "beer" food "cookies"   # 设置多个 key
OK

redis> RANDOMKEY
"fruit"

redis> RANDOMKEY
"food"
```

---

## 14. RENAME OLD_KEY_NAME NEW_KEY_NAME

用于修改 key 的名称 。

> 可用版本：>= 1.0.0

### 返回值

改名成功时提示 OK ，失败时候返回一个错误。

### 示例

```bash
# key 存在且 newkey 不存在
redis> SET message "hello world"
OK

redis> RENAME message greeting
OK

redis> EXISTS message               # message 不复存在
(integer) 0

redis> EXISTS greeting              # greeting 取而代之
(integer) 1
```

---

## 15. RENAMENX OLD_KEY_NAME NEW_KEY_NAME

用于在新的 key 不存在时修改 key 的名称 。

> 可用版本：>= 1.0.0

### 返回值

修改成功时，返回 1 。如果 NEW_KEY_NAME 已经存在，返回 0 。

### 示例

```bash
# newkey 不存在，改名成功
redis> SET player "MPlyaer"
OK

redis> EXISTS best_player
(integer) 0

redis> RENAMENX player best_player
(integer) 1
```

---

## 16. SCAN cursor [MATCH pattern] [COUNT count]

Scan 命令用于迭代数据库中的数据库键。

> 可用版本：>= 2.8.0

### 返回值

数组列表。

### 示例

```bash
redis 127.0.0.1:6379> scan 0   # 使用 0 作为游标，开始新的迭代
1) "17"                        # 第一次迭代时返回的游标
2)  1) "key:12"
    2) "key:8"
    3) "key:4"
    4) "key:14"
    5) "key:16"
    6) "key:17"
    7) "key:15"
    8) "key:10"
    9) "key:3"
   10) "key:7"
   11) "key:1"
redis 127.0.0.1:6379> scan 17  # 使用的是第一次迭代时返回的游标 17 开始新的迭代
1) "0"
2) 1) "key:5"
   2) "key:18"
   3) "key:0"
   4) "key:2"
   5) "key:19"
   6) "key:13"
   7) "key:6"
   8) "key:9"
   9) "key:11"
```

---

## 17. TYPE KEY_NAME

用于返回 key 所储存的值的类型。

> 可用版本：>= 1.0.0

### 返回值

返回 key 的数据类型，数据类型有：

- none (key不存在)
- string (字符串)
- list (列表)
- set (集合)
- zset (有序集)
- hash (哈希表)

### 示例

```bash
# 字符串
redis> SET weather "sunny"
OK

redis> TYPE weather
string
```

---

## 思维导图

![redis-通用命令-思维导图.png](https://cnymw.github.io/GolangStudy/docs/redis-通用命令/redis-通用命令-思维导图.png)

