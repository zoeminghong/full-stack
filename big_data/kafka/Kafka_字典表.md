## Kafka字典表

### ZooKeeper 路径表

| 路径                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| /controller_epoch                 | 控制器记录轮值次数                                           |
| /admin/reassign_paritions         | 引发分群重分配操作，值发生变更会触发`PartitionsReassignedListener` 监听器 |
| /admin/preferred_replica_election | 触发将优先副本选为分区 Leader 的 `PreferredReplicaElectionListener` 监听器 |
| /admin/delete_topics              | 记录待删除主题                                               |
| /isr_change_notification          | 记录ISR发生变更，触发`ISRChangeNotificationListener` 监听器  |
| /brokers/topics                   | 记录主题信息                                                 |
| /brokers/ids                      | 记录代理信息                                                 |
| /brokers/seqid                    | 辅助生成代理的id，当用户没有配置 broker.id 时，zk 会自动生成一个全局唯一的 id，每次自动生成是会从该路由读取当前代理的 id 最大值，然后加 1 |
| /consumers                        | 该节点下创建消费组节点                                       |
| /config/topics                    | 存储动态修改主题级别的配置信息                               |
| /config/clients                   | 存储动态修改客户端级别的配置信息                             |
| /config/changes                   | 动态改变配置时存储相应的信息                                 |
| /cluster/id                       | 保存集群 id 信息                                             |
| /controller                       | 保存控制器对应的 brokerId 信息                               |

### 配置表

| 配置                                        | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| auto.leader.rebalance.enable=true           | 是否进行分区平衡操作的定时任务                               |
| leader.imbalance.check.interval.seconds=300 | 多久执行一次分区平衡操作，与 `auto.leader.rebalance.enable` 连用 |
|                                             |                                                              |
|                                             |                                                              |
|                                             |                                                              |
|                                             |                                                              |
|                                             |                                                              |
|                                             |                                                              |
|                                             |                                                              |



### 监听器

| 监听器名                         | 说明                                                      |
| -------------------------------- | --------------------------------------------------------- |
| PartitionsReassignedListener     |                                                           |
| PreferredReplicaElectionListener |                                                           |
| ISRChangeNotificationListener    |                                                           |
| TopicChangeListener              | 监听 `broker/topics` 的变化，从而进行相应操作             |
| DeleteTopicListener              | 监听 `admin/delete_topics` 完成服务器端删除主题相应的操作 |
| BrokerChangeListener             | 监听 `/brokers/ids` 对代理的增、减做出响应                |
| PartitionModificationListener    | 监听主题的分区变化                                        |
|                                  |                                                           |
|                                  |                                                           |



### 