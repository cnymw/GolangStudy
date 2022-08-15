# Kubernetes 组件

## Kubernetes 集群组件图

![Kubernetes-组件-集群组件图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-组件/Kubernetes-组件-集群组件图.png)

## 控制平面组件（Control Plane Components）

控制平面组件会为集群做出全局决策，比如资源的调度，以及检测和响应集群事件。

控制平面组件可以在集群中的任何节点上运行。在实际使用中，通常会在同一个计算机上启动所有控制平面组件，并且不会在此计算机上运行用户容器。

### kube-apiserver 

apiserver 是 Kubernetes 控制平面的组件，该组件负责公开了 Kubernetes API，负责处理接受请求的工作。apiserver 是 Kubernetes 控制平面的前端。

Kubernetes apiserver 的主要实现是 kube-apiserver。kube-apiserver 设计上考虑了水平扩缩，也就是说，它可通过部署多个实例来进行扩缩。你可以运行 kube-apiserver 的多个实例，并在这些实例之间平衡流量。

### etcd

etcd 是兼顾一致性与高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。

### kube-scheduler

kube-scheduler 是控制平面的组件，负责监视新创建的、未指定运行节点（node）的 Pods，并选择节点来让 Pod 在上面运行。

调度决策考虑的因素包括单个 Pod 及 Pods 集合的资源需求、软硬件及策略约束、亲和性及反亲和性规范、数据位置、工作负载间的干扰及最后时限。

### kube-controller-manager

kube-controller-manager 是控制平面的组件，负责运行控制器进程。

控制器包括：

- 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
- 任务控制器（Job Controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点控制器（Endpoints Controller）：填充端点（Endpoints）对象（即加入 Service 与 Pod）
- 服务帐户和令牌控制器（Service Account & Token Controllers）：为新的命名空间创建默认帐户和 API 访问令牌

### cloud-controller-manager 

cloud-controller-manager 是指嵌入特定云（例如亚马逊 aws，微软 azure 等）的控制逻辑之控制平面组件。cloud-controller-manager 允许你将你的集群连接到云提供商的 API 之上，并将与该云平台交互的组件同与你的集群交互的组件分离开来。

下面的控制器都包含对云平台驱动的依赖：

- 节点控制器（Node Controller）：用于在节点终止响应后检查云提供商以确定节点是否已被删除
- 路由控制器（Route Controller）：用于在底层云基础架构中设置路由
- 服务控制器（Service Controller）：用于创建、更新和删除云提供商负载均衡器

## Node 组件

Node 组件会在每个节点上运行，负责维护运行的 Pod 并提供 Kubernetes 运行环境。

### kubelet 

kubelet 会在集群中每个节点（node）上运行。它保证容器（containers）都运行在 Pod 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。kubelet 不会管理不是由 Kubernetes 创建的容器。

### kube-proxy 

kube-proxy 是集群中每个节点（node）所上运行的网络代理，实现 Kubernetes 服务（Service）概念的一部分。

kube-proxy 维护节点上的一些网络规则，这些网络规则会允许从集群内部或外部的网络会话与 Pod 进行网络通信。

### 容器运行时（Container Runtime）

容器运行环境是负责运行容器的软件。

Kubernetes 支持许多容器运行环境，例如 Docker、containerd、CRI-O 以及 Kubernetes CRI (容器运行环境接口) 的其他任何实现。

## 插件（Addons）

插件使用 Kubernetes 资源（DaemonSet、 Deployment 等）实现集群功能。因为这些插件提供集群级别的功能，插件中命名空间域的资源属于 kube-system 命名空间。

几种常用插件：

- DNS：集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。
- Web 界面（仪表盘）：Dashboard 是 Kubernetes 集群的通用的、基于 Web 的用户界面。它使用户可以管理集群中运行的应用程序以及集群本身，并进行故障排除。
- 容器资源监控：将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中，并提供浏览这些数据的界面。
- 集群层面日志：负责将容器的日志数据保存到一个集中的日志存储中，这种集中日志存储提供搜索和浏览接口。

# 参考资料

- [kubernetes.io 官方文档：Kubernetes 组件](https://kubernetes.io/zh-cn/docs/concepts/overview/components/)

# 思维导图

![Kubernetes-组件-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-组件/Kubernetes-组件-思维导图.png)

# B站学习

[从零开始学习k8s：k8s组件](https://www.bilibili.com/video/BV13G4y1a7oq/)

![Kubernetes-组件-B站.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-组件/Kubernetes-组件-B站.png)

# 抖音学习

![Kubernetes-组件-抖音.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-组件/Kubernetes-组件-抖音.png)
