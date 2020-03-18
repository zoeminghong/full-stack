# Kylin 实时流处理技术探秘

有了解过 Kylin 的都知道其主要是在离线方面的处理，在实时方面的处理大家知之甚少，我希望把自己最近学到的分享给大家。

社区其实在 1.6 版本中已经提供了近实时的方案，其存在分钟级别的准备时间，在对实时要求比较迫切的场景，这种是不能容忍的，于此同时其实现方式是通过每一个批次数据创建一个 segment，一个 segment 对应一个 HBase Table，长期以往会导致大量的 HBase Table 存在和 MR Job 数据。基于以上的原因，Kylin 考虑实时流处理的技术研究与实现。

## 架构解析

![Kylin 实时流处理技术探秘](https://tva1.sinaimg.cn/large/00831rSTly1gcrdhitsskj30sg0fn435.jpg)

![Kylin 实时流处理技术探秘](https://tva1.sinaimg.cn/large/00831rSTly1gcrdhtl7nuj30sg0fjgq1.jpg)

实时流处理主要由 Streaming Receivers、Streaming Coordinator、Query Engine、Build Engine、Metadata Store 构成。

**Streaming Receivers：**用于消费 Kafka topic 中某个 partition 数据，可以由一个或多个 Streaming Receivers 消费同一个 partition，以保证 HA，Streaming Receivers 以集群的形式存在；

**Streaming Coordinator：**作为 Streaming Receivers 的协调器。指定 Streaming Receivers 负责消费 topic 数据；

**Query Engine：**数据查询引擎。提供数据查询能力。

**Build Engine：**构建引擎。负责将数据提交到历史数据中。

**Metadata Store：**元数据存储。存储一些元数据信息，例如 Receivers 分配信息、高可用信息等。

### 数据架构

![Kylin 实时流处理技术探秘](https://tva1.sinaimg.cn/large/00831rSTly1gcrdi3jckqj30sg0f9agp.jpg)

从图中可以看出，数据从 Kafka 这类消息组件中流入内存中，当数据达到指定阈值或者一个固定时间（默认是一个小时）之后，会将内存数据刷写到硬盘中，过一段时间后会将磁盘数据，通过 MapReduce 任务，将数据同步到 HBase。

因而，如果要进行数据查询，需要聚合内存、硬盘和 HBase 三方的数据。

### 存储流程

实时数据会消费到 Streaming Receivers，通过 Streaming Coordinator 指定 Receivers 消费与 Cube 相关的 Topic 中 Partition 的数据，Receivers 会做 Base Cuboid 的构建，另外可以支持创建一些常用的 Cuboid，以提高查询的性能。过一段时间之后，会将数据从本地数据刷写到 HDFS 上，并通知 Coordinator ，待全部的 Replica Set 把 Cube Segment 的所有实时数据上传到 HDFS 后，Coordinator 触发 MapReduce Job 进行一个批量的构建。Job 会从 HDFS 中拉取实时数据进行构建，做一些合并工作并将 Cuboid 同步到 HBase 中，当同步完成之后，Coordinator 会通知实时集群将相应的数据删除。

### 数据查询

当 QueryServer 接受新查询后，会请求 Coordinator 查询的 Cube 是不是实时 Cube，会根据请求是否是实时 Cube 类型，进行分别处理。离线 Cube 会直接查询 HBase 中数据，实时 Cube 请求若数据不在实时数据中，就发 RPC 请求到 HBase，并且同时发查询请求到实时集群，将结果汇总到查询引擎做一个聚合，再返回给用户。

> 无论实时Cube 还是非实时 Cube 请求都会要通过Coordinator，QueryServer 是作为一个统一的请求入口，之后才是根据实时和非实时两种场景进行不同的处理。

## 实现细节

###  Fragment File 存储格式

在上文中已经提到内存数据会刷写到硬盘中， 而 Fragment File 就是相应持久化的文件模式。

![Kylin 实时流处理技术探秘](https://tva1.sinaimg.cn/large/00831rSTly1gcrdi7k9cpj30sg0g4qaw.jpg)

Segmnet Name 由起始时间和结束时间组成。每一次增加 Fragment 文件都会生成一个 Fragment ID，这是一个递增的值。

Fragment 文件结构是一个列式结构，包括两个文件，Fragment 的数据文件，和 Metadata 文件。数据文件可以包含多个 Cuboid 的数据，**默认只会构建一个 Base cuboid**，如果有配置其它 mandatory cuboid 的话，会根据配置生成多个 Cuboid；这里的数据是多个 Cuboid 依次来保存的，每一个 Cuboid 内是以列式存储，相同列的数据存在一起。基本上现在的 OLAP 存储为了性能通常都是列式存储。每一个维度的数据包括这三部分：

- 第一部分是 Dictionary，是对维度值做字典的。
- 第二部分是值，经过编码的。
- 第三部分是倒排索引。

Metadata 文件里面存有重要的元数据，例如一些 Offset，包含这个维度的数据是从哪个位置开始是这个数据，数据长度是多少，Index，也就是反向索引的长度是多少等等，方便以后查询的时候比较快的定位到。元数据还包含一些压缩信息，指定了数据文件是用什么样的方法进行压缩的。

### 压缩

- 像时间相关的维度，它们的数据基本上都是类似的，或者是递增的。还有设计 Cube 的时候也有设计 Row Key，在 Row Key 的顺序排在第一位的，使用 **run length** 压缩效率会比较高，读取的时候效率也会比较高。
- 对其他的数据默认都会用 **LZ4** 的压缩方式，虽然其它压缩算法的压缩率可能比 LZ4 高，但是 LZ4 解压性能更好，我们选择 LZ4 是主要从查询方面去考虑的，所以从其他角度考虑可能会有一些其它结论。

### 高可用

高可用通过引入 Replica Set 概念来实现。一个 Replica Set 可以包含多个 Receiver，一个 Replica Set 的所有的 Receiver 是共享 Assignment 数据的，Replica Set 下面的 Receiver 消费相同的数据。一个 Replica Set 中存在**一个 Leader 做额外的工作，这个工作，是指把这些实时的数据存到 HDFS**，Leader 选举是用 Zookeeper 来做的。以上是实时集群如何实现 HA 的，可以防止宕掉了对查询和构建造成影响。

### check point

check point 作为错误恢复、保证在服务重启的时候不会出现数据丢失的情况发生，其主要有两种方式来保证，一种是Local Check Point，**以本地存储方式**，每5分钟在本地做一个 check point，把消息的信息存到一个文件里，信息包含消费的 offset 信息，本地磁盘信息，例如最大的 Fragments ID 是多少；重新启动的时候根据这个去恢复。第二种是 Remote Check Point 方式，把一些消费状态信息**存在 HBase 的 Segment 里面**，保存历史的 Segment 信息的时候，会把这些消费信息**存在 Segment 的元数据**里面，构建这个 Segment 的时候，最早是消费到哪个数据的，信息存在那里。

## 小结测试题

**1、Kylin 实时流数据涉及哪些位置？**

本地内存、HDFS（硬盘）、HBase 三方面。

**2、简要说说 Streaming Coordinator 作用有哪些？**

1. 协调 Consumers 消费 Partition 数据
2. 通知 Build Engine 启动 MR Job 处理 HDFS 上存储的实时数据
3. 当实时数据存储到 HBase 之后，通知 Build Engine 删除 HDFS 上数据
4. 查询的时候，协调不同的查询类型，进行不同的处理逻辑

**3、在进行构建的时候，Cuboid 方面是怎么处理的？**

实时数据默认会被构建成 Base Cuboid，但如果存在其他的 mandatory cuboid，也是支持进行配置。

**4、为什么使用 LZ4 压缩格式？**

为了查询效率比较高，进行了相应的妥协。

**5、Fragment File 存有哪些信息？**

Cube 名称、根据时间划分的 Segmnet Name、主要文件为 `.data` 和 `.meta` ，在data文件中存有倒排索引、经过编码后的值、字典，meta 文件存有元数据相关信息。

原文地址：https://www.infoq.cn/article/AafpvXOrZcYWUm-kIkVi