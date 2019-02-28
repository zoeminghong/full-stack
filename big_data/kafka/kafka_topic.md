在创建主题时，需要为主题的每个分区分配副本 Leader 之后，才调用回调函数通知客户端。`DelayedCreateTopics` 延迟操作等待该主题的所有分区分配到 Leader 副本或是等待超时后调用回调函数返回给客户端。

**TopicChangeListener** 用于监听 `/brokers/topics` 路径下子节点的变化，当创建一个主题的时，该路径下会创建一个与主题同名的子节点，当该节点发生变化时，就会触发该监听器行为。注意哦，不监听具体主题目录下的变更。TopicChangeListener 触发时会更新代理控制器中的ControllerContext 中维护的主题列表信息及主题对应的 AR 信息。若是新增主题，则会调用控制器的相应方法，为该主题注册一个监听主题分区和副本变化的监听器。

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

