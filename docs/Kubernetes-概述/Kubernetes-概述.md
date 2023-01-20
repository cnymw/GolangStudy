# Kubernetes 概述

## 思维导图预习

![Kubernetes-概述-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-概述/Kubernetes-概述-思维导图.png)

---

## Kubernetes 概述

Kubernetes 是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。 

Kubernetes 拥有一个庞大且快速增长的生态系统，其服务、支持和工具的使用范围广泛。

---

## 部署方式历史

![Kubernetes-概述-部署方式历史.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-概述/Kubernetes-概述-部署方式历史.png)

- 传统部署时代：各个组织机构在物理服务器上运行应用程序
- 虚拟化部署时代：虚拟化技术能够更好地利用物理服务器上的资源，并且因为可轻松地添加或更新应用程序 而可以实现更好的可伸缩性，降低硬件成本等等。
- 容器部署时代：容器类似于 VM，但是它们具有被放宽的隔离属性，可以在应用程序之间共享操作系统（OS）。因此，容器被认为是轻量级的。容器与 VM 类似，具有自己的文件系统、CPU、内存、进程空间等。由于它们与基础架构分离，因此可以跨云和 OS 发行版本进行移植。

---

## Kubernetes 的能力

### 服务发现和负载均衡

Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大，Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

### 存储编排

Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

### 自动部署和回滚

你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器，删除现有容器并将它们的所有资源用于新容器。

### 自动完成装箱计算

Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。

### 自我修复

Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

### 密钥与配置管理

Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

---

## 参考资料

- [kubernetes.io官方文档：Kubernetes 概述](https://kubernetes.io/zh-cn/docs/concepts/overview/)

---

## 思维导图

![Kubernetes-概述-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-概述/Kubernetes-概述-思维导图.png)

---

## B站学习

[从零开始学习k8s：k8s概述](https://www.bilibili.com/video/BV1bV4y147o9/)

![Kubernetes-概述-B站.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-概述/Kubernetes-概述-B站.png)

---

## 抖音学习

![Kubernetes-概述-抖音.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-概述/Kubernetes-概述-抖音.png)
