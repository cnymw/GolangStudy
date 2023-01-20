# Kubernetes 网络模型

## 思维导图预习

![Kubernetes-网络模型-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-网络模型/Kubernetes-网络模型-思维导图.png)

## 什么是网络模型

计算机网络是指由通信线路互相连接的许多自主工作的计算机构成的集合体，各个部件之间以何种规则进行通信，就是网络模型研究的问题。

## 什么是 Kubernetes 网络模型

集群中每一个 Pod 都会获得自己的、独一无二的 IP 地址，这是一个干净的，向后兼容的网络模型。

在这个模型里，从端口分配，命名，服务发现，负载均衡，应用配置和迁移的角度来看，Pod 可以被视作虚拟机或物理主机。

## Kubernetes 网络要求

Kubernetes 强制要求所有网络设施都满足以下基本要求：

- Pod 能够与所有其他节点上的 Pod 通信，且不需要网络地址转译（NAT）
- 节点上的代理（例如：系统守护进程，kubelet）可以和节点上所有 Pod 通信

## 容器共享 Pod 网络

Kubernetes 的 IP 地址存在于 Pod 范围内，容器共享它们的网络命名空间，包括 Pod 的 IP 地址和 MAC 地址。

这意味着：

- Pod 内的容器都可以通过 localhost 到达对方的端口。
- Pod 内的容器需要相互协调端口的使用。

## Kubernetes 网络解决的问题

Kubernetes 网络解决五方面的问题：

1. 一个 Pod 中的容器之间通过本地回路（loopback）通信。
2. 集群网络在不同 Pod 之间提供通信。
3. Service API 允许你向外暴露 Pod 中运行的应用，以支持来自于集群外部的访问。 
4. Ingress 提供专门用于暴露 HTTP 应用程序，网站和 API 的额外功能。
5. 可以使用 Service 来发布仅供集群内部使用的服务。

## 思维导图

![Kubernetes-网络模型-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-网络模型/Kubernetes-网络模型-思维导图.png)
