## 参数说明

**wait_for_active_shards**

在5.0.0版本新引入的一个参数，表示等待活跃的分片数。作用跟consistency类似，可以设置成all或者任意正整数。

比如在这种场景下：集群中有3个节点node-1、node-2和node-3，并且索引中的分片需要复制3份。那么该索引一共拥有4个分片，包括1个主分片和3个复制分片。

默认情况下，索引操作只需要等待主分片可用(wait_for_active_shards为1)即可。

如果node-2和node-3节点挂了，索引操作是不会受影响的(wait_for_active_shards默认为1)；如果设置了wait_for_active_shards为3，那么需要3个节点全部存活；如果设置了wait_for_active_shards为4或者all(一共4个分片，4和all是一样的效果)，那么该集群中的索引操作永远都会失败，因为集群一共就3个节点，不能处理所有的4个分片。

