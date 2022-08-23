# Kubernetes 标签

## 什么是 Kubernetes 标签

标签（Labels）是附加到 Kubernetes 对象（比如 Pods）上的键值对。

标签示例如下：

```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

### 标签作用

- 标签旨在用于指定对用户有意义且相关的对象的标识属性，但不直接对核心系统有语义含义。 
- 标签可以用于组织和选择对象的子集。

### 标签特点

- 每个对象都可以定义一组键/值标签。
- 每个键对于给定对象必须是唯一的。
- 标签能够支持高效的查询和监听操作，对于用户界面和命令行是很理想的。

## 设计标签的目的

标签使用户能够以松散耦合的方式将他们自己的组织结构映射到系统对象，而无需客户端存储这些映射。

服务部署和批处理流水线通常是多维实体（例如，多个分区或部署、多个发行序列、多个层，每层多个微服务）。管理通常需要交叉操作，这打破了严格的层次表示的封装，特别是由基础设施而不是用户确定的严格的层次结构。

示例标签：

- 发行版本 "release" : "stable", "release" : "canary"
- 运行环境 "environment" : "dev", "environment" : "qa", "environment" : "production"

## 标签语法

有效的标签键有两个段：可选的前缀和名称，用斜杠（/）分隔。 

名称段是必需的，必须小于等于 63 个字符，以字母数字字符（[a-z0-9A-Z]）开头和结尾，带有破折号（-），下划线（_），点（ .）和之间的字母数字。

前缀是可选的。如果指定，前缀必须是 DNS 子域：由点（.）分隔的一系列 DNS 标签，总共不超过 253 个字符，后跟斜杠（/）。

如果省略前缀，则假定标签键对用户是私有的。向最终用户对象添加标签的自动系统组件（例如 kube-scheduler、kube-controller-manager、 kube-apiserver、kubectl 或其他第三方自动化工具）必须指定前缀。

kubernetes.io/ 和 k8s.io/ 前缀是为 Kubernetes 核心组件保留的。

有效标签值：

- 必须为 63 个字符或更少（可以为空）
- 除非标签值为空，必须以字母数字字符（[a-z0-9A-Z]）开头和结尾
- 包含破折号（-）、下划线（_）、点（.）和字母或数字

以下是一个有 environment: production 和 app: nginx 标签的 Pod 配置文件：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## 标签选择运算符

与名称和 UID 不同，标签不支持唯一性。通常，我们希望许多对象携带相同的标签。

通过标签选择算符，客户端/用户可以识别一组对象。标签选择算符是 Kubernetes 中的核心分组原语。

在多个需求的情况下，必须满足所有要求，因此逗号分隔符充当逻辑与（&&）运算符。

### 基于等值的需求

基于等值或基于不等值的需求允许按标签键和值进行过滤。匹配对象必须满足所有指定的标签约束，尽管它们也可能具有其他标签。可接受的运算符有 =、== 和 != 三种。前两个表示相等（并且是同义词），而后者表示不相等。

例如：

```shell
environment = production
tier != frontend
```

### 基于集合的需求

基于集合的标签需求允许你通过一组值来过滤键。 持三种操作符：in、notin 和 exists（只可以用在键标识符上）。

例如：

```shell
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

## API

两种标签选择算符都可以通过 REST 客户端用于 list 或者 watch 资源。 

### 基于等值的需求

例如，使用 kubectl 定位 apiserver，可以使用基于等值的标签选择算符可以这么写：

```shell
kubectl get pods -l environment=production,tier=frontend
```

### 基于集合的需求

或者使用基于集合的需求：

```shell
kubectl get pods -l 'environment in (production),tier in (frontend)'
```

# 参考资料

- [kubernetes.io官方文档：标签和选择算符](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)

# 思维导图

![Kubernetes-标签-思维导图.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-标签/Kubernetes-标签-思维导图.png)

# 视频学习

## B站学习

[从零开始学习k8s：k8s标签](https://www.bilibili.com/video/BV1wa411o7wm/)

![Kubernetes-标签-B站.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-标签/Kubernetes-标签-B站.png)

## 抖音学习

![Kubernetes-标签-抖音.png](https://cnymw.github.io/GolangStudy/docs/Kubernetes-标签/Kubernetes-标签-抖音.png)
