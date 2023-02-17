# GolangStudy: 从零学习 Golang

本项目旨在指导程序员们如何从基础开始系统的学习 Go 语言，并学习开发时所需知识点。

Go 是一门非常容易上手的语言，语法简洁，代码易读，如果你有常用语言的基础，那看一本[Go 语言圣经](https://docs.hacknode.org/gopl-zh/index.html)后便可上手开发业务代码。

本项目不是博客，一个问题不会讲的特别细致，如果想要更加深入的了解某个知识点的话，建议使用搜索引擎去搜索经典博客加深理解，项目中也会推荐一些好的文章供参考。

本项目更加看重的是对于知识点系统的一个总结，能够通过一句话，一张思维导图来把一类知识进行讲解，这样在面试前把所有的思维导图都过一遍就快的吸收知识点，平时空余的时候拿出思维导图进行复习也能加深理解。

作者通过该学习笔记，拿到过腾讯云，金山办公，青藤云，神州数码等云相关企业 offer，工作岗位是 Golang 服务端开发。

## 在线课程

本项目准备将学习内容转化为在线学习的方式，利用视频&&文字&&思维导图的方式提高大家的学习效率，具体的效果可以看以下链接：

[在线课程](https://golangstudy.tech/)

![在线课程](https://cnymw.github.io/GolangStudy/docs/img/在线课程.png)


## 如何利用思维导图学习

对于一个新的知识点，我一般是这样结合思维导图来学习的：

1. 找到一个比较详细的资料，例如博客，书籍等，如果在 baidu 上面搜不到易读的内容（可能性较大），可以尝试用 bing 看看英文的资料，能够翻墙的可以使用 google。
2. 通读全文，将每一个陌生的专业词汇记录下来，用于更进一步的学习，逐步发散，逐渐扩充自己的知识库。
3. 捋清文章脉络，找到每个章节的核心概要，将概要记录到思维导图子标题上。
4. 遍历每个概要，逐步下沉概要里的各个关键知识点，记录到思维导图。
5. 比对文章内容和思维导图，确认没有遗漏的知识点。
6. 经常回顾记录的思维导图，用于加深知识点的记忆。
7. 用持续进步的角度审视所记录的思维导图，思维导图不是标准答案，只是你对一个知识点的总结，不一定是绝对正确的，可以持续优化，持续总结出更好的内容。
8. 思维是抽象的，网状的，不是线性的，很多较难的知识点如果用一句线性的语句来描述会非常的难以理解，所以要将重难点内容解析成思维导图，要习惯这种思维模式，这样才算真正的理解了思维导图的作用。

## Golang 学习目录

- Golang 学习路线
  1. [Golang 语言](https://github.com/cnymw/GolangStudy#golang语言)
  2. [Kubernetes](https://github.com/cnymw/GolangStudy#kubernetes)
  3. [docker](https://github.com/cnymw/GolangStudy#docker)
  4. [linux](https://github.com/cnymw/GolangStudy#linux)
  5. 网络
  6. 设计模式
  7. redis
  8. etcd
  9. kafka
  10. Elasticsearch
  11. 算法/数据结构
  12. Leetcode
  13. mysql
  14. 面试题

## Golang语言

Golang 语言的学习可以分为以下方向：

- 基础：开发核心能力
- 高级机制：线程调度，垃圾回收等，进阶的面试经常会考
- 源码解读：对 Golang 加深理解，进阶开发有帮助
- 常用框架：对生产开发有帮助，可以简单了解下，在技术选型的时候可以快速做出判断

学习目录如下：

- 基础
  - [Go 基础语法](/docs/go-基础语法/go-基础语法.md)
  - [Go 数据类型](/docs/go-数据类型/go-数据类型.md)
  - [Go 并发](/docs/go-并发.md)
  - [Go 接口](/docs/go-接口.md)
  - [Go 检测竞态条件](/docs/go-检测竞态条件.md)
- 进阶
  - [Go 调度](/docs/go-调度.md)
  - [Go 垃圾回收](/docs/go-垃圾回收.md)
  - [Go channel](/docs/go-channel/go-channel.md)
  - [Go 性能分析：pprof实战](https://blog.wolfogre.com/posts/go-ppof-practice/)
- 深入 golang sdk 源码
  - [Go 源码解读 标识符](/docs/go-源码解读-标识符/go-源码解读-标识符.md)
  - [Go 源码解读 双向链表 list](/docs/go-源码解读-双向链表list/go-源码解读-双向链表list.md)
  - [Go 源码解读 程序分析pprof](/docs/go-源码解读-程序分析pprof.md)
  - [Go 源码解读 同步模块sync mutex](/docs/go-源码解读-同步sync-mutex.md)
  - [Go 源码解读 同步模块sync once](/docs/go-源码解读-同步sync-once.md)
  - [Go 源码解读 垃圾回收](/docs/go-源码解读-垃圾回收/go-源码解读-垃圾回收.md)
  - [Go 源码解读 channel](/docs/go-源码解读-channel/go-源码解读-channel.md)
- 拓展 zap
  - [Go 源码解读 高性能日志库 zap](/docs/go-zap.md)
- 拓展 grpool
  - [Go 源码解读 轻量级线程池 grpool](/docs/go-grpool.md)
- 面试题
  - [Go 有几种连接字符串的方法?](/docs/go-面试题-连接字符串方法/go-面试题-连接字符串方法.md)
  
---

## Kubernetes

Golang 常用在微服务，分布式场景，不经常用于较大的业务场景中。

常见的如开发一个微服务，部署到阿里/腾讯/华为云中，或者云原生自建的 Kubernetes 里，这个时候就需要学习并精通 Kubernetes 相关的知识。

很多企业在自建云，或者做云原生的转型，部署服务会逐渐的使用 Kubernetes，而放弃之前的物理机，虚拟机部署模式。所以面试的时候问到 Kubernetes 是比较常见的。

学习目录如下：

- [kubernetes.io 官方文档](https://kubernetes.io/zh-cn/)
  - [Kubernetes 概述｜视频学习｜完整版](/docs/Kubernetes-概述/Kubernetes-概述.md)
  - [Kubernetes 组件｜视频学习｜完整版](/docs/Kubernetes-组件/Kubernetes-组件.md)
  - [Kubernetes 理解对象｜视频学习｜完整版](/docs/Kubernetes-理解对象/Kubernetes-理解对象.md)
  - [Kubernetes 标签｜视频学习｜完整版](/docs/Kubernetes-标签/Kubernetes-标签.md)
  - [Kubernetes namespace｜视频学习｜完整版](/docs/Kubernetes-namespace/Kubernetes-namespace.md)
  - [Kubernetes 节点｜视频学习｜完整版](/docs/Kubernetes-节点/Kubernetes-节点.md)
  - [Kubernetes 工作负载](/docs/Kubernetes-工作负载/Kubernetes-工作负载.md)
  - [Kubernetes 网络模型](/docs/Kubernetes-网络模型/Kubernetes-网络模型.md)
- [Kubernetes 基本概念和术语](/docs/Kubernetes-基本概念和术语.md)
- [Kubernetes kubectl命令](/docs/Kubernetes-kubectl命令.md)

---

## Docker
- [Docker — 从入门到实践](https://vuepress.mirror.docker-practice.com)
- [Docker 基础](/docs/docker-docker基础.md)

---

## linux

- [Linux 命令大全](https://man.linuxde.net)
- [Linux 教程](https://www.runoob.com/linux/linux-tutorial.html)
- [Linux 文件操作](/docs/linux-文件操作.md)
- [Linux inode详解](https://www.cnblogs.com/llife/p/11470668.html)
- [Linux 监测系统](/docs/linux-监测系统.md)
- [Linux 抓包工具tcpdump详解](/docs/linux-抓包命令tcpdump.md)
- [Linux tcpdump命令详解(转)](https://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html)
- [Linux tcp分析命令ss详解](/docs/linux-tcp分析命令ss.md)
- [Linux curl命令详解](/docs/linux-curl命令.md)
- [Linux namespaces 命名空间](/docs/linux-namespaces.md)
- 面试题
  - [进程和线程，协程的区别](/docs/操作系统-面试题-进程线程协程的区别.md)
- 专业博客
  - [redhat sysadmin](https://www.redhat.com/sysadmin/)

---

## 数据结构

- [线性表](/docs/数据结构-线性表.md)
    - [栈](/docs/数据结构-栈.md)
    - [顺序表](/docs/数据结构-顺序表.md)
    - [链表](/docs/数据结构-链表/数据结构-链表.md)
    - [队列](/docs/数据结构-队列.md)
- 树
    - [二叉树](/docs/数据结构-二叉树.md)
    - [二叉搜索树(TODO)](/docs/数据结构-二叉搜索树.md)
- [哈希表](/docs/数据结构-哈希表.md)
- 面试题
  - [B树和B+树之间的区别](/docs/数据结构-面试题-B树和B+树的区别.md)

---

## 算法

- [排序算法总结](/docs/算法-排序算法/算法-排序算法.md)
    - [插入排序](/docs/算法-插入排序.md)
    - [堆排序](/docs/算法-堆排序.md)
    - [归并排序](/docs/算法-归并排序.md)
    - [快速排序](/docs/算法-快速排序.md)
- [动态规划](/docs/算法-动态规划.md)
- [前中后缀表达式](/docs/算法-前中后缀表达式.md)
- [滑动窗口](/docs/算法-滑动窗口.md)
- [二分查找](/docs/算法-二分查找.md)
- [分布式id生成算法：雪花算法](/docs/算法-雪花算法.md)

---

## 数据库

- [mysql InnoDB 体系结构](/docs/数据库-mysql-InnoDB体系结构.md)
  - [mysql InnoDB 内存结构](/docs/数据库-mysql-InnoDB内存结构.md)
  - [mysql InnoDB 磁盘结构](/docs/数据库-mysql-InnoDB磁盘结构.md)
- [mysql InnoDB 主从复制](/docs/数据库-mysql-InnoDB主从复制.md)
- mysql InnoDB 索引
  - [mysql InnoDB 聚集索引和辅助索引](/docs/数据库-mysql-InnoDB聚集索引和辅助索引.md)
- InnoDB 锁和事务模型
  - [mysql InnoDB 多版本控制](/docs/数据库-mysql-InnoDB多版本控制.md)
  - [mysql InnoDB 锁机制](/docs/数据库-mysql-Innodb锁机制.md)
  - [mysql InnoDB 事务隔离级别](/docs/数据库-mysql-InnoDB事务隔离级别.md)
  - [mysql InnoDB 锁定读](/docs/数据库-mysql-InnoDB锁定读.md)
  - [mysql InnoDB 死锁](/docs/数据库-mysql-死锁.md)  
  - [mysql InnoDB 死锁案例](https://github.com/aneasystone/mysql-deadlocks)
- Mysql常用工具
  - [mysql 慢查询分析工具 mysqldumpslow](/docs/数据库-mysql-慢查询分析工具mysqldumpslow.md)
  - [mysql 压测工具 mysqlslap](/docs/数据库-mysql-压测工具mysqlslap.md)

---

## 设计模式

- [策略模式](/docs/设计模式-策略模式.md)
- [观察者模式](/docs/设计模式-观察者模式.md)
- [装饰器模式](/docs/设计模式-装饰器模式.md)

---

## redis

- [redis 持久化](/docs/redis-持久化/redis-持久化.md)
- [redis sentinel](/docs/redis-sentinel/redis-sentinel.md)
- [redis 集群](/docs/redis-集群/redis-集群.md)
- [redis 实现分布式锁](/docs/redis-实现分布式锁/redis-实现分布式锁.md)
- 面试题
  - [redis 有什么作用](/docs/redis-面试题-redis有什么作用.md)
  - [redis 是单线程但为什么执行速度这么快](/docs/redis-面试题-redis为什么这么快/redis-面试题-redis为什么这么快.md)

---

## elasticsearch

- [Elasticsearch](/docs/Elasticsearch-概览.md)
- [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

---

## 消息中间件

- [kafka 介绍](/docs/消息中间件-kafka-介绍.md)
- 面试题
  - [kafka 为什么会丢消息?](/docs/消息中间件-面试题-kafka为什么丢消息.md)

--- 

## 网络

- [TCP-IP 详解：链路层](/docs/网络-TCP-IP详解-链路层.md)
- [TCP-IP 详解：IP 网际协议](/docs/网络-TCP-IP详解-IP.md)
- [TCP-IP 详解：ARP 地址解析协议](/docs/网络-TCP-IP详解-ARP.md)
- [TCP-IP 详解：RARP 逆地址解析协议](/docs/网络-TCP-IP详解-RARP.md)
- [TCP-IP 详解：ICMP Internet控制报文协议](/docs/网络-TCP-IP详解-ICMP.md)
- [TCP-IP 详解：TCP 传输控制协议](/docs/网络-TCP-IP详解-TCP传输控制协议.md)
- [grpc：grpc简介](/docs/网络-grpc简介.md)
- 面试题
  - [HTTP 1.0、1.1、2.0、3.0 区别](/docs/网络-http-http版本区别.md)
  
---

## 分布式

- [分布式：分布式架构](/docs/分布式-分布式架构.md)
- [分布式：一致性协议](/docs/分布式-一致性协议.md)
- [分布式：断路器模式](/docs/分布式-断路器模式.md)
- [分布式：etcdctl 基本使用](/docs/分布式-etcd-etcdctl基本使用.md)
- [分布式：理解 etcd](https://zhuanlan.zhihu.com/p/96721097)
- [分布式：service mesh 入门](/docs/分布式-servicemesh-servicemesh入门.md)

---

## leetcode

- 链表
  - [leetcode-2-两数相加](/docs/leetcode-2-两数相加/leetcode-2-两数相加.md)
  - [leetcode-19-删除链表的倒数第 N 个结点](/docs/leetcode-19-删除链表的倒数第N个结点/leetcode-19-删除链表的倒数第N个结点.md)
  - [leetcode-21-合并两个有序链表](/docs/leetcode-21-合并两个有序链表.md)
  - [leetcode-61-旋转链表](/docs/leetcode-61-旋转链表.md)
  - [leetcode-92-反转链表II](/docs/leetcode-92-反转链表II/leetcode-92-反转链表II.md)
- 哈希表
  - [leetcode-1-两数之和](/docs/leetcode-1-两数之和/leetcode-1-两数之和.md)
  - [leetcode-3-无重复字符的最长子串](/docs/leetcode-3-无重复字符的最长子串.md)
  - [leetcode-30-串联所有单词的子串](/docs/leetcode-30-串联所有单词的子串.md)
  - [leetcode-242-有效的字母异位词](/docs/leetcode-242-有效的字母异位词.md)
  - [leetcode-349-两个数组的交集](/docs/leetcode-349-两个数组的交集.md)
  - [leetcode-350-两个数组的交集2](/docs/leetcode-350-两个数组的交集2.md)
- 动态规划
  - [leetcode-5-最长回文子串](/docs/leetcode-5-最长回文子串.md)
  - [leetcode-53-最大子序和](/docs/leetcode-53-最大子序和.md)
  - [leetcode-64-最小路径和](/docs/leetcode-64-最小路径和.md)
  - [leetcode-70-爬楼梯](/docs/leetcode-70-爬楼梯.md)
  - [leetcode-115-不同的子序列](/docs/leetcode-115-不同的子序列.md)
- 树
  - [leetcode-100-相同的树](/docs/leetcode-100-相同的树.md)
  - [leetcode-1038-从二叉搜索树到更大和树](/docs/leetcode-1038-从二叉搜索树到更大和树.md)
- 栈
  - [leetcode-150-逆波兰表达式求值](/docs/leetcode-150-逆波兰表达式求值.md)
  - [leetcode-224-基本计算器](/docs/leetcode-224-基本计算器.md)
  - [leetcode-225-用队列实现栈](/docs/leetcode-225-用队列实现栈.md)
- [leetcode-15-三数之和](/docs/leetcode-15-三数之和.md)
- [leetcode-27-移除元素](/docs/leetcode-27-移除元素.md)
- [leetcode-56-合并区间](/docs/leetcode-56-合并区间.md)
- [leetcode-164-最大间距](/docs/leetcode-164-最大间距.md)
- [leetcode-922-按奇偶排序数组2](/docs/leetcode-922-按奇偶排序数组2.md)
- [leetcode-976-三角形的最大周长](/docs/leetcode-976-三角形的最大周长.md)

## 面试题库

- [Golang面试题](https://kdocs.cn/l/ckJHEO8q0oSh)
- [Kubernetes面试题](https://kdocs.cn/l/cgIwelr0RVaj)
- [操作系统面试题](https://kdocs.cn/l/ciLsKpZYE4fC)
- [算法面试题](https://kdocs.cn/l/cpOknBhkxqLx)
- [网络面试题](https://kdocs.cn/l/crwWVLgkX1zC)
- [数据库面试题](https://kdocs.cn/l/chPMUMaPoxgj)

---

# 参考资料
- [麻省理工学院公开课：算法导论](http://open.163.com/special/opencourse/algorithms.html)
- [LeetCode All in One 题目讲解汇总](https://github.com/grandyang/leetcode)
- [kafka教程](https://www.orchome.com/kafka/index)
