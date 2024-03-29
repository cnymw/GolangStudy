# redis 集群

Redis 集群是一个分布式（distributed）、容错（fault-tolerant）的 Redis 实现， 集群可以使用的功能是普通单机 Redis 所能使用的功能的一个子集（subset）。

Redis 集群中不存在中心（central）节点或者代理（proxy）节点， 集群的其中一个主要设计目标是达到线性可扩展性（linear scalability）。

Redis 集群为了保证一致性（consistency）而牺牲了一部分容错性： 系统会在保证对网络断线（net split）和节点失效（node failure）具有有限（limited）抵抗力的前提下， 尽可能地保持数据的一致性。

> 集群将节点失效视为网络断线的其中一种特殊情况。

集群的容错功能是通过使用主节点（master）和从节点（slave）两种角色（role）的节点（node）来实现的：

- 主节点和从节点使用完全相同的服务器实现， 它们的功能（functionally）也完全一样， 但从节点通常仅用于替换失效的主节点。

- 不过， 如果不需要保证“先写入，后读取”操作的一致性（read-after-write consistency）， 那么可以使用从节点来执行只读查询。

## Redis 集群实现的功能子集

Redis 集群实现了单机 Redis 中， 所有处理单个数据库键的命令。

针对多个数据库键的复杂计算操作， 比如集合的并集操作、合集操作没有被实现， 那些理论上需要使用多个节点的多个数据库键才能完成的命令也没有被实现。

Redis 集群不像单机 Redis 那样支持多数据库功能， 集群只使用默认的 0 号数据库， 并且不能使用 SELECT index 命令。

## Redis 集群协议中的客户端和服务器

Redis 集群中的节点有以下责任：

- 持有键值对数据。

- 记录集群的状态，包括键到正确节点的映射（mapping keys to right nodes）。

- 自动发现其他节点，识别工作不正常的节点，并在有需要时，在从节点中选举出新的主节点。

为了执行以上列出的任务， 集群中的每个节点都与其他节点建立起了“集群连接（cluster bus）”， 该连接是一个 TCP 连接， 使用二进制协议进行通讯。

节点之间使用 Gossip 协议 来进行以下工作：

- 传播（propagate）关于集群的信息，以此来发现新的节点。

- 向其他节点发送 PING 数据包，以此来检查目标节点是否正常运作。

- 在特定事件发生时，发送集群信息。

因为集群节点不能代理（proxy）命令请求， 所以客户端应该在节点返回 -MOVED 或者 -ASK 转向（redirection）错误时， 自行将命令请求转发至其他节点。

因为客户端可以自由地向集群中的任何一个节点发送命令请求， 并可以在有需要时， 根据转向错误所提供的信息， 将命令转发至正确的节点， 所以在理论上来说， 客户端是无须保存集群状态信息的。

不过， 如果客户端可以将键和节点之间的映射信息保存起来， 可以有效地减少可能出现的转向次数， 籍此提升命令执行的效率。

## 键分布模型

Redis 集群的键空间被分割为 16384 个槽（slot）， 集群的最大节点数量也是 16384 个。

> 推荐的最大节点数量为 1000 个左右。

每个主节点都负责处理 16384 个哈希槽的其中一部分。

当我们说一个集群处于“稳定”（stable）状态时， 指的是集群没有在执行重配置（reconfiguration）操作， 每个哈希槽都只由一个节点进行处理。

> 重配置指的是将某个/某些槽从一个节点移动到另一个节点。

> 一个主节点可以有任意多个从节点， 这些从节点用于在主节点发生网络断线或者节点失效时， 对主节点进行替换。

以下是负责将键映射到槽的算法：

```text
HASH_SLOT = CRC16(key) mod 16384
```

以下是该算法所使用的参数：

- 算法的名称: XMODEM (又称 ZMODEM 或者 CRC-16/ACORN)

- 结果的长度: 16 位

- 多项数（poly）: 1021 (也即是 x16 + x12 + x5 + 1)

- 初始化值: 0000

- 反射输入字节（Reflect Input byte）: False

- 发射输出 CRC （Reflect Output CRC）: False

- 用于 CRC 输出值的异或常量（Xor constant to output CRC）: 0000

- 该算法对于输入 "123456789" 的输出: 31C3

在我们的测试中， CRC16 算法可以很好地将各种不同类型的键平稳地分布到 16384 个槽里面。

## 集群节点属性

每个节点在集群中都有一个独一无二的 ID ， 该 ID 是一个十六进制表示的 160 位随机数， 在节点第一次启动时由 /dev/urandom 生成。

节点会将它的 ID 保存到配置文件， 只要这个配置文件不被删除， 节点就会一直沿用这个 ID 。

节点 ID 用于标识集群中的每个节点。 一个节点可以改变它的 IP 和端口号， 而不改变节点 ID 。 集群可以自动识别出 IP/端口号的变化， 并将这一信息通过 Gossip 协议广播给其他节点知道。

以下是每个节点都有的关联信息， 并且节点会将这些信息发送给其他节点：

- 节点所使用的 IP 地址和 TCP 端口号。

- 节点的标志（flags）。

- 节点负责处理的哈希槽。

- 节点最近一次使用集群连接发送 PING 数据包（packet）的时间。

- 节点最近一次在回复中接收到 PONG 数据包的时间。

- 集群将该节点标记为下线的时间。

- 该节点的从节点数量。

- 如果该节点是从节点的话，那么它会记录主节点的节点 ID 。 如果这是一个主节点的话，那么主节点 ID 这一栏的值为 0000000 。

以上信息的其中一部分可以通过向集群中的任意节点（主节点或者从节点都可以）发送 CLUSTER NODES 命令来获得。

以下是一个向集群中的主节点发送 CLUSTER NODES 命令的例子， 该集群由三个节点组成：

```bash
$ redis-cli cluster nodes
d1861060fe6a534d42d8a19aeb36600e18785e04 :0 myself - 0 1318428930 connected 0-1364
3886e65cc906bfd9b1f7e7bde468726a052d1dae 127.0.0.1:6380 master - 1318428930 1318428931 connected 1365-2729
d289c575dcbc4bdd2931585fd4339089e461a27d 127.0.0.1:6381 master - 1318428931 1318428931 connected 2730-4095
```

在上面列出的三行信息中， 从左到右的各个域分别是： 节点 ID ， IP 地址和端口号， 标志（flag）， 最后发送 PING 的时间， 最后接收 PONG 的时间， 连接状态， 节点负责处理的槽。

## MOVED 转向

一个 Redis 客户端可以向集群中的任意节点（包括从节点）发送命令请求。 节点会对命令请求进行分析， 如果该命令是集群可以执行的命令， 那么节点会查找这个命令所要处理的键所在的槽。

如果要查找的哈希槽正好就由接收到命令的节点负责处理， 那么节点就直接执行这个命令。

另一方面， 如果所查找的槽不是由该节点处理的话， 节点将查看自身内部所保存的哈希槽到节点 ID 的映射记录， 并向客户端回复一个 MOVED 错误。

## ASK 转向

当节点需要让一个客户端长期地（permanently）将针对某个槽的命令请求发送至另一个节点时， 节点向客户端返回 MOVED 转向。

另一方面， 当节点需要让客户端仅仅在下一个命令请求中转向至另一个节点时， 节点向客户端返回 ASK 转向。

从客户端的角度来看， ASK 转向的完整语义（semantics）如下：

- 如果客户端接收到 ASK 转向， 那么将命令请求的发送对象调整为转向所指定的节点。

- 先发送一个 ASKING 命令，然后再发送真正的命令请求。

- 不必更新客户端所记录的槽 8 至节点的映射： 槽 8 应该仍然映射到节点 A ， 而不是节点 B 。

一旦节点 A 针对槽 8 的迁移工作完成， 节点 A 在再次收到针对槽 8 的命令请求时， 就会向客户端返回 MOVED 转向， 将关于槽 8 的命令请求长期地转向到节点 B 。

## 集群在线重配置（live reconfiguration）

Redis 集群支持在集群运行的过程中添加或者移除节点。

实际上， 节点的添加操作和节点的删除操作可以抽象成同一个操作， 那就是， 将哈希槽从一个节点移动到另一个节点：

- 添加一个新节点到集群， 等于将其他已存在节点的槽移动到一个空白的新节点里面。

- 从集群中移除一个节点， 等于将被移除节点的所有槽移动到集群的其他节点上面去。

因此， 实现 Redis 集群在线重配置的核心就是将槽从一个节点移动到另一个节点的能力。 因为一个哈希槽实际上就是一些键的集合， 所以 Redis 集群在重哈希（rehash）时真正要做的， 就是将一些键从一个节点移动到另一个节点。

要理解 Redis 集群如何将槽从一个节点移动到另一个节点， 我们需要对 CLUSTER 命令的各个子命令进行介绍， 这些命理负责管理集群节点的槽转换表（slots translation table）。

以下是 CLUSTER 命令可用的子命令：

- CLUSTER ADDSLOTS slot1 [slot2] ... [slotN]

- CLUSTER DELSLOTS slot1 [slot2] ... [slotN]

- CLUSTER SETSLOT slot NODE node

- CLUSTER SETSLOT slot MIGRATING node

- CLUSTER SETSLOT slot IMPORTING node

## 容错

### 节点失效检测

以下是节点失效检查的实现方法：

- 当一个节点向另一个节点发送 PING 命令， 但是目标节点未能在给定的时限内返回 PING 命令的回复时， 那么发送命令的节点会将目标节点标记为 PFAIL （possible failure，可能已失效）。等待 PING
  命令回复的时限称为“节点超时时限（node timeout）”， 是一个节点选项（node-wise setting）。

- 每次当节点对其他节点发送 PING 命令的时候， 它都会随机地广播三个它所知道的节点的信息， 这些信息里面的其中一项就是说明节点是否已经被标记为 PFAIL 或者 FAIL 。

- 当节点接收到其他节点发来的信息时， 它会记下那些被其他节点标记为失效的节点。 这称为失效报告（failure report）。

- 如果节点已经将某个节点标记为 PFAIL ， 并且根据节点所收到的失效报告显式， 集群中的大部分其他主节点也认为那个节点进入了失效状态， 那么节点会将那个失效节点的状态标记为 FAIL 。

- 一旦某个节点被标记为 FAIL ， 关于这个节点已失效的信息就会被广播到整个集群， 所有接收到这条信息的节点都会将失效节点标记为 FAIL 。

简单来说， 一个节点要将另一个节点标记为失效， 必须先询问其他节点的意见， 并且得到大部分主节点的同意才行。

因为过期的失效报告会被移除， 所以主节点要将某个节点标记为 FAIL 的话， 必须以最近接收到的失效报告作为根据。

在以下两种情况中， 节点的 FAIL 状态会被移除：

- 如果被标记为 FAIL 的是从节点， 那么当这个节点重新上线时， FAIL 标记就会被移除。保持（retaning）从节点的 FAIL 状态是没有意义的， 因为它不处理任何槽， 一个从节点是否处于 FAIL 状态，
  决定了这个从节点在有需要时能否被提升为主节点。

- 如果一个主节点被打上 FAIL 标记之后， 经过了节点超时时限的四倍时间， 再加上十秒钟之后， 针对这个主节点的槽的故障转移操作仍未完成， 并且这个主节点已经重新上线的话， 那么移除对这个节点的 FAIL 标记。

### 集群状态检测（已部分实现）

每当集群发生配置变化时（可能是哈希槽更新，也可能是某个节点进入失效状态）， 集群中的每个节点都会对它所知道的节点进行扫描（scan）。

一旦配置处理完毕， 集群会进入以下两种状态的其中一种：

- FAIL ： 集群不能正常工作。 当集群中有某个节点进入失效状态时， 集群不能处理任何命令请求， 对于每个命令请求， 集群节点都返回错误回复。

- OK ： 集群可以正常工作， 负责处理全部 16384 个槽的节点中， 没有一个节点被标记为 FAIL 状态。

这说明即使集群中只有一部分哈希槽不能正常使用， 整个集群也会停止处理任何命令。

不过节点从出现问题到被标记为 FAIL 状态的这段时间里， 集群仍然会正常运作， 所以集群在某些时候， 仍然有可能只能处理针对 16384 个槽的其中一个子集的命令请求。

以下是集群进入 FAIL 状态的两种情况：

- 至少有一个哈希槽不可用，因为负责处理这个槽的节点进入了 FAIL 状态。

- 集群中的大部分主节点都进入下线状态。当大部分主节点都进入 PFAIL 状态时，集群也会进入 FAIL 状态。

第二个检查是必须的， 因为要将一个节点从 PFAIL 状态改变为 FAIL 状态， 必须要有大部分主节点进行投票表决， 但是， 当集群中的大部分主节点都进入失效状态时， 单凭一个两个节点是没有办法将一个节点标记为 FAIL 状态的。

### 从节点选举

一旦某个主节点进入 FAIL 状态， 如果这个主节点有一个或多个从节点存在， 那么其中一个从节点会被升级为新的主节点， 而其他从节点则会开始对这个新的主节点进行复制。

新的主节点由已下线主节点属下的所有从节点中自行选举产生， 以下是选举的条件：

- 这个节点是已下线主节点的从节点。

- 已下线主节点负责处理的槽数量非空。

- 从节点的数据被认为是可靠的， 也即是， 主从节点之间的复制连接（replication link）的断线时长不能超过节点超时时限（node timeout）乘以 REDIS_CLUSTER_SLAVE_VALIDITY_MULT
  常量得出的积。

如果一个从节点满足了以上的所有条件， 那么这个从节点将向集群中的其他主节点发送授权请求， 询问它们， 是否允许自己（从节点）升级为新的主节点。

如果发送授权请求的从节点满足以下属性， 那么主节点将向从节点返回 FAILOVER_AUTH_GRANTED 授权， 同意从节点的升级要求：

- 发送授权请求的是一个从节点， 并且它所属的主节点处于 FAIL 状态。

- 在已下线主节点的所有从节点中， 这个从节点的节点 ID 在排序中是最小的。

- 这个从节点处于正常的运行状态： 它没有被标记为 FAIL 状态， 也没有被标记为 PFAIL 状态。

一旦某个从节点在给定的时限内得到大部分主节点的授权， 它就会开始执行以下故障转移操作：

- 通过 PONG 数据包（packet）告知其他节点， 这个节点现在是主节点了。

- 通过 PONG 数据包告知其他节点， 这个节点是一个已升级的从节点（promoted slave）。

- 接管（claiming）所有由已下线主节点负责处理的哈希槽。

- 显式地向所有节点广播一个 PONG 数据包， 加速其他节点识别这个节点的进度， 而不是等待定时的 PING / PONG 数据包。

所有其他节点都会根据新的主节点对配置进行相应的更新，特别地：

- 所有被新的主节点接管的槽会被更新。

- 已下线主节点的所有从节点会察觉到 PROMOTED 标志， 并开始对新的主节点进行复制。

- 如果已下线的主节点重新回到上线状态， 那么它会察觉到 PROMOTED 标志， 并将自身调整为现任主节点的从节点。

在集群的生命周期中， 如果一个带有 PROMOTED 标识的主节点因为某些原因转变成了从节点， 那么该节点将丢失它所带有的 PROMOTED 标识。

