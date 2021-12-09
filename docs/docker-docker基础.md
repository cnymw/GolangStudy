# docker 基础

## 云计算平台

云计算是一种资源的服务模式，该模式可以实现随时随地，便捷地按需地从可配置计算资源共享池中获取所需的资源（如网络，服务器，存储，应用及服务）

### 经典云计算架构

经典云计算架构包括 IaaS，PaaS，Saas 三层服务：

- IaaS：Infrastructure as a Service，基础设施即服务，为基础设施运维人员服务，提供计算，存储，网络及其他基础资源，云平台使用者可以在上面部署和运行包括操作系统和应用程序在内的任意软件，无需再为基础设施的管理而分心。
- PaaS：Platform as a Service，平台即服务，为应用开发人员服务，提供支撑应用运行所需的软件运行时环境，相关工具与服务，如数据库服务，日志服务，监控服务等，让应用开发者可以专注于核心业务的开发。
- SaaS：Software as a Service，软件即服务，为一般用户服务，提供了一套完整可用的软件系统，让一般用户无需关注技术细节，只需通过浏览器，应用客户端等方式就能使用部署在云上的应用服务。

## 容器化

### 容器技术生态系统

![容器技术生态系统](https://cnymw.github.io/GolangStudy/docs/img/docker-docker基础-容器技术生态系统.jpg)

### 容器技术带来的好处

- 持续部署与测试：容器消除了线上线下的环境差异，保证了应用生命周期的环境一致性和标准化。
- 跨云平台支持：越来越多的云平台都支持容器，用户再也无需担心受到云平台的捆绑，同时也让应用多平台混合部署成为可能。
- 环境标准化和版本控制：基本容器提供的环境一致性和标准化，能够对整个应用运行环境实现版本控制，一旦出现故障可以快速回滚。
- 高资源利用率与隔离：容器没有管理程序的额外开销，与底层共享操作系统，性能更加优良，系统负载更低，在同等条件下可以运行更多的应用实例，可以更充分地利用系统资源。
- 容器跨平台性与镜像：容器在原有 Linux 容器的基础上进行大胆革新，为容器设定了一整套标准化的配置方法，将应用及其依赖的运行环境打包成镜像，真正实现了"构建一次，到处运行"的理念，大大提高了容器的跨平台性。
- 应用镜像仓库：因为 docker 的跨平台适配性，相当于为用户提供了一个非常有用的应用商店，所有人都可以自由地下载微服务组件。

### 什么是 docker

docker 是以 docker 容器为资源分割和调度的基本单位，封装整个软件运行时环境，为开发者和系统管理员设计的，用于构建，发布和运行分布式应用的平台。

它是一个跨平台，可移植并且简单易用的容器解决方案。

### 什么是容器云

容器云以容器为资源分割和调度的基本单位，封装整个软件运行时环境，为开发者和系统管理员提供用于构建，发布和运行分布式应用的平台。

- 当容器云专注于资源共享与隔离，容器编排与部署时，它接近传统的IaaS
- 当容器云渗透到应用支撑与运行时环境时，它接近传统的PaaS

### 什么是 docker registry

docker registry 是存储容器镜像的仓库，用户可以通过 docker client 与 docker registry 进行通信，以此来完成镜像的搜索，下载和上传等相关操作。

Docker Hub 是由 docker 公司在互联网上提供的一个镜像仓库，提供镜像的公有与私有存储服务，它是用户最主要的镜像来源。

## docker 操作参数基本解读

### docker 命令整体概览

```bash
$ sudo docker

Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default
                           "/Users/user/.docker")
  -c, --context string     Name of the context to use to connect to the
                           daemon (overrides DOCKER_HOST env var and
                           default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level
                           ("debug"|"info"|"warn"|"error"|"fatal")
                           (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default
                           "/Users/user/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default
                           "/Users/user/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default
                           "/Users/user/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  app*        Docker Application (Docker Inc., v0.8.0)
  builder     Manage builds
  buildx*     Build with BuildKit (Docker Inc., v0.3.1-tp-docker)
  checkpoint  Manage checkpoints
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  deploy      Deploy a new stack or update an existing stack
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

整理一下所有的命令，如下图所示：

![docker操作参数解读](https://cnymw.github.io/GolangStudy/docs/img/docker-docker基础-操作参数解读.jpg)

所有的命令结构图如下所示：

![docker命令结构图](https://cnymw.github.io/GolangStudy/docs/img/docker-docker基础-命令结构图.png)

### docker info

docker info 命令用于检查 docker 是否正确安装。

```bash
$ docker info

Client:
 Debug Mode: false
 Plugins:
  app: Docker Application (Docker Inc., v0.8.0)
  buildx: Build with BuildKit (Docker Inc., v0.3.1-tp-docker)
  ...
```

### docker version

docker version 命令用于检查 docker 版本信息

```bash
$ docker version

Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:22:34 2019
 OS/Arch:           darwin/amd64
 Experimental:      true

...
```

### docker run

docker run 命令用于基于特定的镜像创建一个容器，并依据选项来控制该容器。

以下命令从 ubuntu 镜像启动一个容器，并执行 echo 命令打印。执行完命令后，容器将停止运行。

```bash
$ sudo docker run ubuntu echo "Hello World"

Hello World
```

以下命令启动一个容器，并为它分配一个伪终端执行 bash 命令，用户可以在该伪终端与容器进行交互。

- -i 表示使用交互模式，始终保持输入流开放
- -t 表示分配一个伪终端，即可在容器中利用打开的伪终端进行交互操作
- -c 用于给运行在容器中的所有进程分配 CPU 的 shares 值，这是一个相对权重，实际的处理速度还与宿主机的 CPU 相关
- -m 用于限制为容器中所有进程分配的内存信息，以 B，K，M，G 为单位
- -v 用于挂载一个 volume，可以用多个 -v 参数同时挂载多个 volume
- -p 用于将容器的端口暴露给宿主机的端口，通过端口的暴露，可以让外部主机通过宿主机暴露的端口来访问容器内的应用。

```bash
$sudo docker run -it ubuntu bash

root@test:/#
```

### docker start/stop/restart

docker run 命令可以新建一个容器来运行，对于已经存在的容器，可以通过 docker start/stop/restart 命令来启动，停止和重启。

### docker pull

docker pull 命令用于从 Docker registry 中拉取 image 或 repository。

### docker push

docker push 命令可以将本地的 image 或 repository 推送到 Docker Hub 的公共或私有镜像库，以及私有服务器。

### docker images

docker images 命令可以列出主机上的镜像，默认只列出最顶层的镜像

### docker rmi/rm

docker rmi 用于删除镜像。

docker rm 命令用于删除容器。

需要注意的是，使用 rmi 删除镜像时，如果已有基于该镜像启动的容器存在，则无法直接删除，需要首先删除容器。

### docker attach

docker attach 用于连接到正在运行的容器，观察该容器的运行情况，或与容器的主进程进行交互。

### docker inspect

docker inspect 命令可以查看镜像和容器的详细信息，默认会列出全部信息，可以通过 --format 参数来指定输出的模板格式，以便输出特定信息。

### docker ps

docker ps 命令可以查看容器的相关信息，默认只显示正在运行的容器的信息。最常用的功能就是查看容器的 Container ID，以便对特定容器进行操作。

### docker commit

docker commit 命令可以将一个容器固化为一个新的镜像。当需要制作特定的镜像时，会进行修改容器的配置，如在容器中安装特定工具等，通过 commit 命令可以将这些修改保存起来，使其不会因为容器的停止而丢失。

### docker events/history/logs

docker events 用于查看 Docker 系统的日志信息，会打印出实时的系统事件。

docker history 命令会打印出指定镜像的历史版本信息，即构建该镜像的每一层镜像的命令记录。

docker logs 命令会打印出容器中进程的运行日志。