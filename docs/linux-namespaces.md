# Linux namespaces 命名空间

## namespaces 定义

namespaces 是 Linux 内核的一个特性，它对内核资源进行分区，使得一组进程可以看到一组资源，而另一组进程则可以看到另一组资源。该特性的工作原理是，一组资源和进程具有相同的 namespaces，但这些 namespaces 引用不同的资源。

两个（或更多）namespaces 可以位于同一台物理计算机上，namespaces 可以共享对某些资源的访问权限，也可以具有独占访问权限。

## 常用的 namespaces

### Process isolation(PID namespaces)

- [PID namespace](https://www.redhat.com/sysadmin/pid-namespace)

### Network interfaces(net namespace)

- [net namespace](https://www.redhat.com/sysadmin/net-namespaces)

### Unix Timesharing System (uts namespace)

- [uts namespace](https://www.redhat.com/sysadmin/uts-namespace)

### User namespace

- [user namespace](https://www.redhat.com/sysadmin/building-container-namespaces)

### Mount (mnt namespace)

- [mnt namespace](https://www.redhat.com/sysadmin/mount-namespaces)

### Interprocess communication (IPC)

IPC 通过使用共享内存区域，消息队列和信号量来处理进程之间的通信，这种管理最常见的应用可能是使用数据库。

共享内存允许两个或多个程序访问相同的信息，一个程序的更改将立即对另一个程序可见。

### Cgroups

- [cgroups namespace:part 1](https://www.redhat.com/sysadmin/cgroups-part-one)
- [cgroups namespace:part 2](https://www.redhat.com/sysadmin/cgroups-part-two)
- [cgroups namespace:part 3](https://www.redhat.com/sysadmin/cgroups-part-three)
- [cgroups namespace:part 4](https://www.redhat.com/sysadmin/cgroups-part-four)

## 思维导图

![linux-namespaces-思维导图.png](https://cnymw.github.io/GolangStudy/docs/img/linux-namespaces-思维导图.png)
