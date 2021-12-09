# kafka 面试题：kafka 为什么会丢消息

## 生产者丢消息

先介绍一下生产者发送消息的一般流程：

1. 生产者是与 leader 直接交互，所以先从集群获取 topic 对应分区的 leader 元数据；

2. 获取到 leader 分区元数据后直接将消息发给过去；

3. Kafka Broker 对应的 leader 分区收到消息后写入文件持久化；

4. Follower 拉取 Leader 消息与 Leader 的数据保持一致；

5. Follower 消息拉取完毕需要给 Leader 回复 ACK 确认消息；

6. Kafka Leader 和 Follower 分区同步完，Leader 分区会给生产者回复 ACK 确认消息。

生产者采用 push 模式将数据发布到 broker，每条消息追加到分区中，顺序写入磁盘。消息写入 Leader 后，Follower 是主动与 Leader 进行同步。

Kafka 通过配置 request.required.acks 属性来确认 Producer 的消息：

- 0：表示不进行消息接收是否成功的确认；不能保证消息是否发送成功，生成环境基本不会用。

- 1：默认值，表示当 Leader 接收成功时确认；只要 Leader 存活就可以保证不丢失，保证了吞吐量。所以默认的 producer 级别是 at least once。

- all：保证 leader 和 follower 不丢，但是如果网络拥塞，没有收到 ACK，会有重复发的问题。

生成者根据不同的配置存在以下可能：

- 如果 acks 配置为 0，发生网络抖动消息丢了，生产者不校验 ACK 自然就不知道丢了。

- 如果 acks 配置为 1 保证 leader 不丢，但是如果 leader 挂了，恰好选了一个没有 ACK 的 follower，那也丢了。

- 如果 acks 配置为 all 保证 leader 和 follower 不丢，但是如果网络拥塞，没有收到 ACK，会有重复发的问题。

## Broker 丢消息

操作系统本身有一层缓存，叫做 Page Cache，当往磁盘文件写入的时候，系统会先将数据流写入缓存中，至于什么时候将缓存的数据写入文件中是由操作系统自行决定。

Kafka 提供了一个参数 producer.type 来控制是不是主动 flush，如果 Kafka 写入到 mmap 之后就立即 flush 然后再返回 Producer 叫同步 (sync)；写入 mmap 之后立即返回
Producer 不调用 flush 叫异步 (async)。

Kafka 通过多分区多副本机制中已经能最大限度保证数据不会丢失，如果数据已经写入系统 cache 中但是还没来得及刷入磁盘，此时突然机器宕机或者掉电那就丢了，当然这种情况很极端。

## 消费者丢失消息

消费者通过 pull 模式主动的去 kafka 集群拉取消息，与 producer 相同的是，消费者在拉取消息的时候也是找 leader 分区去拉取。

消费消息的时候主要分为两个阶段：

1. 标识消息已被消费，commit offset坐标；

2. 处理消息。

先 commit 再处理消息。如果在处理消息的时候异常了，但是 offset 已经提交了，这条消息对于该消费者来说就是丢失了，再也不会消费到了。

先处理消息再 commit。如果在 commit 之前发生异常，下次还会消费到该消息，重复消费的问题可以通过业务保证消息幂等性来解决。

# 思维导图

![消息中间件-面试题-kafka为什么会丢消息-思维导图.png](https://cnymw.github.io/GolangStudy/docs/img/消息中间件-面试题-kafka为什么会丢消息-思维导图.png)
