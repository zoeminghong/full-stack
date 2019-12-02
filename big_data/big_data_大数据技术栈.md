# 大数据技术栈

## Accumulo

Accumulo是一款开源分布式NoSQL数据库,基于谷歌的BigTable构建而成。其能够非常高效地对超大规模数据集(通常即指大数据)执行CRUD(即创建、读取、更新与删除)操作。相较于其它类似的分布式数据库选项(例如HBase或者CouchDB)，Accumulo的优势在于能够立足单元级访问控制层面提供细粒度安全性控制。

Accumulo构建于其它Apache软件的基础之上。Accumulo以键-值对形式表现其数据，并将这些数据作为文件存储在HDFS上。其同时利用 ZooKeeper对不同进程之间的设置进行同步。

## HBase

HBase是一种分布式面向列式存储的数据库。以 Key-Value 方式进行数据存储，运行于 HDFS 之上，与 Phoenix 相结合可以支持二级索引和 SQL 化查询功能。

## Apache Kafka

kafka是一种高吞吐量的分布式发布订阅消息系统，她有如下特性：

通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。

- 高吞吐量：即使是非常普通的硬件kafka也可以支持每秒数十万的消息。
- 支持通过kafka服务器和消费机集群来分区消息。
- 支持Hadoop并行数据加载。

## Apache Kudu

Kudu是一个针对Apache Hadoop平台而开发的列式存储管理器。Kudu共享Hadoop生态系统应用的常见技术特性：它在商品硬件上运行，支持水平可扩展及高可用操作。Kudu 适合实时分析的场景， 此外，Kudu还有更多优化的特点：

- OLAP工作的快速处理。
- 不依赖 Hadoop 环境
- 与 Apache Impala（incubating）紧密集成，使其与 Apache Parquet 一起使用 HDFS 成为一个很好的可变的替代方案。
- 强大而灵活的一致性模型，允许您根据每个 per-request（请求选择）一致性要求，包括 strict-serializable（严格可序列化）一致性的选项。
- 针对同时运行顺序和随机工作负载的情况性能很好。
- 使用 Cloudera Manager 轻松维护和管理。
- High availability（高可用性）。
- 结构化数据模型。
- 支持 Select / Update / Delete 操作，批量操作性能比较好。
- hash 分区适合大量写操作，range 分区适合大量读操作。
- Kudu可以实时存储，保存所有历史数据，并提供低延迟随机查询与批量扫描数据分析。
- Kudu 其实是 HBase 与 HDFS 的中间产物，牺牲写的性能去提高批量扫描数据的能力。
- 加持 Impala 实现 SQL 化查询。
- 支持结构化数据，预定义表结构
- 有限支持事务

[拓展阅读](https://cloud.tencent.com/developer/article/1408441)

[拓展阅读](https://www.cnblogs.com/zlslch/p/7607353.html)

[拓展阅读](https://kakack.github.io/2016/12/Apache-Kudu%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%80%9D%E6%83%B3-%E6%9E%B6%E6%9E%84%E4%B8%8EImpala%E5%AE%9E%E8%B7%B5/)

[Apache Kudu 1.4.0 中文文档](https://www.bookstack.cn/books/kudu-1.4)

## Apache Phoenix

Apache Phoenix支持在Hadoop平台上的OLTP与运营分析工作，对于低延时的应用，它结合了以下优势：

- 支持标准的SQL与JDBC APIs，完成ACID事物操作；
- 可以集成HBase，Spark，Hive，Pig，Flume及MapsReduce；

## Apache Oozie

Apache Oozie是一个工作流调用系统，用于管理Apache Hadoop任务。Oozie工作流作业是DAGs动作。Oozie与其他Hadoop栈集成，支持Hadoop各种类型作业，包括Java map-reduce,Streaming map-reduce,Pig,Hive,Sqoop与Distcp，以及系统具体的作业。 Oozie是一个稳定的，可靠的，可扩展的系统。

## Apache Flink

Apache Flink 是高效和分布式的通用数据处理平台。 Apache Flink 声明式的数据分析开源系统，结合了分布式 MapReduce 类平台的高效，灵活的编程和扩展性。同时在并行数据库发现查询优化方案。 Apache Flink是一个框架，用于流计算处分布式处理引擎。可以处理有界与无界的数据流。 1.无界数据：指的是数据流事件有开始，但没有明确定义结束的事件，无界数据流必须处理； 2.有界数据：指的是数据流有明确定义开始和结束，将要执行的计算操作需要在所有数据被注入后执行。

应用部署

Apache Flink作为一个分布式系统，所有操作基于内存处理，所以应用执行要求计算资源。Apache Flink可以与常见的集群集成，例如Hadoop YARN，Apache Mesos与Kubernetes，当然可以支持作为独立的集群进行部署。

## Apache Avro

Avro（读音类似于[ævrə]）是Hadoop的一个子项目，由Hadoop的 创始人Doug Cutting（也是Lucene，Nutch等项目的创始人）牵头开发。Avro是一个数据序列化系统，设计用于支持大 批量数据交换的应用。它的主要特点有：支持二进制序列化方式，可以便捷，快速地处理大量数据；动态语言友好，Avro提供的机制使动态语言可以方便地处理 Avro数据。

Hive

## [ClickHouse](http://clickhouse.com.cn/topic/5bd674449d28dfde2ddc63c0)

ClickHouse 由俄罗斯 Yandex 公司开发。专为在线数据分析而设计。Yandex 是俄罗斯搜索引擎公司。官方提供的文档表名，ClickHouse 日处理记录数”十亿级”。

特性：

- 采用列式存储
- 数据压缩
- 基于磁盘的存储，大部分列式存储数据库为了追求速度，会将数据直接写入内存，按时内存的空间往往很小
- CPU 利用率高，在计算时会使用机器上的所有 CPU 资源
- 支持分片，并且同一个计算任务会在不同分片上并行执行，计算完成后会将结果汇总
- 支持 SQL，SQL 几乎成了大数据的标准工具，使用门槛较低，不支持特殊的子查询和窗口函数。
- 支持联表查询
- 支持实时更新
- 自动多副本同步
- 支持索引
- 分布式存储查询
- 不支持事物。
- 不支持 Update/Delete 操作

在业界被广泛应用于 APP 分析，比如漏斗，留存。但是在我们的测试的中，当机器数量比较少时 ( <10 台 )，耗时依然在 10 秒以上。

[参考阅读](https://mp.weixin.qq.com/s/4WCgzvTjOvNONrrZLc7yOQ)

## Impala

高性能实时数据分析工具。impala 侧重于实时性要求高和计算复杂度低的场景，hive 对时效性比较低和需要支持计算复杂度高的场景。

特性：

- 处理速度比 Hive 快
- 依赖 Hive 进行一些数据的存储
- 查询速度快
- 支持 HBASE、Hive、Kudu 数据格式
- 支持 SQL
- 不支持UDF
- 最大使用内存，中间结果不写磁盘，及时通过网络以stream的方式传递。

[拓展阅读](https://blog.csdn.net/javajxz008/article/details/50523332)

## StreamSets

Streamsets是一款大数据实时采集和ETL工具，可以实现不写一行代码完成数据的采集和流转。通过拖拽式的可视化界面，实现数据管道(Pipelines)的设计和定时任务调度。最大的特点有：

常见的Origins有Kafka、HTTP、UDP、JDBC、HDFS等；Processors可以实现对每个字段的过滤、更改、编码、聚合等操作；Destinations跟Origins差不多，可以写入Kafka、Flume、JDBC、HDFS、Redis等。

- Field Master 给指定字符串字段加密，可以选择Mast的类型
- Stream Selector 对数据流做分流处理
- Field Merger 支持数据合并，但只支持map、List结构的数据
- Aggregator 在一段时间间隔内做聚合指标，比如sum、avg、max、min等
- Delay 延迟一段时间
- Field Flattener 拆分map、List结构的数据成没有嵌套的数据结构
- Field Hasher 对指定字段进行encode,策略
- 可视化界面操作，不写代码完成数据的采集和流转
- 内置监控，可是实时查看数据流传输的基本信息和数据的质量
- 强大的整合力，对现有常用组件全力支持，包括50种数据源、44种数据操作、46种目的地
- 对于Streamsets来说，最重要的概念就是数据源(Origins)、操作(Processors)、目的地(Destinations)。创建一个Pipelines管道配置也基本是这三个方面

## Presto

**Presto**是一种用于大数据的高性能分布式[SQL](https://zh.wikipedia.org/wiki/SQL)查询引擎。其架构允许用户查询各种数据源，如Hadoop、AWS S3、Alluxio、[MySQL](https://zh.wikipedia.org/wiki/MySQL)、[Cassandra](https://zh.wikipedia.org/wiki/Cassandra)、Kafka和MongoDB。甚至可以在单个查询中查询来自多个数据源的数据。Presto是[Apache许可证](https://zh.wikipedia.org/wiki/Apache许可证)下发布的社区驱动的开源软件。

最初的需求是解决 Facebook 的 Apache Hive 在他们 PB 级的数据仓库上运行 SQL 分析。Hive不适合Facebook的规模，而Presto是为了填补快速查询这块的差距而发明的。

Presto的架构非常类似于使用集群计算（MPP）的传统[数据库管理系统](https://zh.wikipedia.org/wiki/数据库管理系统)。它可以视为一个协调器节点，与多个工作节点同步工作。客户端提交已解析和计划的SQL语句，然后将并行任务安排给工作机。工作机一同处理来自数据源的行并生成返回给客户端的结果。与在每个查询上使用Hadoop的[MapReduce](https://zh.wikipedia.org/wiki/MapReduce)机制的原始Apache Hive执行模型相比，Presto不会将中间结果写入磁盘，从而显着提高速度。Presto是用[Java语言](https://zh.wikipedia.org/wiki/Java)编写的。单个Presto查询可以组合来自多个源的数据。**Presto提供数据源的连接器，包括Alluxio、Hadoop分布式文件系统、Amazon S3中的文件、[MySQL](https://zh.wikipedia.org/wiki/MySQL)、[PostgreSQL](https://zh.wikipedia.org/wiki/PostgreSQL)、[Microsoft SQL Server](https://zh.wikipedia.org/wiki/Microsoft_SQL_Server)、Amazon Redshift、Apache Kudu、Apache Phoenix、Apache Kafka、[Apache Cassandra](https://zh.wikipedia.org/wiki/Cassandra)、Apache Accumulo、MongoDB和Redis。**与其他只支持Hadoop特定发行版的工具（如Cloudera Impala）不同，Presto可以使用任何风格的Hadoop，也可以不用Hadoop。Presto支持计算和存储的分离，可以在本地和[云中](https://zh.wikipedia.org/wiki/雲端運算)部署。

与 Impala 性能比较方面，Impala 胜出，Impala 性能还是蛮明显的，但数据源方面，Presto 支持更多的数据源，但两者都不支持 HBase。

[拓展阅读](https://blog.csdn.net/u012551524/article/details/79124532)



