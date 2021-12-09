# Kubernetes kubectl命令

`kubectl` 命令用于控制 Kubernetes 集群。至于配置，`kubectl` 在 ` $HOME/.kube` 目录中查找名为 config 的文件。通过设置 `KUBECONFIG`
环境变量或者设置 `--kubeconfig` 标志位，可以指定其他 kubeconfig 文件。

本文章介绍 `kubectl` 语法，并提供常见示例。

## 语法

使用以下语法从终端运行 `kubectl` 命令：

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

这里的 `command`，`TYPE`，`NAME`，`flags` 含义如下：

- command：指定要对一个或多个资源执行的操作，例如 create，get，describe，delete

- TYPE：指定资源类型。资源类型不区分大小写，可以指定单数，复数或缩写形式。例如，以下命令输出相同：

```bash
kubectl get pod pod1
kubectl get pods pod1
kubectl get po pod1
```

- NAME：指定资源的名称。名称区分大小写。如果省略名称，将显示所有资源的详细信息，例如 `kubectl get pods`。 对多个资源执行操作时，可以按类型和名称指定每个资源，或指定一个或多个文件：

    - 要按类型和名称指定资源，请执行以下操作：

        - 若资源类型相同，则对其进行分组：`TYPE1 name1 name2 name<#>` ，示例：`kubectl get pod example-pod1 example-pod2`

        - 要单独指定多个资源类型：`TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name<#>`
          ，示例：`kubectl get pod/example-pod1 replicationcontroller/example-rc1`

    - 使用一个或多个文件指定资源：`-f file1 -f file2 -f file<#>`

        - 使用 YAML 而不是 JSON，因为 YAML 更易于使用，特别是对于配置文件，示例：`kubectl get -f ./pod.yaml`

- flags：指定可选标志。例如，可以使用 -s 或 --server 标志指定 Kubernetes API 服务器的地址和端口

> 注意：从命令行指定的 flag 将覆盖默认值和任何相应的环境变量

## 操作

下面包括所有 `kubectl` 操作的简短说明和通用语法。

### alpha

列出与 alpha 功能相对应的可用命令，默认情况下，Kubernetes 集群中未启用这些功能。

```bash
kubectl alpha SUBCOMMAND [flags]
```

### annotate

添加或更新一个或多个资源的注释。

```bash
kubectl annotate (-f FILENAME | TYPE NAME | TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]
```

### api-resources

列出可用的 API 资源。

```bash
kubectl api-resources [flags]
```

### api-versions

列出可用的 API 版本。

```bash
kubectl api-versions [flags]
```

### apply

来自于文件或 stdin 的配置更改应用到资源上。

```bash
kubectl apply -f FILENAME [flags]	
```

### attach

连接(attach)到正在运行的容器上，用于查看输出流或和容器进行交互（stdin）。

```bash
kubectl attach POD -c CONTAINER [-i] [-t] [flags]	
```

### auth

检查授权。

```bash
kubectl auth [flags] [options]	
```

### autoscale

自动缩放由 replication controller 管理的 pod 集合。

```bash
kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [flags]	
```

### certificate

修改证书资源。

```bash
kubectl certificate SUBCOMMAND [options]	
```

### cluster-info

显示集群中主机和服务的端点（endpoint）信息。

```bash
kubectl cluster-info [flags]	
```

### completion

为指定的 shell（bash 或 zsh）输出 shell 补全代码。

```bash
kubectl completion SHELL [options]	
```

### config

修改 kubeconfig 文件。

```bash
kubectl config SUBCOMMAND [flags]	
```

### convert

在不同 API 版本之间切换配置文件。YAML 和 JSON 格式都可以接受。注意需要安装 kubectl 转换插件。

```bash
kubectl convert -f FILENAME [options]	
```

### cordon

将节点标记为不可调度。

```bash
kubectl cordon NODE [options]	
```

### cp

在容器中复制文件和目录，可以复制进入容器，也可以从容器中复制出来。

```bash
kubectl cp <file-spec-src> <file-spec-dest> [options]	
```

### create

从文件或 stdin 读信息，创建一个或多个资源。

```bash
kubectl create -f FILENAME [flags]	
```

### delete

从文件，stdin 读信息或指定标签选择器，名称，资源选择器或资源中删除资源。

```bash
kubectl delete (-f FILENAME | TYPE [NAME | /NAME | -l label | --all]) [flags]	
```

### describe

显示一个或多个资源的详细状态。

```bash
kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | /NAME | -l label]) [flags]	
```

### diff

比较在线配置和文件或 stdin 数据的差别。

```bash
kubectl diff -f FILENAME [flags]	
```

### drain

释放节点，为维护做好准备。

```bash
kubectl drain NODE [options]	
```

### edit

使用默认编辑器编辑，来更新服务器上一个或多个资源的定义。

```bash
kubectl edit (-f FILENAME | TYPE NAME | TYPE/NAME) [flags]	
```

### exec

对 pod 里的容器执行命令。

```bash
kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [-- COMMAND [args...]]	
```

### explain

获取各种资源的文档。例如 pod，节点，serivce 等。

```bash
kubectl explain [--recursive=false] [flags]	
```

### expose

将 replication controller，service 或 pod 暴露为新的 Kubernetes Service。

```bash
kubectl expose (-f FILENAME | TYPE NAME | TYPE/NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [flags]	
```

### get

列出一个或多个资源。

```bash
kubectl get (-f FILENAME | TYPE [NAME | /NAME | -l label]) [--watch] [--sort-by=FIELD] [[-o | --output]=OUTPUT_FORMAT] [flags]	
```

### kustomize

列出根据 kustomize.yaml 文件中的指令生成的一组 API 资源。参数必须是包含文件的目录路径，或者是 git 仓库 URL，路径后缀指定里与存储库根目录相同的路径。

```bash
kubectl kustomize <dir> [flags] [options]	
```

### label

添加或更新一个或多个资源的标签。

```bash
kubectl label (-f FILENAME | TYPE NAME | TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]	
```

### logs

打印 pod 中容器的日志。

```bash
kubectl logs POD [-c CONTAINER] [--follow] [flags]	
```

### options

列出适用于全部命令的全局命令行选项列表。

```bash
kubectl options	
```

### patch

通过使用策略性的 merge 补丁程序，来更新资源的一个或多个字段。

```bash
kubectl patch (-f FILENAME | TYPE NAME | TYPE/NAME) --patch PATCH [flags]	
```

### plugin

提供与插件交互的实用程序。

```bash
kubectl plugin [flags] [options]	
```

### port-forward

将一个或多个本地端口转发到 pod。

```bash
kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N] [flags]	
```

### proxy

运行 Kubernetes API 服务器的代理。

```bash
kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [flags]	
```

### replace

从文件或 stdin 读取信息，来替换资源。

```bash
kubectl replace -f FILENAME	
```

### rollout

管理资源的 rollout。有效的资源类型包括：deployments，daemonsets 和 statefulsets。

```bash
kubectl rollout SUBCOMMAND [options]	
```

### run

在集群上运行指定的镜像 image。

```bash
kubectl run NAME --image=image [--env="key=value"] [--port=port] [--dry-run=server|client|none] [--overrides=inline-json] [flags]	
```

### scale

更新 replication controller 指定的资源数量。

```bash
kubectl scale (-f FILENAME | TYPE NAME | TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]	
```

### set

配置应用程序资源。

```bash
kubectl set SUBCOMMAND [options]	
```

### taint

更新一个或多个节点上的污染 taint。

```bash
kubectl taint NODE NAME KEY_1=VAL_1:TAINT_EFFECT_1 ... KEY_N=VAL_N:TAINT_EFFECT_N [options]	
```

### top

显示资源（CPU/内存/存储）使用情况。

```bash
kubectl top [flags] [options]	
```

### uncordon

将节点标记为可调度。

```bash
kubectl uncordon NODE [options]	
```

### version

显示在客户端和服务器上运行的 Kubernetes 版本。

```bash
kubectl version [--client] [flags]	
```

### wait

实验性：等待一个或多个资源上的特定条件。

```bash
kubectl wait ([-f FILENAME] | resource.group/resource.name | resource.group [(-l label | --all)]) [--for=delete|--for condition=available] [options]	
```

## 输出选项

有关如何格式化或排序某些命令的输出信息，请参考以下部分。

### 格式化输出

所有 kubectl 命令的默认输出格式都是可读的纯文本格式。要以特定格式将详细信息输出到终端窗口，可以将 -o 或 --output 标志添加到 kubectl 命令中。

### 语法

```bash
kubectl [command] [TYPE] [NAME] -o <output_format>
```

根据 kubectl 操作，支持以下输出格式：

输出格式|描述 
---|:--:
-o custom-columns=spec|使用逗号分割的自定义列表来打印信息 
-o custom-columns-file=filename|使用filename文件中的自定义模板打印表格 
-o json|输出JSON格式的API对象
-o jsonpath=template|打印jsonpath表达式中定义的字段 
-o jsonpath-file=filename|打印filename文件中jsonpath表达式定义的字段 
-o name|仅打印资源名称，不打印其他内容
-o wide|以纯文本格式输出任何附加信息。对于 pods，包含了节点名称。 
-o yaml|输出YAML格式的API对象。