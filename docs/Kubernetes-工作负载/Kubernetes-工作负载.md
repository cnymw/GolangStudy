# Kubernetes 工作负载

## 工作负载概念

工作负载是在 Kubernetes 上运行的应用程序。

在 Kubernetes 中，无论你的负载是由单个组件还是由多个一同工作的组件构成，你都可以在一组 Pod 中运行它。在 Kubernetes 中，Pod 代表的是集群上处于运行状态的一组容器的集合。

---

## Pod

### Pod 概念

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

### 什么是 Pod？

Pod 的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离方面，即用来隔离容器的技术。在 Pod 的上下文中，每个独立的应用可能会进一步实施隔离。

Pod 类似于共享名字空间并共享文件系统卷的一组容器。

### 如何使用 Pod

Pod 通常不是直接创建的，而是使用工作负载资源创建的。

Kubernetes 集群中的 Pod 主要有两种用法：

- 运行单个容器的 Pod。"每个 Pod 一个容器" 模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
- 运行多个协同工作的容器的 Pod。Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。这些位于同一位置的容器可能形成单个内聚的服务单元：一个容器将文件从共享卷提供给公众，而另一个单独的 “边车”（sidecar）容器则刷新或更新这些文件。Pod 将这些容器和存储资源打包为一个可管理的实体。

---

## 内置的工作负载

- Deployment：Deployment 很适合用来管理你的集群上的无状态应用，Deployment 中的所有 Pod 都是相互等价的，并且在需要的时候被替换。
- ReplicaSet：ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。
- StatefulSet：让你能够运行一个或者多个以某种方式跟踪应用状态的 Pod。
- DaemonSet：定义提供节点本地支撑设施的 Pod。
- Job：定义一些一直运行到结束并停止的任务。Job 用来执行一次性任务。
- CronJob：定义一些一直运行到结束并停止的任务。CronJob 用来执行的根据时间规划反复运行的任务。
- ReplicationController：确保在任何时候都有特定数量的 Pod 副本处于运行状态。

---

## Deployments

### Deployments 概念

一个 Deployment 为 Pod 和 ReplicaSet 提供声明式的更新能力。

你负责描述 Deployment 中的目标状态，而 Deployment 控制器（Controller）以受控速率更改实际状态，使其变为期望状态。

---

## ReplicaSet

### ReplicaSet 概念

ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。 因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。

---

## StatefulSet

### StatefulSet 概念

StatefulSet 是用来管理有状态应用的工作负载 API 对象。

StatefulSet 用来管理某 Pod 集合的部署和扩缩，并为这些 Pod 提供持久存储和持久标识符。

---

## DaemonSet

### DaemonSet 概念

DaemonSet 确保全部（或者某些）节点上运行一个 Pod 的副本。当有节点加入集群时，也会为他们新增一个 Pod。当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

### DaemonSet 典型用法

- 在每个节点上运行集群守护进程
- 在每个节点上运行日志收集守护进程
- 在每个节点上运行监控守护进程

---

## Job

### Job 概念

Job 会创建一个或者多个 Pod，并将继续重试 Pod 的执行，直到指定数量的 Pod 成功终止。

### Job 状态流转

- 随着 Pod 成功结束，Job 跟踪记录成功完成的 Pod 个数。 当数量达到指定的成功个数阈值时，任务（即 Job）结束。
- 删除 Job 的操作会清除所创建的全部 Pod。
- 挂起 Job 的操作会删除 Job 的所有活跃 Pod，直到 Job 被再次恢复执行。

---

## CronJob

### CronJob 概念

CronJob 创建基于时隔重复调度的 Jobs。

### CronJob 用途

CronJob 用于执行周期性的动作，例如备份、报告生成等。这些任务中的每一个都应该配置为周期性重复的（例如：每天/每周/每月一次）；你可以定义任务开始执行的时间间隔。

---

## ReplicationController

### ReplicationController 概念

ReplicationController 确保在任何时候都有特定数量的 Pod 副本处于运行状态。换句话说，ReplicationController 确保一个 Pod 或一组同类的 Pod 总是可用的。

### ReplicationController 如何工作

当 Pod 数量过多时，ReplicationController 会终止多余的 Pod。当 Pod 数量太少时，ReplicationController 将会启动新的 Pod。

## 思维导图

![Kubernetes-工作负载-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-工作负载/Kubernetes-工作负载-思维导图.png)
