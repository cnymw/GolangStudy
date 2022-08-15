# Kubernetes 理解 Kubernetes 对象

## 什么是 Kubernetes 对象

在 Kubernetes 系统中，Kubernetes 对象是持久化的实体。Kubernetes 使用这些实体去表示整个集群的状态。

Kubernetes 对象描述了如下信息：

- 哪些容器化应用正在运行（以及在哪些节点上运行）
- 可以被应用使用的资源
- 关于应用运行时表现的策略，比如重启策略、升级策略以及容错策略

### Kubernetes 目标性记录

Kubernetes 对象是"目标性记录"，一旦创建该对象，Kubernetes 系统将不断工作以确保该对象存在。

### Kubernetes 集群的期望状态

通过创建对象，你就是在告知 Kubernetes 系统，你想要的集群工作负载状态看起来应是什么样子的，这就是 Kubernetes 集群所谓的期望状态（Desired State）。

### 如何操作 Kubernetes 对象

无论是创建、修改或者删除，都需要使用 Kubernetes API。比如，当使用 kubectl 命令行接口（CLI）时，CLI 会调用必要的 Kubernetes API；也可以在程序中使用客户端库，来直接调用 Kubernetes API。

### 对象规约（Spec）与状态（Status）

必须在创建对象时设置 spec，描述你希望对象所具有的特征：期望状态（Desired State）。

status 描述了对象的当前状态（Current State），它是由 Kubernetes 系统和组件设置并更新的。

在任何时刻，Kubernetes 控制平面都一直都在积极地管理着对象的实际状态，以使之达成期望状态。

### 如何描述 Kubernetes 对象

创建 Kubernetes 对象时，必须提供对象的 spec，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。 

这里有一个 .yaml 示例文件，展示了如何用 Kubernetes 部署 nginx：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # 告知 Deployment 运行 2 个与该模板匹配的 Pod
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

然后通过 kubectl 命令行接口（CLI）的 apply 命令，可以部署 nginx：

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

### yaml 必须字段

在想要创建的 Kubernetes 对象所对应的 .yaml 文件中，需要配置的字段如下：

- apiVersion：创建该对象所使用的 Kubernetes API 的版本
- kind：想要创建的对象的类别
- metadata：帮助唯一标识对象的一些数据，包括一个 name 字符串、UID 和可选的 namespace
- spec：你所期望的该对象的状态

# 参考资料

- [kubernetes.io 官方文档：理解 Kubernetes 对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/)

# 思维导图

![Kubernetes-理解对象-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-理解对象/Kubernetes-理解对象-思维导图.png)

# B站学习

[从零开始学习k8s：k8s组件](https://www.bilibili.com/video/BV13G4y1a7oq/)

![Kubernetes-组件-B站.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-组件/Kubernetes-组件-B站.png)

# 抖音学习

![Kubernetes-组件-抖音.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-组件/Kubernetes-组件-抖音.png)
