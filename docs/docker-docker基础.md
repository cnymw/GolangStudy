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

## docker 操作参数基本解读


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
