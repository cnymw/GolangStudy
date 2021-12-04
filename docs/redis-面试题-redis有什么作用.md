# redis 面试题：redis 有什么作用

该面试题实际上是考察对 redis 数据结构（功能）的熟悉程度，因为 redis 的作用是取决于功能，功能是取决于各个命令的作用。

我们可以根据 redis 数据结构（功能）来依次回答：

## 字符串 string

- 当缓存层来加速读写性能降低后端的压力
- 分布式共享 session，分布式锁等分布式场景

## 哈希 hash

- 电商网站购物车设计与实现：hash 为用户 id，key 为商品 id，value 为商品数量：

```bash
# 用户 userA 购买 iphone 100 台
redis> HSET userA iphone 100
```

- 电商做活动商家设置抢购上限：

```bash
# 商家 companyB 设置 iphone11 上限 100 台，iphone12 上限 200 台，iphone13 上限 300 台
redis> HMSET companyB iphone11 100 iphone12 200 iphone 300
```

## 列表 list

- 实现任务队列，当遇到大批量但是不需要立即执行的命令，可以利用 redis 列表功能来实现异步任务队列
- 对数据量较大的集合数据进行增删改，例如关注列表，粉丝列表，留言列表等

```bash
# 在我的粉丝列表里添加A，B，C三位粉丝
redis> LPUSH myfans A B C
(integer) 3
```

## 集合 set

- 计算两个列表并集，差集，例如查询用户 A 和用户 B 之间关注列表的差异

```bash
# joe 看过了 3 部电影
redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

# peter 看过了 3 部电影
redis> SMEMBERS peter's_movies
1) "bet man"
2) "start war"
3) "2012"

# 计算出 joe 和 peter 之间关注电影的差异，并存放到差异的电影列表中去（可能 peter 和 joe 想一起看电影，又不想看都看过的）
redis> SDIFFSTORE joe_diff_peter joe's_movies peter's_movies
(integer) 2

# 得出差异电影列表
redis> SMEMBERS joe_diff_peter
1) "hi, lady"
2) "Fast Five"
```

## 有序集合 zset

在某个存在排名，数量计算的场景下：

- 获取某用户的排行
- 返回 Top N 的排行
- 获取总的参与选手数量

```bash
#给Alice投票
redis> zincrby vote_activity 1 Alice
"1" 
#给Bob投票
redis> zincrby vote_activity 1 Bob
"1"
#给Alice投票
redis> zincrby vote_activity 1 Alice
"2"
#查看Alice投票数
redis> zscore vote_activity Alice
"2"
#获取Alice排名(从高到低，zero-based)
redis> zrevrank vote_activity Alice
(integer) 0
#获取前10名(从高到低)
redis> zrevrange vote_activity 0 9
1) "Alice"
2) "Bob"
#获取前10名及对应的分数(从高到低)
redis> zrevrange vote_activity 0 9 withscores
1) "Alice"
2) "2"
3) "Bob"
4) "1"
#获取总参与选手数
redis> zcard vote_activity
(integer) 2
```

## 基数估算 HyperLogLog

- 基数统计，例如一个网站一天内存在多个用户访问流水，每个用户的访问流水都算一次访问，统计网站总的访问流水可以用到 HyperLogLog 。

```bash
# benjamin leetcode nowcoder 今天都访问了 github 网站
redis> PFADD  github  "benjamin"  "leetcode"  "nowcoder"
(integer) 1

# 今天统计结果，github 总共有三个用户访问
redis> PFCOUNT  github
(integer) 3

# benjamin 用户比较勤奋，额外多访问了一次 github
redis> PFADD  github  "benjamin"

# github 统计仍然只有三个用户访问
redis> PFCOUNT  github
(integer) 3
```

## 发布订阅 pub/sub

在了解发布订阅的场景前需要先了解它的缺点：

1. 消息无法持久化，存在丢失风险
2. 没有类似 ACK 的机制，发布方不会确保订阅方成功接受
3. 广播机制，下游消费能力取决于消费方本身

所以根据这些缺点可以有如下几种场景会用到 redis 发布订阅功能：

- 构建实时消息系统，比如普通的即使聊天，群聊等
- 订阅新闻推送

# 思维导图

![redis-面试题-redis有什么作用-思维导图.png](https://cnymw.github.io/GolangStudy/docs/img/redis-面试题-redis有什么作用-思维导图.png)
