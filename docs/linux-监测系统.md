# linux 监测系统

## linux 监测程序

### linux 探查进程

当程序运行在系统上时，我们称之为进程（process），在 linux 中使用 ps 命令监测这些进程。

```bash
  PID TTY           TIME CMD
 3498 ttys000    0:00.05 /bin/bash --rcfile /Applications/GoLand.app/Contents/p
 3520 ttys000    0:00.73 node /usr/local/bin/docsify serve ./go-study/
 3547 ttys001    0:00.03 -bash
```

默认情况下，ps 命令只会显示运行在当前控制台下的属于当前用户的进程。

关于 ps 命令的详细参数可以查看 [ps命令](https://man.linuxde.net/ps)

常用的命令为:
```bash
BenjaminYoungdeMacBook-Pro:~ benjamin$ ps -ef | grep bash
  UID  PID    PPID  C   STIME    TTY        TIME   CMD
  501  3498   963   0 10:57上午 ttys000    0:00.05 /bin/bash --rcfile /Applications/GoLand.app/Contents/plugins/terminal/jediterm-bash.in -i
  501  3547  3546   0 10:58上午 ttys001    0:00.04 -bash
  501  3799  3547   0 11:18上午 ttys001    0:00.00 grep bash
```

这个例子用了两个参数：-e 参数指定显示所有运行在系统上的进程；-f 参数则扩展了输出，这些扩展的列包含了有用的信息。

常用字段含义如下:

- UID：启动进程的用户
- PID：进程的进程 ID
- PPID：父进程的进程号（如果该进程是由另一个进程启动的）
- C：进程生命周期中的 CPU 利用率
- STIME：进程启动时的系统时间
- TTY：进程启动时的终端设备
- TIME：运行进程时需要的累计 CPU 时间。
- CMD：启动的程序名称

### 实时监测进程

top 命令跟 ps 命令相似，能够显示进程信息，但它是实时显示的。

```bash
Processes: 328 total, 2 running, 326 sleeping, 1707 threads            21:50:09
Load Avg: 2.32, 1.99, 1.84  CPU usage: 3.29% user, 2.5% sys, 94.64% idle
SharedLibs: 246M resident, 66M data, 43M linkedit.
MemRegions: 88281 total, 6609M resident, 199M private, 2414M shared.
PhysMem: 14G used (2465M wired), 1600M unused.
VM: 1669G vsize, 1371M framework vsize, 0(0) swapins, 0(0) swapouts.
Networks: packets: 381767/142M in, 544681/85M out.
Disks: 306167/4331M read, 170260/6105M written.

PID   COMMAND      %CPU TIME     #TH    #WQ  #PORT MEM    PURG   CMPR PGRP PPID
2121  mdworker_sha 0.0  00:00.20 4      1    61    7228K  0B     0B   2121 1
2120  top          11.8 00:00.71 1/1    0    26    3680K  0B     0B   2120 2097
2097  bash         0.0  00:00.02 1      0    21    2572K  0B     0B   2097 2096

```

以下是各个参数的含义：

- Processes：显示总共的，运行中的，睡眠中的进程数，以及线程数
- Load Avg：三个值分别对应系统当前1分钟、5分钟、15分钟内的平均 load。load 用于反映当前系统的负载情况。
对于单核CPU而言，当平均负载为0时，表示CPU完全空闲，当平均负载为1时，表示CPU为满负荷状态，但两个极端都不应出现在我们的服务器上，前者说明系统没有被充分利用到，后者说明系统濒临奔溃。
关于系统负载可以查看文章：[读懂系统负载(Load Avg)的含义](https://www.cnblogs.com/softidea/p/5728956.html)
- CPU usage：三个值分别代表用户态使用的cpu时间比，系统态使用的cpu时间比，空闲的cpu时间比。关于 CPU 信息可以查看文章：[你不一定懂的cpu显示信息](https://www.cnblogs.com/yjf512/p/3383915.html)
- SharedLibs：代码和数据段的驻留大小，以及链接编辑器内存使用情况。
- MemRegions：内存区域的数量和总大小，以及划分为专用（分为非库和库）和共享组件的内存区域的总大小。
- PhysMem：物理内存使用情况，分为有线、活动、非活动、已用和空闲组件。
- VM：总虚拟内存、共享库消耗的虚拟内存以及分页和分页的数量。
- Networks：输入和输出网络数据包的数量和总大小。
- Disks：磁盘读写的数量和总大小。
- PID：进程的进程 ID
- COMMAND：启动的程序名称
- %CPU：CPU 使用率
- TIME：程序执行时间
- TH：线程数（总计/运行）
- WQ：工作队列（总计/正在运行）
- PORT：Mach 端口的数值，关于 Mac 系统的 Mach 机制可以查看该文章：[Mach消息发送机制](https://www.jianshu.com/p/a764aad31847)
- MEM：进程使用的内存大小
- PURG：清除的页数和当前可清除的页数
- CMPR：寄存器和累加器比较
- PGRP：进程 group id
- PPID：父进程 id

### 结束进程

在 Linux 中，进程之间通过信号来通信。进程的信号就是预定义好的一个消息，进程能识别它并决定忽略还是作出反应。

以下就是信号列表：

信号|名称|描述
---|:--:|--
1|HUP|挂起
2|INT|中断
3|QUIT|结束运行
9|KILL|无条件终止
11|SEGV|段错误
15|TERM|尽可能终止
17|STOP|无条件停止运行，但不终止
18|TSTP|停止或暂停，但继续在后台运行
19|CONT|在 STOP 或 TSTP 之后恢复执行

kill命令可以通过进程 ID（PID）给进程发信号，默认情况下，kill 命令会向命令行中列出的全部 PID 发送一个 TERM 信号。

killall 命令非常强大，它支持通过进程名而不是 PID 来结束进程，也支持通配符，这在系统因负载过大而变得很慢时很有用。

```bash
killall http*
```

上述命令结束了所有以 http 开头的进程。

## 监测磁盘空间

### 挂载(卸载)存储媒体

Linux 文件系统将所有的磁盘都并入一个虚拟目录下，在使用新的存储媒体之前，需要把它放到虚拟目录下，这项工作叫做挂载（mounting）

Linux 上用来挂载媒体的命令叫做 mount，mount 命令会输出当前系统上挂载的设备列表。

```bash
BenjaminYoungdeMacBook-Pro:~ benjamin$ mount
/dev/disk1s1 on / (apfs, local, journaled)
devfs on /dev (devfs, local, nobrowse)
/dev/disk1s4 on /private/var/vm (apfs, local, noexec, journaled, noatime, nobrowse)
map -hosts on /net (autofs, nosuid, automounted, nobrowse)
map auto_home on /home (autofs, automounted, nobrowse)
```

mount 命令提供如下四部分信息：

- 媒体的设备文件名
- 媒体挂载到虚拟目录的挂载点
- 文件系统类型
- 已挂载媒体的访问状态

从 Linux 系统上移除一个可移动设备时，不能直接从系统上移除，而应该先卸载，卸载设备的命令是 umount。

```bash
umount [directory | device ]
```

### 查看磁盘空间

df 命令可以让你很方便地查看所有已挂载磁盘的使用情况。

```bash
BenjaminYoungdeMacBook-Pro:~ benjamin$ df
Filesystem    512-blocks      Used Available Capacity iused               ifree %iused  Mounted on
/dev/disk1s1   489620264 286466736 199699312    59% 1893125 9223372036852882682    0%   /
devfs                663       663         0   100%    1148                   0  100%   /dev
/dev/disk1s4   489620264   2097192 199699312     2%       1 9223372036854775806    0%   /private/var/vm
map -hosts             0         0         0   100%       0                   0  100%   /net
map auto_home          0         0         0   100%       0                   0  100%   /home
```

命令输出的含义如下：

- 设备的文件位置
- 能容纳多少个 512 字节大小的块
- 已用了多少个 512 字节大小的块
- 还有多少个 512 字节大小的块可用
- 可用空间所占的比例
- 已使用的 inode 数量
- inode 空闲数量
- inode 的使用比例
- 设备挂载到了哪个挂载点上

du 命令可以显示某个特定目录（默认情况下是当前目录）的磁盘使用情况，可用来快速判断系统上某个目录下是不是有超大文件。

```bash
BenjaminYoungdeMacBook-Pro:Downloads benjamin$ du
256	./java_api_client/docs
120	./java_api_client/gradle/wrapper
120	./java_api_client/gradle
8	./java_api_client/.swagger-codegen
72	./java_api_client/.idea/libraries
136	./java_api_client/.idea
16	./java_api_client/src/test/java/cn/ben/yun/api
40	./java_api_client/src/test/java/cn/ben/yun
64	./java_api_client/src/test/java/cn/ben
88	./java_api_client/src/test/java/cn
112	./java_api_client/src/test/java
136	./java_api_client/src/test
40	./java_api_client/src/main/java/cn/ben/yun/auth
280	./java_api_client/src/main/java/cn/ben/yun/model
208	./java_api_client/src/main/java/cn/ben/yun/api
736	./java_api_client/src/main/java/cn/ben/yun
760	./java_api_client/src/main/java/cn/ben
784	./java_api_client/src/main/java/cn
808	./java_api_client/src/main/java
840	./java_api_client/src/main
1000	./java_api_client/src
1672	./java_api_client
36768	.
```

每行输出左边的数值是每个文件或目录占用的磁盘块数。这个列表是从目录层级的最底部开始，然后按文件，子目录，目录逐级向上。

下面是 du 命令常用的几个命令行参数：
- c：显示所列出文件总的大小
- h：按用户易读的格式输出大小

