# Parition 分区

Parition 的信息存放在哪里的？

BrokerChangeListener 监听 `/brokers/ids` 路径下各个代理的变换。若出现代理宕机下线、新增代理都会触发该监听器，从而调用控制器的 `ControllerContext.controllerChannelManger` 对节点变化做出响应。

可以通过 `/brokers/topics/${topic}/paritions/${paritionId}/state` 查看分区信息。

## 选举

由于每个分区 Leader 负责分区的相关操作，在 Kafka 中分区 Leader 需要尽可能的均衡分布在各个代理中，以达到性能的负载均衡。

当分区状态发生变化时，PartitionLeaderSelector 分区选举请进行分区 Leader 的选举。Kafka 提供 5 种分区选举器。Kafka 会根据不同的分区的状态变化，选用不同的选举器。

### 选举器