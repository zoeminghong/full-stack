# TiDB

![tidb-architecture](assets/tidb-architecture.png)

## TiDB 简介

- 高度兼容 MySQL
- 水平弹性扩展
- 分布式事务
- 真正金融级高可用
- 一站式 HTAP 解决方案
- 云原生 SQL 数据库



TiDB

TiDB 是无状态的，推荐至少部署两个实例，前端通过负载均衡组件对外提供服务。当单个实例失效时，**会影响正在这个实例上进行的 Session，从应用的角度看，会出现单次请求失败的情况**，重新连接后即可继续获得服务。单个实例失效后，可以重启这个实例或者部署一个新的实例。

PD

PD 是一个集群，通过 Raft 协议保持数据的一致性，单个实例失效时，如果这个实例不是 Raft 的 leader，那么服务完全不受影响；如果这个实例是 Raft 的 leader，会重新选出新的 Raft leader，自动恢复服务。**PD 在选举的过程中无法对外提供服务，这个时间大约是3秒钟**。推荐至少部署三个 PD 实例，单个实例失效后，重启这个实例或者添加新的实例。

TiKV

TiKV 是一个集群，通过 Raft 协议保持数据的一致性（副本数量可配置，默认保存三副本），并通过 PD 做负载均衡调度。单个节点失效时，会影响这个节点上存储的所有 Region。对于 Region 中的 Leader 结点，会中断服务，等待重新选举；对于 Region 中的 Follower 节点，不会影响服务。**当某个 TiKV 节点失效，并且在一段时间内（默认 30 分钟）无法恢复**，PD 会将其上的数据迁移到其他的 TiKV 节点上。

## 功能

- 兼容 MYSQL 语法
- 支持读取历史版本数据，`system variable: tidb_snapshot`

### GC机制

TiDB 的事务的实现采用了 MVCC（多版本并发控制）机制，当新写入的数据覆盖旧的数据时，旧的数据不会被替换掉，而是与新写入的数据同时保留，并以时间戳来区分版本。这点跟 HBase 非常相像，TiDB 触发 GC 的时候进行垃圾数据的清理，默认是 10 min一次。

#### Resolve Locks

TiDB 的事务是基于 [Google Percolator](https://ai.google/research/pubs/pub36726) 模型实现的，事务的提交是一个两阶段提交的过程。第一阶段完成时，所有涉及的 key 会加上一个锁，其中一个锁会被设定为 Primary，其余的锁（Secondary）则会指向 Primary；第二阶段会将 Primary 锁所在的 key 加上一个 Write 记录，并去除锁。这里的 Write 记录就是历史上对该 key 进行写入或删除，或者该 key 上发生事务回滚的记录。Primary 锁被替换为何种 Write 记录标志着该事务提交成功与否。接下来，所有 Secondary 锁也会被依次替换。如果替换这些 Secondary 锁的线程死掉了，锁就残留了下来。

Resolve Locks 这一步的任务即对 safe point 之前的锁进行回滚或提交，**取决于其 Primary 是否被提交**。如果一个 Primary 锁也残留了下来，那么该事务应当视为超时并进行回滚。这一步是必不可少的，因为如果其 Primary 的 Write 记录由于太老而被 GC 清除掉了，那么就再也无法知道该事务是否成功。如果该事务存在残留的 Secondary 锁，那么也无法知道它应当被回滚还是提交，也就无法保证一致性。

Resolve Locks 的执行方式是由 GC leader 对所有的 Region 发送请求进行处理。从 3.0 起，这个过程默认会并行地执行，并发数量默认与 TiKV 节点个数相同。

#### Delete Ranges

在执行 `DROP TABLE/INDEX` 等操作时，会有大量连续的数据被删除。如果对每个 key 都进行删除操作、再对每个 key 进行 GC 的话，那么执行效率和空间回收速度都可能非常的低下。事实上，这种时候 TiDB 并不会对每个 key 进行删除操作，而是**将这些待删除的区间及删除操作的时间戳记录下来。Delete Ranges 会将这些时间戳在 safe point 之前的区间进行快速的物理删除。**

#### Do GC

这一步即删除所有 key 的过期版本。为了保证 safe point 之后的任何时间戳都具有一致的快照，这一步删除 safe point 之前提交的数据，但是会保留 safe point 前的最后一次写入（除非最后一次写入是删除）。

TiDB 2.1 及更早版本使用的 GC 方式是由 GC leader 向所有 Region 发送 GC 请求。从 3.0 起，GC leader 只需将 safe point 上传至 PD。每个 TiKV 节点都会各自从 PD 获取 safe point。当 TiKV 发现 safe point 发生更新时，便会对当前节点上所有作为 leader 的 Region 进行 GC。与此同时，GC leader 可以继续触发下一轮 GC。

[GC 配置](https://pingcap.com/docs-cn/v3.0/reference/garbage-collection/configuration/)

### TiFlash

https://www.chainnews.com/articles/268894218963.htm

## 问题点

- TiDB 的水平拓展是怎么实现的，如果新增一台机器，性能的提升曲线是怎样的
- [distsql](https://pingcap.com/docs-cn/v3.0/reference/garbage-collection/configuration/) 什么用
- 历史数据保留策略时，binlog情况
- Confluence 是否支持同步