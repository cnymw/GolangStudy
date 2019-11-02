# 一，redis 简介

- redis 就是一个数据库
- redis 的数据是存在内存中的，所以读写速度非常快
- redis 被广泛应用于缓存方向。
- redis 也经常用来做分布式锁
- redis 提供了多种数据类型来支持不同的业务场景。
- redis 支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。

# 二，基本概念

## 2.1为什么要用 redis/为什么要用缓存
###2.1.1高性能

![redis高性能](/docs/img/redis基础/redis-tree.png)

###2.1.2高并发