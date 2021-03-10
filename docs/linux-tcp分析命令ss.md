# Linux ss 详解

运维经常使用的工具之一是 netstat，但是，netstat 命令已被弃用，取而代之的是更快，更易于阅读的 ss 命令。

ss 命令是一个用于转储套接字统计信息的工具，它以与 netstat 类似的方式（但是更简单，更快）显示信息。

ss 命令还可以显示比大多数其他工具更多的 TCP 和状态信息。

因为 ss 是新的 netstat，所以我们将研究如何使用这个工具，以便你可以更轻松的获得有关 Linux 机器以及网络连接情况的信息。

ss 命令行程序可以显示数据包，TCP，UDP，DCCP，RAW 和 Unix 域套接字等状态。

使用 ss，你可以获得有关 Linux 机器如何与其他机器，网络和服务通信的非常详细的信息（有关网络连接，网络协议统计信息和 Linux 套接字连接的详细信息）。有了这些信息，你可以更轻松的解决各种网络问题。

## 基本用法

ss 命令的工作方式与 Linux 平台上的任何命令类似：执行命令，然后使用可用选项的任意组合。

如果你浏览 ss 手册页（执行`man ss`），你会注意到 netstat 命令的选项几乎没有，但是，这并不意味着缺少功能，事实上，ss 是相当强大的。

如果执行 ss 命令，但是不加任何参数或选项，它将返回具有已建立连接的 TCP 套接字的完整列表（如下图）。

![ss](https://cnymw.github.io/GolangStudy/docs/img/linux-tcp分析命令ss-ss.png)

由于 ss 命令不带选项的话将显示大量信息（所有 tcp，udp 和 unix 套接字连接详细信息），因此你还可以将该命令输出发送到一个文件以供以后查看，如下所示：

```bash
ss > ss_output
```

如果我们只想查看当前的正在监听的 sockets，可以执行以下命令：

```bash
ss -l
```

更具体的，可以这样想：ss 可以通过使用 -t 选项查看 TCP 连接，UDP 连接通过使用 -u 选项，UNIX 连接通过使用 -x 选项查看。

```bash
ben@ben:~$ ss -u
Recv-Q Send-Q      Local Address:Port        Peer Address:Port                
0      0           10.13.83.134:45133        162.159.200.123:ntp  
```

默认情况下，单独使用 -t，-u 或 -x 选项将只列出已建立（或已连接）的连接，如果我们想获取正在监听的连接，我们必须添加 -a 选项，如：

```bash
ss -t -a
```

在上面例子中，你可以看到 UDP 连接（处于不同的状态）是 from 我的机器的 IP 地址，from 不同的端口，to 不同的 IP 地址，to 不同的端口。

与此命令的 netstat 版本不同，ss 不显示负责这些连接的 PID 和命令名。即便如此，你仍需要大量信息来进行故障排除。

如果怀疑这些端口或 url 中的任何一个，你现在就知道是哪个 IP 地址/端口建立了连接，有了这些信息，你现在可以在故障排除的早期阶段提供帮助。

## 用 ss 过滤 TCP 状态

ss 命令可用的一个非常方便的选项是能够使用 TCP 状态（连接的"生命周期"）过滤。使用状态，你可以更轻松的过滤 ss 命令结果。

ss 工具可与所有标准 TCP 状态一起使用：

```bash
established

syn-sent

syn-recv

fin-wait-1

fin-wait-2

time-wait

closed

close-wait

last-ack

listening

closing
```

其他可用的状态标识符包括：

```bash
all (上面所有的状态)

connected (除了 listen 和 closed 之外的所有状态)

synchronized (除了 syn-sent 之外的所有已连接状态)

bucket (minisockets 维护的状态，比如 time-wait 和 syn-recv)

big (和 bucket 相反的状态)
```

处理状态的语法很简单：

```bash
For tcp ipv4:
ss -4 state FILTER

For tcp ipv6:
ss -6 state FILTER
```

其中 FILTER 是要使用的状态的名称。

假设你要查看计算机上所有正在监听的 IPV4 sockets，命令为：

```bash
ss -4 state listening
```

执行的结果如下所示：

```bash
ben@ben:~$ ss -4 state listening
Netid  Recv-Q Send-Q  Local Address:Port    Peer Address:Port                
tcp    0      551     127.0.0.1:8073        *:*                    
tcp    0      551     127.0.0.1:10249       *:*                    
tcp    0      551     10.13.83.134:10250    *:*                    
tcp    0      551     127.0.0.1:10251       *:*                    
tcp    0      551     10.13.83.134:2379     *:*                    
tcp    0      551     127.0.0.1:2379        *:*                                                                  *:*     
```

## 显示从特定地址连接的 sockets

你可以给 ss 一个方便的任务是让他报告由另一个 IP 地址建立的连接。假设你想了解 IP 地址为 192.168.1.139 的计算机是否/如何连接到服务器。

你可以执行命令：

```bash
ss dst 192.168.1.139
```

结果如下所示，里面显示计算机 IP 10.226.33.103 正在 ssh 10.13.83.134 服务器：

```bash
ben@ben:~$ ss dst 10.226.33.103
Netid  State      Recv-Q Send-Q    Local Address:Port         Peer Address:Port                
tcp    ESTAB      0      272       10.13.83.134:ssh           10.226.33.103:60234
```