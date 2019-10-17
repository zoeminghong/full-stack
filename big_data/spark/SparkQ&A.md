# Spark QA

##### Spark是什么？

Spark是基于 map reduce 算法模型实现的**分布式计算框架**，拥有**Hadoop MapReduce所具有的优点**，并且解决了Hadoop MapReduce中的诸多缺陷。

##### Spark 优化或解决了 Hadoop 哪些不足？

**减少磁盘I/O：**随着实时大数据应用越来越多，Hadoop作为离线的高吞吐、低响应框架已不能满足这类需求。HadoopMapReduce的map端将中间输出和结果存储在磁盘中，reduce端又需要从磁盘读写中间结果，势必造成磁盘IO成为瓶颈。Spark允许将map端的中间输出和结果存储在内存中，reduce端在拉取中间结果时避免了大量的磁盘I/O。Hadoop Yarn中的ApplicationMaster申请到Container后，具体的任务需要利用NodeManager从HDFS的不同节点下载任务所需的资源（如Jar包），这也增加了磁盘I/O。Spark将应用程序上传的资源文件缓冲到Driver本地文件服务的内存中，当Executor执行任务时直接从Driver的内存中读取，也节省了大量的磁盘I/O。

**增加并行度：**由于将中间结果写到磁盘与从磁盘读取中间结果属于不同的环节，Hadoop将它们简单的通过串行执行衔接起来。Spark把不同的环节抽象为Stage，允许多个Stage既可以串行执行，又可以并行执行。
避免重新计算：当Stage中某个分区的Task执行失败后，会重新对此Stage调度，但在重新调度的时候会过滤已经执行成功的分区任务，所以不会造成重复计算和资源浪费。

**可选的Shuffle排序：**HadoopMapReduce在Shuffle之前有着固定的排序操作，而Spark则可以根据不同场景选择在map端排序或者reduce端排序。

**灵活的内存管理策略：**Spark将内存分为堆上的存储内存、堆外的存储内存、堆上的执行内存、堆外的执行内存4个部分。Spark既提供了执行内存和存储内存之间是固定边界的实现，又提供了执行内存和存储内存之间是“软”边界的实现。Spark默认使用“软”边界的实现，执行内存或存储内存中的任意一方在资源不足时都可以借用另一方的内存，最大限度的提高资源的利用率，减少对资源的浪费。Spark由于对内存使用的偏好，内存资源的多寡和使用率就显得尤为重要，为此Spark的内存管理器提供的Tungsten实现了一种与操作系统的内存Page非常相似的数据结构，用于直接操作操作系统内存，节省了创建的Java对象在堆中占用的内存，使得Spark对内存的使用效率更加接近硬件。Spark会给每个Task分配一个配套的任务内存管理器，对Task粒度的内存进行管理。Task的内存可以被多个内部的消费者消费，任务内存管理器对每个消费者进行Task内存的分配与管理，因此Spark对内存有着更细粒度的管理。

##### 什么是宽依赖、窄依赖？

窄依赖指的是每一个parent RDD的Partition最多被子RDD的一个Partition使用。

宽依赖指的是多个子RDD的Partition会依赖同一个parent RDD的Partition。

**RDD的每个Partition,仅仅依赖于父RDD中的一个Partition,这才是窄。**

##### 窄依赖类型？

OneToOneDependency 与 RangeDependency

https://blog.csdn.net/JavaMoo/article/details/78441208

##### 如何区分 action 与 transformation？

如果返回的是 RDD 类型，那么这是 transformation; 如果返回的是其他数据类型，那么这是 action。

