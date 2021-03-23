# GolangStudy: Golang 面试学习

[![在线阅读](https://badgen.net/badge/page/%E5%9C%A8%E7%BA%BF%E9%98%85%E8%AF%BB?icon=github&label)](https://cnymw.github.io/GolangStudy)
[![相关代码](https://badgen.net/badge/icon/%E7%9B%B8%E5%85%B3%E4%BB%A3%E7%A0%81?icon=github&label)](https://github.com/cnymw/learnGo)
[![贡献者](https://badgen.net/github/contributors/cnymw/GolangStudy)](https://github.com/cnymw/GolangStudy/graphs/contributors)

本项目旨在指导程序员们如何从基础开始系统的学习 Go 语言，并学习面试所需知识点。

Go 是一门非常容易上手的语言，语法简洁，代码易读，如果你有常用语言的基础，那看一本[Go 语言圣经](https://docs.hacknode.org/gopl-zh/index.html)后便可上手开发业务代码。

但是，想通过 Go 语言的面试可能需要更加系统，全面的知识，本项目通过知识点的总结来提升面试通过的几率。

本项目不是博客，一个问题不会讲的特别细致，如果想要更加深入的了解某个知识点的话，建议使用搜索引擎去搜索经典博客加深理解，项目中也会推荐一些好的文章供参考。

本项目更加看重的是对于知识点系统的一个总结，能够通过一句话，一张思维导图来把一类知识进行讲解，这样在面试前把所有的思维导图都过一遍就能更大的提升面试通过的几率了，平时空余的时候拿出思维导图进行复习也能加深理解。

## Golang 小程序在线学习

为了更方便的学习 Golang，特意开发了一款小程序，将学习内容同步到小程序中，希望能够通过手机学习的同学们可以扫一下二维码关注下小程序：

<img src="https://cnymw.github.io/GolangStudy/docs/img/首页-小程序.jpg" width="20%"/>

## go

- [Go 基础](/docs/go-基础.md)
- [Go 并发](/docs/go-并发.md)
- [Go 源码 SDK 目录](https://cloud.tencent.com/developer/doc/1101)
- [Go 性能分析：pprof实战](https://blog.wolfogre.com/posts/go-ppof-practice/)
- 源码解读
  - [Go 源码解读 标识符](/docs/go-源码解读-标识符.md)
  - [Go 源码解读 程序分析pprof](/docs/go-源码解读-程序分析pprof.md)
  - [Go 源码解读 同步模块sync mutex](/docs/go-源码解读-同步sync-mutex.md)
  - [Go 源码解读 同步模块sync once](/docs/go-源码解读-同步sync-once.md)
  - [Go 源码解读 高性能日志库 zap](/docs/go-zap.md)
  

## 数据结构

- [线性表](/docs/数据结构-线性表.md)
    - [栈](/docs/数据结构-栈.md)
    - [顺序表](/docs/数据结构-顺序表.md)
    - [链表](/docs/数据结构-链表.md)
    - [队列](/docs/数据结构-队列.md)
- 树
    - [二叉树](/docs/数据结构-二叉树.md)
    - [二叉搜索树](/docs/数据结构-二叉搜索树.md)
    - [红黑树](/docs/数据结构-红黑树.md)
    - [AVL 树](/docs/数据结构-AVL树.md) 
- [哈希表](/docs/数据结构-哈希表.md)

## 算法

- [排序算法总结](/docs/算法-排序算法.md)
    - [插入排序](/docs/算法-插入排序.md)
    - [堆排序](/docs/算法-堆排序.md)
    - [归并排序](/docs/算法-归并排序.md)
    - [快速排序](/docs/算法-快速排序.md)
- [动态规划](/docs/算法-动态规划.md)
- [前中后缀表达式](/docs/算法-前中后缀表达式.md)

## 数据库

- [mysql InnoDB 存储引擎](/docs/数据库-InnoDB存储引擎.md)
- [mysql InnoDB 锁机制](/docs/数据库-mysql-innodb锁机制.md)
- [mysql InnoDB 死锁案例](https://github.com/aneasystone/mysql-deadlocks)
- [mysql 慢查询分析工具 mysqldumpslow](/docs/数据库-mysql-慢查询分析工具mysqldumpslow.md)


## 设计模式

- [策略模式](/docs/设计模式-策略模式.md)
- [观察者模式](/docs/设计模式-观察者模式.md)
- [装饰器模式](/docs/设计模式-装饰器模式.md)

## OAuth 2.0

- [OAuth2 规范-rfc6749](/docs/oauth2-rfc6749.md)
- [OAuth2 规范中文翻译](https://github.com/jeansfish/RFC6749.zh-cn/blob/master/SUMMARY.md)

## docker

- [Docker — 从入门到实践](https://vuepress.mirror.docker-practice.com)
- [docker 基础](/docs/docker-docker基础.md)

## 操作系统

- [Linux 命令大全](https://man.linuxde.net)
- [Linux 教程](https://www.runoob.com/linux/linux-tutorial.html)
- [Linux 文件操作](/docs/linux-文件操作.md)
- [Linux inode详解](https://www.cnblogs.com/llife/p/11470668.html)
- [Linux 监测系统](/docs/linux-监测系统.md)
- [Linux 抓包工具tcpdump详解](/docs/linux-抓包命令tcpdump.md)
- [Linux tcpdump命令详解(转)](https://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html)
- [linux tcp分析命令ss详解](/docs/linux-tcp分析命令ss.md)

## 网络

- [TCP-IP 详解：链路层](/docs/网络-TCP-IP详解-链路层.md)
- [TCP-IP 详解：IP 网际协议](/docs/网络-TCP-IP详解-IP.md)
- [TCP-IP 详解：ARP 地址解析协议](/docs/网络-TCP-IP详解-ARP.md)
- [TCP-IP 详解：RARP 逆地址解析协议](/docs/网络-TCP-IP详解-RARP.md)

## leetcode

- [leetcode-1-两数之和](/docs/leetcode-1-两数之和.md)
- [leetcode-3-无重复字符的最长子串](/docs/leetcode-3-无重复字符的最长子串.md)
- [leetcode-5-最长回文子串](/docs/leetcode-5-最长回文子串.md)
- [leetcode-15-三数之和](/docs/leetcode-15-三数之和.md)
- [leetcode-21-合并两个有序链表](/docs/leetcode-21-合并两个有序链表.md)
- [leetcode-27-移除元素](/docs/leetcode-27-移除元素.md)
- [leetcode-30-串联所有单词的子串](/docs/leetcode-30-串联所有单词的子串.md)
- [leetcode-53-最大子序和](/docs/leetcode-53-最大子序和.md)
- [leetcode-56-合并区间](/docs/leetcode-56-合并区间.md)
- [leetcode-64-最小路径和](/docs/leetcode-64-最小路径和.md)
- [leetcode-70-爬楼梯](/docs/leetcode-70-爬楼梯.md)
- [leetcode-100-相同的树](/docs/leetcode-100-相同的树.md)
- [leetcode-115-不同的子序列](/docs/leetcode-115-不同的子序列.md)
- [leetcode-150-逆波兰表达式求值](/docs/leetcode-150-逆波兰表达式求值.md)
- [leetcode-164-最大间距](/docs/leetcode-164-最大间距.md)
- [leetcode-224-基本计算器](/docs/leetcode-224-基本计算器.md)
- [leetcode-225-用队列实现栈](/docs/leetcode-225-用队列实现栈.md)
- [leetcode-242-有效的字母异位词](/docs/leetcode-242-有效的字母异位词.md)
- [leetcode-349-两个数组的交集](/docs/leetcode-349-两个数组的交集.md)
- [leetcode-350-两个数组的交集2](/docs/leetcode-350-两个数组的交集2.md)
- [leetcode-922-按奇偶排序数组2](/docs/leetcode-922-按奇偶排序数组2.md)
- [leetcode-976-三角形的最大周长](/docs/leetcode-976-三角形的最大周长.md)
- [leetcode-1038-从二叉搜索树到更大和树](/docs/leetcode-1038-从二叉搜索树到更大和树.md)

# 参考资料
- [麻省理工学院公开课：算法导论](http://open.163.com/special/opencourse/algorithms.html)
- [LeetCode All in One 题目讲解汇总](https://github.com/grandyang/leetcode)

# 公众号

<img src="https://cnymw.github.io/GolangStudy/docs/img/首页-wechat.jpg" width="20%"/>

# 联系作者

欢迎大家指出不足，如有任何疑问，请邮件联系 benjaminymw at foxmail dot com 或者直接修复并提交 Pull Request。