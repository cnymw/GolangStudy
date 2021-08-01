# redis 字符串命令

## SET

### SET key value[EX seconds][PX milliseconds][NX|XX]

> 可用版本：>=1.0.0

将字符串值`value`关联到`key`。

如果`key`已经持有其他值，`set`就覆盖旧值，无视类型。

当`set`命令对一个带有生存时间（TTL）的`key`进行设置之后，该`key`原有的 TTL 将被清除。

### 可选参数

- EX seconds：将键的过期时间设置为`seconds`秒。
- PX milliseconds：将键的过期时间设置为`milliseconds`毫秒。
- NX：只在键不存在时，才对键进行设置操作。
- XX：只在键已经存在时，才对键进行设置操作。

### 代码示例

对不存在的键进行设置：

```bash
redis> SET key "value"
OK

redis> GET key
"value"
```

使用`EX`选项：

```bash
redis> SET key-with-expire-time "hello" EX 10086
OK

redis> GET key-with-expire-time
"hello"

redis> TTL key-with-expire-time
(integer) 10069
```

使用`NX`选项：

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

## SETNX

### SETNX key value

> 可用版本：>=1.0.0

只在键`key`不存在的情况下，将键`key`的值设置为`value`。

若键`key`已经存在，则`SETNX`命令不做任何动作。

### 代码示例

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

## SETEX

### SETEX keys seconds value

> 可用版本：>=2.0.0

将键`key`的值设置为`value`，并将键`key`的生存时间设置为`seconds`秒钟。

如果键`key`已经存在，那么`SETEX`命令将覆盖已有的值。

`SETEX`和`SET`的不同之处在于`SETEX`是一个原子（atomic）操作， 它可以在同一时间内完成设置值和设置过期时间这两个操作， 因此`SETEX`命令在储存缓存的时候非常实用。

### 代码示例

键`key`已经存在， 使用`SETEX`覆盖旧值：

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

## PSETEX

> 可用版本：>=2.6.0

这个命令和`SETEX`命令相似， 但它以毫秒为单位设置`key`的生存时间， 而不是像`SETEX`命令那样以秒为单位进行设置。

### 代码示例

```bash
redis> PSETEX mykey 1000 "Hello"
OK

redis> PTTL mykey
(integer) 999

redis> GET mykey
"Hello"
```

## GET

### GET key

> 可用版本：>=1.0.0

返回与键`key`相关联的字符串值。

### 代码示例

```bash
redis> GET db
(nil)

redis> SET db redis
OK

redis> GET db
"redis"
```

## GETSET

### GETSET key value

> 可用版本：>=1.0.0

将键`key`的值设为`value`， 并返回键`key`在被设置之前的旧值。

### 代码示例

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

## STRLEN

### STRLEN key

> 可用版本：>=2.2.0

返回键`key`储存的字符串值的长度。

### 代码示例

获取字符串值的长度：

```bash
redis> SET mykey "Hello world"
OK

redis> STRLEN mykey
(integer) 11
```

## APPEND

### APPEND key value

> 可用版本：>=2.0.0

如果键`key`已经存在并且它的值是一个字符串，`APPEND`命令将把`value`追加到键`key`现有值的末尾。

如果`key`不存在，`APPEND`就简单地将键`key`的值设为`value`， 就像执行`SET key value`一样。

### 代码示例

对已存在的字符串进行`APPEND`：


```bash
redis> APPEND myphone " - 1110"     # 长度从 5 个字符增加到 12 个字符
(integer) 12

redis> GET myphone
"nokia - 1110"
```

## SETRANGE

### SETRANGE key offset value

> 可用版本：>=2.2.0

从偏移量`offset`开始， 用`value`参数覆写(overwrite)键`key`储存的字符串值。

`SETRANGE`命令会确保字符串足够长以便将`value`设置到指定的偏移量上， 如果键`key`原来储存的字符串长度比偏移量小(比如字符串只有`5`个字符长，但你设置的`offset`是`10`)， 那么原字符和偏移量之间的空白将用零字节(zerobytes, "\x00" )进行填充。

当生成一个很长的字符串时，Redis 需要分配内存空间，该操作有时候可能会造成服务器阻塞(block)。

### 代码示例

对非空字符串执行 SETRANGE 命令：

```bash
redis> SET greeting "hello world"
OK

redis> SETRANGE greeting 6 "Redis"
(integer) 11

redis> GET greeting
"hello Redis"
```

## GETRANGE 

### GETRANGE key start end

> 可用版本：>=2.4.0

返回键`key`储存的字符串值的指定部分，字符串的截取范围由`start`和`end`两个偏移量决定 (包括`start`和`end`在内)。

负数偏移量表示从字符串的末尾开始计数，`-1`表示最后一个字符，`-2`表示倒数第二个字符，以此类推。

### 代码示例

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