# Kubernetes namespace

## 什么是 Kubernetes namespace

在 Kubernetes 中，名字空间（Namespace）提供一种机制，将同一集群中的资源划分为相互隔离的组。

同一 namespace 内的资源名称要唯一，但跨 namespace 时没有这个要求。

namespace 作用域仅针对带有 namespace 的对象，例如 Deployment、Service 等，这种作用域对集群访问的对象不适用，例如 StorageClass、Node、PersistentVolume 等。

## 什么时候使用 Kubernetes namespace

namespace 适用于存在很多跨多个团队或项目的用户的场景。对于只有几到几十个用户的集群，根本不需要创建或考虑 namespace。

在多个用户之间划分集群资源的时候，可以使用 namespace。

## 什么时候不必使用 Kubernetes namespace

不必使用多个 namespace 来分隔仅仅轻微不同的资源，例如同一软件的不同版本，应该使用标签（label）来区分同一 namespace 中的不同资源。

## 如何使用 namespace

> 说明： 避免使用前缀 kube- 创建 namespace，因为它是为 Kubernetes 系统 namespace 保留的。

### 查看 namespace

你可以使用以下命令列出集群中现存的 namespace：

```shell
kubectl get namespace
```

Kubernetes 会创建四个初始 namespace：

- default：没有指明使用其它 namespace 的对象所使用的默认 namespace
- kube-system：Kubernetes 系统创建对象所使用的 namespace
- kube-public：这个 namespace 是自动创建的，所有用户（包括未经过身份验证的用户）都可以读取它。这个 namespace 主要用于集群使用，以防某些资源在整个集群中应该是可见和可读的。这个 namespace 的公共方面只是一种约定，而不是要求。
- kube-node-lease：此 namespace 用于与各个节点相关的租约（Lease）对象。节点租期允许 kubelet 发送心跳，由此控制面能够检测到节点故障。

### 为请求设置 namespace

要为当前请求设置 namespace，请使用 --namespace 参数。

```shell
kubectl run nginx --image=nginx --namespace=<namespace 名称>
kubectl get pods --namespace=<namespace 名称>
```

### 设置 namespace 偏好

你可以永久保存 namespace，以用于对应上下文中所有后续 kubectl 命令。

```shell
kubectl config set-context --current --namespace=<namespace 名称>
```

## namespace 和 DNS

当你创建一个 service 时， Kubernetes 会创建一个相应的 DNS 条目。

该条目的形式是 <service 名称>.<namespace 名称>.svc.cluster.local，这意味着如果容器只使用 <service 名称>，它将被解析到本地 namespace 的 service。这对于跨多个 namespace（如开发、测试和生产）使用相同的配置非常有用。如果你希望跨 namespace 访问，则需要使用完全限定域名（FQDN）。

## 并非所有对象都在 namespace 中 

大多数 kubernetes 资源（例如 Pod、Service、副本控制器等）都位于某些 namespace 中。但是 namespace 资源本身并不在 namespace 中。而且底层资源，例如节点和持久化卷不属于任何 namespace。

```shell
# 位于 namespace 中的资源
kubectl api-resources --namespaced=true

# 不在 namespace 中的资源
kubectl api-resources --namespaced=false
```

# 参考资料

- [kubernetes.io官方文档：名字空间](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/namespaces/)

# 思维导图

```markmap
- 什么是 Kubernetes namespace
- 什么时候使用 Kubernetes namespace
- 什么时候不必使用 Kubernetes namespace
- 如何使用 namespace
- namespace 和 DNS
- 并非所有对象都在 namespace 中
```

![Kubernetes-namespace-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-namespace/Kubernetes-namespace-思维导图.png)

# 视频学习

## B站学习

[从零开始学习k8s：k8s namespace](https://www.bilibili.com/video/BV12T411A7HN/)

![Kubernetes-namespace-B站.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-namespace/Kubernetes-namespace-B站.png)

## 抖音学习

![Kubernetes-namespace-抖音.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-namespace/Kubernetes-namespace-抖音.png)
