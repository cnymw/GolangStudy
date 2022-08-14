# Kubernetes 基本概念和术语

## Master

Kubernetes 里的 Master 指的是集群控制节点，每个 Kubernetes 集群里需要有一个 Master 节点来负责整个集群的管理和控制，基本上 Kubernetes 所有的控制命令都发送给它，如果 Master
宕机或不可用，那么对集群内容器应用的管理都将失效。

Master 节点上运行着以下关键进程：

- Kubernetes API Server（kube-apiserver）：提供了 HTTP Rest 接口的关键服务进程，是 Kubernetes 里所有资源的增删改查操作的唯一入口，也是集群控制的入口进程。

- Kubernetes Controller Manager（kube-controller-manager）：Kubernetes 里所有资源对象的自动化控制中心。

- Kubernetes Scheduler（kube-scheduler）：负责资源调度（Pod 调度）的进程。

在 Master 节点上还需要启动一个 etcd 服务，Kubernetes 里的所有资源对象的数据全部是保存在 etcd 中。

## Node

Kubernetes 集群中的其他机器被称为 Node 节点，是集群中的工作负载节点，每个 Node 都会被 Master 分配一些工作负载（Docker 容器），当某个 Node 宕机时，其上的工作负载会被 Master
自动转移到其他节点上去。

每个 Node 节点上都运行以下一组关键进程：

- kubelet：负责 Pod 对应的容器的创建，启停等任务，同时与 Master 节点密切协作，实现集群管理的基本功能。

- kube-proxy：实现 Kubernetes Service 的通信与负载均衡机制的重要组件。

- Docker Engine（docker）：Docker 引擎，负责本机的容器创建和管理工作。

## Pod

Pod 是 Kubernetes 最重要也最基本的概念。

Pod 由如下组成：

- 每个 Pod 都有一个特殊的被称为根容器的 Pause 容器。

- 包含一个或多个紧密相关的用户业务容器。

![Kubernetes-基本概念和术语-Pod的组成.png](https://cnymw.github.io/GolangStudy/docs/img/Kubernetes-基本概念和术语-Pod的组成.png)

### Pod 的设计理念

为什么 Kubernetes 会设计出一个全新的 Pod 概念并且有特殊的组成结构：

1. 在一组容器作为一个单元的情况下，难以对"整体"简单地进行判断及有效地进行行动。引入业务无关且不易死亡的 Pause 容器作为 Pod 的根容器，以它的状态代表整个容器组的状态，就解决了这个问题。

2. Pod 里的多个业务容器共享 Pause 容器的 IP，共享 Pause 容器挂载的 Volume，既简化了密切关联的业务容器之间的通信问题，也解决了它们之间的文件共享问题。

### Pod 配额限定

在 Kubernetes 里，一个计算资源进行配额限定需要设定以下两个参数：

- Requests：该资源的最小申请量，系统必须满足要求。

- Limits：该资源最大允许使用的量，不能被突破，当容器试图使用超过这个量的资源时，可能会被 Kubernetes Kill 并重启。

## Label（标签）

Label 是一个 key=value 的键值对，其中 key 与 value 由用户自己指定。

Label 可以附加到各种资源对象上，例如 Node，Pod，Service，RC 等，一个资源对象可以定义任意数量的 Label，同一个 Label 也可以被添加到任意数量的资源对象上去，Label
通常在资源对象定义时确定，也可以在对象创建后动态添加或删除。

打了 Label（标签）后，可以通过 Label Selector（标签选择器）查询和筛选拥有某些 Label 的资源对象，Kubernetes 通过这种方式实现了类似 SQL 的简单又通用的对象查询机制。

有两种 Label Selector 的表达式：基于等式和基于集合的。

- name = value

- name != value

- name in (value1,value2)

- name not in (value1,value2)

## Replication Controller

RC 是 Kubernetes 系统中的核心概念之一，它定义了一个期望的场景，声明某种 Pod 的副本数量在任意时刻都符合某个预期值，包括如下几个部分：

- Pod 期待的副本数（replicas）

- 用于筛选目标 Pod 的 Label Selector

- 当 Pod 的副本数量小于预期数量时，用于创建新 Pod 的 Pod 模板（template）

在 Kubernetes v1.2 时，它升级成了另外一个新的概念：Replica Set，它与当前 RC 存在的唯一区别是，Replica Set 支持基于集合的 Label Selector，而 RC 只支持基于等式的 Label
Selector，这使得 Replica Set 的功能更强。

总结 RC（Replica Set）的特性与作用：

- 在大多数情况下，我们通过定义一个 RC 实现 Pod 的创建过程及副本数量的自动控制。

- RC 里包括完整的 Pod 定义模板。

- RC 通过 Label Selector 机制实现对 Pod 副本的自动控制。

- 通过改变 RC 里的 Pod 副本数 ，可以实现 Pod 的扩容或缩容功能。

- 通过改变 RC Pod 模板中的镜像版本，可以实现 Pod 的滚动升级功能。

## Deployment

Deployment 是 Kubernetes v1.2 引入的新概念，目的是为了更好地解决 Pod 的编排问题。为此，Deployment 在内部使用了 Replica Set 来实现目的。

Deployment 相对于 RC 的最大升级是可以随时知道当前 Pod 的部署进度。

Deployment 的使用场景有如下几个：

- 创建一个 Deployment 对象来生成对应的 Replica Set 井完成 Pod 副本的创建过程

- 检查 Deployment 的状态来看部署动作是否完成（Pod 副本的数量是否达到预期的值）

- 更新 Deployment 以创建新的 Pod（比如镜像升级）

- 如果当前 Deployment 不稳定，则回滚到一个早先的版本

- 暂停 Deployment 以便于一次性修改多个 PodTemplateSpec 的配置项，之后再恢复 Deployment，进行新的发布

- 扩展 Deployment 以应对高负载

- 查看 Deployment 的状态，以此作为发布是否成功的指标

- 清理不再需要的旧版本 ReplicaSet

## Horizontal Pod Autoscaler

Horizontal Pod Autoscaler 中文名为 Pod 横向自动扩容，简称 HPA。

HPA 也属于一种 Kubernetes 资源对象，实现原理是通过追踪分析 RC 控制的所有目标 Pod 的负载变化情况，来确定是否需要针对性的调整目标 Pod 的副本数。

有以下两种方式作为 Pod 负载的度量指标：

- CPU 占用率

- 应用程序自定义的度量指标，比如服务在每秒内的相应的请求数（TPS 或 QPS）

## StatefulSet

在 Kubernetes 中，Pod 的管理对象 RC，Deployment，DaemonSet 和 Job 都是面向无状态的服务，但现实中有很多服务是有状态的，特别是一些复杂的中间件集群，这些集群有以下一些共同点：

- 每个节点都有固定的身份 ID，通过这个 ID，集群中的成员可以相互发现并且通信。

- 集群的规模是比较固定的，集群规模不能随意变动

- 集群里的每个节点都是有状态的，通常会持久化数据到永久存储中

- 如果磁盘损坏，则集群里的某个节点无法正常运行，集群功能受损

为了解决以上场景问题，Kubernetes 引入了 StatefulSet ，StatefulSet 本质上是 Deployment/RC 的一个特殊变种，它有如下特性：

- StatefulSet 里的每个 Pod 都有稳定，唯一的网络标识，可以用来发现集群内的其他成员。假设 StatefulSet 的名字叫 kafka，那么第一个 Pod 叫 kafka-0，第二个叫 kafka-1，以此类推。

- StatefulSet 控制的 Pod 副本的启停顺序是受控的，操作第 n 个 Pod 时，前 n-1 个 Pod 已经是运行且准备好的状态。

- StatefulSet 里的 Pod 采用稳定的持久化存储卷，通过 PV/PVC 来实现，删除 Pod 时默认不会删除与 StatefulSet 相关的存储卷（为了保证数据的安全）。

## Service（服务）

### 概述

Service 也是 Kubernetes 里最核心的资源对象之一，Kubernetes 里的每个 Service 就是微服务架构中的"微服务"。

![Kubernetes-基本概念和术语-Service.png](https://cnymw.github.io/GolangStudy/docs/img/Kubernetes-基本概念和术语-Service.png)

Kubernetes 的 Service 定义了一个服务的访问入口地址，通过这个入口地址访问其背后的一组由 Pod 副本组成的集群实例，Service 与其后端 Pod 副本集群之间是通过 Label Selector 来实现对接的。RC
的作用是保证 Service 的服务能力和服务质量始终处于预期的标准。

运行在每个 Node 上的 kube-proxy 进程是一个智能的软件负载均衡器，它负责把对 Service 的请求转发到后端的某个 Pod 实例上，并在内部实现服务的负载均衡与会话保持机制。

但 Service 不是共用一个负载均衡的 IP 地址，而是每个 Service 分配了一个全局唯一的虚拟 IP 地址，这个虚拟 IP 被称为 ClusterIP。这样每个服务就变成了具备唯一 IP 地址的节点，服务调用就变成了最基础的
TCP 网络通信问题。

Service 一旦被创建，Kubernetes 就会自动为它分配一个可用的 Cluster IP，在 Service 的整个生命周期内，它的 Cluster IP 不会发生改变，于是服务发现问题可以解决：只要用 Service 的
Name 与 Service 的 Cluster IP 地址做一个 DNS 域名映射即可解决。

### Kubernetes 的服务发现机制

Kubernetes 最早采用了 Linux 环境变量的方式来通过 Service 的名字找到对应的 Cluster IP，即每个 Service 生成一些对应的 Linux 环境变量（Env），并在每个 Pod
容器在启动时，自动注入这些环境变量。

考虑到环境变量的方式获取 Service 的 IP 与端口的方式仍然不方便，后来 Kubernetes 通过 Add-On 增值包的方式引入了 DNS 系统，把服务名作为 DNS 域名。

### 外部系统访问 Service 的问题

Kubernetes 里存在三种 IP：

- Node IP：Node 节点的 IP 地址，是 Kubernetes 集群中每个节点的物理网卡的 IP 地址，所有属于这个网络的服务器之间都能通过这个网络直接通信。Kubernetes 集群之外的节点访问 Kubernetes
  集群之内的某个节点或者 TCP/IP 服务时，必须要通过 Node IP 进行通信。

- Pod IP：Pod 的 IP 地址，它是 Docker Engine 根据 docker0 网桥的 IP 地址段进行分配的，通常是一个虚拟的二层网络。Kubernetes 要求位于不同的 Node 上的 Pod 能够彼此直接通信，所以
  Pod 里的容器访问另外一个 Pod 里的容器，就是通过 Pod IP 所在的虚拟二层网络进行通信的，而真实的 TCP/IP 流量是通过 Node IP 所在的物理网卡流出的。

- Cluster IP：Service 的 IP 地址，它是一个虚拟的 IP，原因有如下几点：
    - Cluster IP 仅仅作用于 Kubernetes Service 这个对象，并由 Kubernetes 管理和分配 IP 地址（来源于 Cluster IP 地址池）
    - Cluster IP 无法被 Ping，因为没有一个实体网络对象来响应
    - Cluster IP 只能结合 Service Port 组成一个具体的通信端口，单独的 Cluster IP 不具备 TCP/IP 通信的基础，并且它们属于 Kubernetes
      集群封闭空间，集群之外的节点如果要访问这个通信端口，需要做一些额外的工作
    - 在 Kubernetes 集群之内，Node IP 网与 Cluster IP 网之间的通信，采用的是 Kubernetes 自己设计的一种编程方式的特殊的路由规则，与我们熟悉的 IP 路由有很大的不同

## Volume（存储卷）

Volume 是 Pod 中能够被多个容器访问的共享目录。

### Kubernetes Volume 和 Docker Volume 区别

Kubernetes 的 Volume 概念，用途和目的与 Docker 的 Volume 比较类似，但两者不能等价：

- Kubernetes 中的 Volume 定义在 Pod 上，被一个 Pod 里的多个容器挂载到具体的文件目录下

- Kubernetes 中的 Volume 与 Pod 的生命周期相同，但与容器的生命周期不相关，当容器终止或重启时，Volume 中的数据也不会丢失

- Kubernetes 支持多种类型的 Volume，例如 GlusterFS，Ceph 等分布式文件系统

### Volume 类型

- emptyDir：它的初始内容为空，并且无需指定宿主机上对应的目录文件，因为这是 Kubernetes 自动分配的一个目录，当 Pod 从 Node 上移除时，emptyDir 中的数据也会被永久删除。用途如下：
    - 临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留
    - 长时间任务的中间过程 CheckPoint 的临时保存目录
    - 一个容器需要从另一个容器中获取数据的目录（多容器共享目录）

- hostPath：Pod 上挂载宿主机上的文件或目录，可以用于以下几方面：
    - 容器应用程序生成的日志文件需要永久保存时，可以使用宿主机的高速文件系统进行存储
    - 需要访问宿主机上 Docker 引擎内部数据结构的容器应用时，可以通过定义 hostPath 为宿主机 /var/lib/docker 目录，使容器内部应用可以直接访问 Docker 的文件系统

- gcePersistentDisk：表示使用谷歌公有云提供的永久磁盘（Persistent Disk，PD）存放 Volume 的数据，PD 上的内容会永久保存，当 Pod 被删除时，PD 只是被卸载（Unmount），但不会被删除。

- awsElasticBlockStore：使用亚马逊公有云提供的 EBS Volume 存储数据。

- NFS：使用 NFS 网络文件系统提供的共享目录存储数据

- iscsi：使用 iSCSI 存储设备上的目录挂载到 Pod 中

- flocker：使用 Flocker 来管理存储卷

- glusterfs：使用 GlusterFs 网络文件系统的目录

- rbd：使用 Ceph 块设备共享存储（Rados Block Device）

- gitRepo：从 Git 仓库 clone 一个 git repository 供 Pod 使用

- secret：为 Pod 提供加密的信息。secret volume 是通过 tmfs（内存文件系统）实现的，这种类型的 volume 总是不会持久化的。

## Persistent Volume

Persistent Volume（PV）可以理解成 Kubernetes 集群中的某个网络存储中对应的一块存储，它与 Volume 很类似，但有如下区别：

- PV 只能是网络存储，不属于任何 Node，但可以在每个 Node 上访问

- PV 并不是定义在 Pod 上的，而是独立于 Pod 之外定义

如果某个 Pod 想申请某种类型的 PV，则首先需要定义一个 Persistent Volume Claim（PVC）对象。

PV 是有状态的对象，它有以下几种状态：

- Available：空闲状态

- Bound：已经绑定到某个 PVC 上

- Released：对应的 PVC 已经删除，但资源还没有被集群收回

- Failed：PV 自动回收失败

## Namespace（命名空间）

Namespace 通过将集群内部的资源对象分配到不同的 Namespace 中，形成逻辑上分组的不同项目，小组或用户组，便于不同的分组在共享使用整个集群的资源的同时还能被分别管理。

Kubernetes 集群在启动后，会创建一个名为"default"的 Namespace，通过 kubectl 可以查看到：

```bash
kubectl get namespace
```

当我们给每个租户创建一个 Namespace 来实现多租户的资源隔离时，还能结合 Kubernetes 的资源配额管理，限定不同的租户能占用的资源，例如 CPU 使用量，内存使用量等。

## Annotation（注解）

Annotation 使用 key/value 键值对的形式进行定义，用于用户任意定义的附加信息，以便于外部工具进行查找。

通常用 Annotation 记录的信息如下：

- build 信息，release 信息，Docker 镜像信息等

- 日志库，监控库，分析库等资源库的地址信息

- 程序调试工具信息，例如工具名称，版本号等

- 团队的联系信息

# 思维导图

![Kubernetes-基本概念和术语-思维导图.png](https://cnymw.github.io/GolangStudy/docs/img/Kubernetes-基本概念和术语-思维导图.png)
