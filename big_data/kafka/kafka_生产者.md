## 生产者

生产者在 Kafka 中是必须存在的角色。在 Kafka 中 `ReplicatManager` **负责将生产者发送的消息写入到 Leader 副本、管理 Follower 副本与 Leader 副本之间的数据同步以及副本角色之间的转换。**

消息发送流程：

1. 客户端调用 `KafkaProduce.send()` 方法
2. `KafkaApis.handleProducerRequest()` 方法调用 `ReplicaManager.appendMessages()` 方法将消息追加到相应的分区的 Leader 副本中
3. 从 Leader 副本同步数据到各 Follower 副本中
4. 向生产者做出响应

在消息未被分区的所有 Follower 副本从 Leader 副本同步更新完成之前，`ProduceRequest` 的 acks 为 `-1` ，此时，无法进行下一条消息的操作。

`DelayedProduce` 其作用就是在 acks 为 -1 时，延迟回调 responseCallback 向生产者做出相应，直至所有 Follower 副本从 Leader 副本同步更新完成后，才响应生产者。

**猜想**

- 副本数太多会导致生产者的响应时间太长，TPS 下降
- 在写入数据至 Leader 和同步数据至 Follower 的过程中出现异常，会如何处理？DelayedProduce 可能出现 delayMS 超时的情况

## Follower 副本同步

在 Follower 副本进行从 Leader 副本处同步数据的时，会发起 **FetchRequest** 请求。

**FetchRequest** 是由 `KafkaApis.handleFetchRequest()` 方法处理的，在该方法中 `ReplicaManager.fetchMessage()` 方法从相应的分区的 Leader 副本拉取消息。

`ReplicaManager.fetchMessage()` 方法中会创建 **DelayedFetch** 延迟操作，用于处理拉取数据操作。

##### 为什么在拉取数据的时候需要延迟操作呢？

是为了让当前拉取获得足够的消息数据。

## acks

Kafka提供三种消息确认机制（acks），通过属性`request.required.acks`设置。

acks=0：生产者不用等待代理返回确认信息，而连续发送消息

acks=1：默认模式，生产者需要等待Leader副本已成功将消息写入日志文件中。一定程度上降低数据丢失的风险。在Leader副本宕机时，Follower副本没有及时同步数据，之后代理也不会再向宕机的Leader进行数据的获取，数据就出现了丢失

acks=-1：Leader副本和所有Follower列表中副本都完成数据的存储才会想生产者发生确认信息，即同步副本必须大于1，通过`min.insync.replicas`设置，当同步副本不足该配置值时，生产者会抛异常。该种模式会影响生产者的发送数据的速度以及吞吐量。

### 生产者配置说明

| 属性值                             | 默认值           | 描述                                                         |
| ---------------------------------- | ---------------- | ------------------------------------------------------------ |
| Message.send.max.retries           | 3                | 生产者在丢失该消息前进行重试次数                             |
| Retry.backoff.ms                   | 100              | 检测新的Leader是否已选举出来，更新主题的MetaData之前生产者需要等待的时间 |
| Queue.buffering.max.ms             | 1000             | 当到达该时间后消息开始批量发送，若异步模式下，同时配置了`Batch.num.messages`，则达到这两个阀值之一都将开始批量发送消息 |
| Queue.buffering.max.message        | 10000            | 在异步模式下，在生产者必须被阻塞或者数据必须丢失之前，可以缓存到队列中的未发送的最大消息条数，即初始化消息队列的长度 |
| Batch.num.messages                 | 200              | 在异步模式下每次批量发送消息的最大消息数                     |
| Request.timeout.ms                 | 1500             | 当需要acks时，生产者等待代理应答的超时时间，若该时间范围内没有应答，则会发送错误到客户端 |
| Topic.metadata.refresh.interval.ms | 5min             | 生产者定时请求更新主题元数据的时间间隔。若设置为0，则在每个消息发送后都去请求更新数据 |
| Client.id                          | Console.producer | 生产者指定的一个标识字段，在每次请求中包含该字段，用来追踪调用，根据该字段在逻辑上可以确认是哪个应用发出的请求 |
| Queue.enqueue.timeout.ms           | 2147483647       | 该值为0，标识该队列没满是直接入队，满了则立即丢弃，负数表示无条件阻塞且不丢弃，正数表示阻塞达到该值时长后抛出QueueFullException异常 |

