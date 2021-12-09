# TCP-IP 详解：ICMP Internet控制报文协议

ICMP 报文通常被 IP 层或更高层协议（TCP 或 UDP）使用，它传递差错报文以及其他需要注意的信息。

## 报文格式

![ICMP 报文](https://cnymw.github.io/GolangStudy/docs/img/网络-TCP-IP详解-ICMP-报文.png)

- 8位类型字段：类型字段可以有 15 个不同的值，以描述特定类型的 ICMP 报文。
- 8位代码：某些 ICMP 报文使用代码字段的值来进一步描述不同的条件。
- 16位校验和字段：校验和计算整个 ICMP 报文，使用的算法和 IP 首部校验和算法相同。

## 报文类型

ICMP 报文类型由报文中的类型字段和代码字段共同决定。

![ICMP 报文类型](https://cnymw.github.io/GolangStudy/docs/img/网络-TCP-IP详解-ICMP-报文类型.png)

为了防止 ICMP 差错报文对广播分组响应所带来的广播风暴，下面几种情况都不会产生 ICMP 差错报文：

- ICMP差错报文，在对 ICMP 差错报文进行响应时，永远不会生成另一份 ICMP 差错报文，如果没有这个限制，可能会遇到一个差错产生另一个差错，无休止循环下去。
- 目的地址是广播地址，或多播地址的 IP 数据报
- 作为链路层广播的数据报
- 不是 IP 分片的第一片
- 源地址不是单个主机的数据报。也就是说，源地址不能为零地址，环回地址，广播地址或多播地址

# 思维导图

![思维导图](https://cnymw.github.io/GolangStudy/docs/img/网络-TCP-IP详解-ICMP-思维导图.png)
