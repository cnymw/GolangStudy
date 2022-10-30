# Kubernetes Service

## Service 定义

Service 是一种可以访问逻辑上的一组 Pod 的策略，通常称为微服务。

Service 所针对的 Pod 集合通常是通过选择算符来确定的。

## 如何使用 Service

Service 在 Kubernetes 中是一个 REST 对象，和 Pod 类似。像所有的 REST 对象一样，Service 定义可以基于 POST 方式，请求 API server 创建新的实例。 

Kubernetes 为服务分配一个 IP 地址（有时称为 Cluster IP），该 IP 地址由服务代理使用。

服务选择算符的控制器不断扫描与其选择算符匹配的 Pod，然后将所有更新发布到也称为"my-service"的 Endpoint 对象。

服务的默认协议是 TCP，你还可以使用任何其他受支持的协议。

由于许多服务需要公开多个端口，因此 Kubernetes 在服务对象上支持多个端口定义。每个端口定义可以具有相同的 protocol，也可以具有不同的协议。

### 如何使用没有选择算符的 Service

由于选择算符的存在，服务最常见的用法是为 Kubernetes Pod 的访问提供抽象，但是当与相应的 Endpoints 对象一起使用且没有选择算符时，服务也可以为其他类型的后端提供抽象，包括在集群外运行的后端。例如：

- 希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。
- 希望服务指向另一个名字空间（Namespace）中或其它集群中的服务。
- 你正在将工作负载迁移到 Kubernetes。在评估该方法时，你仅在 Kubernetes 中运行一部分后端。

由于此服务没有选择算符，因此不会自动创建相应的 Endpoints 对象。你可以通过手动添加 Endpoints 对象，将服务手动映射到运行该服务的网络地址和端口。

## 虚拟 IP 和 Service 代理

在 Kubernetes 集群中，每个 Node 运行一个 kube-proxy 进程。kube-proxy 负责为 Service 实现了一种 VIP（虚拟 IP）的形式，而不是 ExternalName 的形式。

### 为什么不使用 DNS 轮询

使用服务代理有以下几个原因：

- DNS 实现的历史由来已久，它不遵守记录 TTL，并且在名称查找结果到期后对其进行缓存。
- 有些应用程序仅执行一次 DNS 查找，并无限期地缓存结果。
- 即使应用和库进行了适当的重新解析，DNS 记录上的 TTL 值低或为零也可能会给 DNS 带来高负载，从而使管理变得困难。

