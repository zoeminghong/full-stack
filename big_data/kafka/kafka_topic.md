# 主题管理

**控制器** 负责主题相关操作。

以下以主题名为 “foo”，3 个分区，2 个副本进行说明。

### 创建主题

- 在 `/brokers/topics` 目录下创建 foo 子节点
- 触发监听 ` /brokers/topics` 目录的 TopicChangeListener 和监听 `/brokers/ids` 目录的 BrokerChangeListener 监听器

在创建 foo 目录完成之后，服务端已经进行正常返回，真正创建主题操作是控制器异步进行。

#### TopicChangeListener

 用于监听 `/brokers/topics` 路径下子节点的变化，当创建一个主题的时，该路径下会创建一个与主题同名的子节点，当该节点发生变化时，就会触发该监听器行为。注意哦，不监听具体主题目录下的变更。TopicChangeListener 触发时会更新代理控制器中的ControllerContext 中维护的主题列表信息及主题对应的 AR 信息。若是新增主题，则会调用控制器的相应方法，为该主题注册一个监听主题分区和副本变化的监听器。

TopicChangeListener 在创建主题的过程中分为更新缓存和真正创建主题过程。

##### 更新缓存

更新 ControllerContext 缓存信息

##### 真正创建主题

分区及副本的状态装换、分区 Leader 分配、分区存储日志文件的创建。

通过状态机为每个新创建的主题向 ZK 注册一个监听分区变化的监听器 PartitionModificationListener，该监听器监听 `/brokers/topics/foo/partitions` 节点的信息变化。

**创建分区**

调用分区状态机的 `handleStateChanges()` 方法，将新增主题的分区状态设置为 NewPartition 状态

调用副本状态机的 `handleStateChange()` 方法，将新增主题的每个分区的副本状态设置为 NewReplica 状态

调用分区状态机的 `handleStateChanges()` 方法，将新增主题的各分区状态设置为 OnlineParition 状态，将各分区 AR 中第一个副本选为该分区的 Leader， 将 AR 作为 ISR，然后创建各分区节点并写入分区的详细元数据信息。本例会在 `/brokers/topics/foo/` 路径下创建分区元数据信息。同时更新 ControllerContext 中缓存的信息。

调用副本状态机将新增主题的每个分区的副本状态设置为 OnlineReplica 状态。

> 以上步骤都会进行代理之间数据的同步，Leader 与 Follower 之间的数据同步

### 删除主题

当配置项存在 `delete.topic.enable=true` 时，Kafka 才被运行执行删除操作。当客户端执行删除 Topic 操作时，在 ZK 的 `/admin/delete_topics` 路径下创建一个与待删除主题同名的节点，而实际的删除操作是异步交由 **控制器** 进行执行。

在控制前实例化的时候创建了一个分区状态机，而分区状态机注册了一个监听 zk 的 `/admin/delete_topic` 子节点变化的 `DeleteTopicsListener` 监听器.

在创建主题时，需要为主题的每个分区分配副本 Leader 之后，才调用回调函数通知客户端。`DelayedCreateTopics` 延迟操作等待该主题的所有分区分配到 Leader 副本或是等待超时后调用回调函数返回给客户端。

若检测到主题当前是在进行优先副本选为分区 Leader 操作或是在进行分区重分配操作时，则将该主题加入到暂时不可删除的主题集合中，等待相应的操作完成之后，在调用 `TopicDeletionManager.resumeDeletionForTopics` 方法将这些主题从不可删除的主题集合中剔除。

删除主题涉及 `/brokers/topics、/config/topics/ 和 /admin/delete_topics` 等节点数据的删除。

**DeleteTopicListenter** 用于监听 `/admin/delete_topic` 子节点的变化，当删除一个主题的时候，会在该目录下创建一个与待删主题同名的子节点。DeleteTopicListenter 触发时会将相应的待删主题目录删除，并将该主题加入到 TopicDeletionManager 维护的待删除记录中，等待 TopicDeletionManager 执行删除操作。

##### 为什么删除与变更主题分开处理？



## 分区副本分配

分区副本可以在创建主题的时候指定分配方案，也可以采用Kafka默认的分区副本分配策略创建副本分配方案。

分配策略通过随机数方式来确定第一个副本的位置，以及第二次轮询相对前一次分配的移位量是为了尽可能的把分区副本均匀分布。实现各代理负载均衡的效果。

客户端创建分区副本方案

```shell
./kafka-topic.sh --create --zookeeper server-1:2181,server-2:2181,server-3:2181 --replication-factor 3 --partitions 6 --topic replica-topic
```

通过在zookeeper中使用如下命令获取分区分配方案

```shell
get /brokers/topics/<topic名>
```

每个分区中的首个副本，被称为优先副本。Kafka会保证优先副本会被均匀分布到集群所有的代理节点上，刚创建的主题一般会选择优先副本作为分区的Leader，这样一个主题的所有分区的Leader被均匀的分布到集群中了。

因而推荐`paritionNum % brokerNum=0`进行配置，同时副本数要小于节点数（brokers）

