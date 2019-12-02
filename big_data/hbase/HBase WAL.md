# HBase WAL

WAL (Write-Ahead-Log) 预写日志是在 HBase RegionServer 中执行 Put、Delete 等操作时，往 HDFS 中追加相应行为的日志信息，其目的是 HBase 中存在 MemStore 这个中间过程，其属于内存方式存储数据，当 RegionServer 宕机时，数据会发生丢失，而 WAL 的行为就是保证该情况发生时，数据不丢失，因而，当 WAL 存储成功之后，才向客户端相应。

## 详述

一个写的请求过来，会根据 RowKey 计算出哪个 RegionServer 进行相应的数据处理，数据通过 WAL 方式将数据存储当前 RegionServer，再通过并行或者串行方式进行数据的同步到其他 RegionServer 下，保证了数据不丢失。

**在一个 RegionServer 上的所有的 Region 都共享一个 HLog** ，一次数据的提交是先写 WAL ，写入成功后，再写 MemStore。

WAL 支持在客户端通过 `setWriteToWAL(false)` 方法关闭日志操作，从而提升写入速度，同时也伴随着数据丢失的风险。

由 WAL 产生的日志文件是 SequenceFile 格式。其 key 是 HLogKey 实例。HLogKey 中记录了写入数据的归属信息，除了 table 和region 名字外，同时还包括 sequence number 和 timestamp。 timestamp 是写入时间，sequence number 的起始值为 0，或者是最近一次存入文件系统中 Sequence number。 

WAL 每次写 Log 日志时，默认会往集群其他 DataNode 节点进行数据同步，以保障其中一台 RegionServer 服务宕机了，不会出现数据的丢失。同步方式支持：**Pipeline 和 n-Way Writes**，也就是串行和并行，只有最后一个节点同步完成，才算同步结束。Pipeline 串行相应的时间上比较慢，n-Way Writes 通过并行度提高的效率，但需要更多的资源消耗，在实际生产中，根据实际情况选择一种方案。

如果被设置每次不同步，则写操作会被 RegionServer 缓存，并启动一个 LogSyncer 线程来定时同步日志，定时时间默认是1秒，也可由 `hbase.regionserver.optionallogflushinterval` 设置。

## Log Split

一个 RegionServer 中只有一个 HLog，该 RegionServer 中的所有 Region 数据都会往该 HLog 进行 Append 操作，导致数据是不连续的。当 RegionServer 宕机之后，对 Region 数据进行还原，需要使用 HLog，因此需要**把 HLog 中的更新按照 region 分组**，这一把 HLog 中更新日志分组的过程就称为 log split。

log split 由 Master 分配不同的任务给不同的 RegionServer 处理，提升速度，相应的操作信息通过 zk 进行记录和分发。

分布式日志分割可以通过配置项 `hbase.master.distributed.log.splitting` 来控制，默认为 true, 即默认情况下分布式日志分割是打开的。

## WAL滚动

WAL是一个环状的滚动日志结构，这样可以保证写入效果最高并且保证空间不会持续变大。
触发滚动的条件：

- WAL的检查间隔：`hbase.regionserver.logroll.period` 。默认一小时，上面说了，通过 sequenceid，把当前WAL的操作和 HDFS 对比，看哪些操作已经被持久化了。就被移动到oldWAL目录中。
- 当 WAL 文件所在的块 block 快要满了
- 当 WAL 所占的空间大于或者等于某个阈值（hbase.regionserver.hlog.blocksize乘hbase.regionserver.logroll.multiplier）blocksize是存储系统的块大小，如果你是基于HDFS只要设定为HDFS的块大小即可，multiplier是一个百分比，默认0.95，即WAL所占的空间大于或者等于95%的块大小，就被归到oldWAL文件中。

## HLog Clean

HLog 文件大小由 `hbase.regionserver.hlog.blocksize` 决定，该值应小于等于 HDFS File Block Size 大小，一般HDFS block 为 64 MB，大于 HDFS block 设定值，会出现日志不清理的问题。

HLog 文件数是可以存在多个的文件，文件个数公式：`(regionserver_heap_size * memstore fraction) / (default_WAL_size)` 。

| **Configuration Property**            | **Description**                      | **Default**                              |
| :------------------------------------ | :----------------------------------- | :--------------------------------------- |
| hbase.regionserver.maxlogs            | Sets the maximum number of WAL files | 32                                       |
| hbase.regionserver.logroll.multiplier | Multiplier of HDFS block size        | 0.95                                     |
| hbase.regionserver.hlog.blocksize     | Optional override of HDFS block size | Value assigned to actual HDFS block size |

`/hbase` 有一个 oldWAL 目录，为了避免恢复的时候因为 HLog 过大导致的效率低下，HLog 过大时就会触发强制刷盘操作。对于已经刷盘的数据，其对应的 HLog 会过期，过期的 HLog 会被移动到 oldWAL。

oldWAL 什么时候被彻底删除呢？Master会定期的去清理这个文件，如果当这个WAL不需要作为用来恢复数据的备份，那么就可以删除。两种情况下，可能会引用WAL文件，此时不能删除
1、TTL进程：该进程会保障WAL文件存活到hbase.master.logcleaner.ttl定义的超时时间为止，默认10分钟。
2、备份机制：如果你开启了备份机制replication（把一个集群的数据实时备份到里另一个集群），那么HBASE要保障备份集群已经完全不需要这个文件了。如果你手头就一个集群，那么就不需要考虑这个文件了

## 配置项

**hbase.regionserver.optionallogflushinterval**
将Hlog同步到HDFS的间隔。如果Hlog没有积累到一定的数量，到了时间，也会触发同步。默认是1秒，单位毫秒。 
默认: 1000 
**hbase.regionserver.logroll.period** 
提交commit log的间隔，不管有没有写足够的值。 
默认: 3600000 （1个小时） 
**hbase.master.logcleaner.ttl** 
Hlog存在于.oldlogdir 文件夹的最长时间, 超过了就会被 Master 的线程清理掉. 
默认: 600000   （10分钟） 
**hbase.master.logcleaner.plugins** 
值用逗号间隔的文本表示。这些WAL/HLog  cleaners会按顺序调用。可以把先调用的放在前面。可以实现自己的LogCleanerDelegat，加到Classpath下，然后在这里写上类的全路径就可以。一般都是加在默认值的前面。 
具体的初始是在CleanerChore 的initCleanerChain方法，此方法同时也实现HFile的cleaner的初台化。 
默认: org.apache.hadoop.hbase.master.TimeToLiveLogCleaner 

**hbase.regionserver.hlog.blocksize** 
**hbase.regionserver.maxlogs** 

WAL的最大值由`hbase.regionserver.maxlogs * hbase.regionserver.hlog.blocksize` (2GB by default)决定。一旦达到这个值，Memstore flush就会被触发。通过WAL限制来触发Memstore的flush并非最佳方式，这样做可能会会一次flush很多Region，引发flush雪崩。 

最好将hbase.regionserver.hlog.blocksize * hbase.regionserver.maxlogs 设置为稍微大于hbase.regionserver.global.memstore.lowerLimit * HBASE_HEAPSIZE. 

**Configure the size and number of WAL files**

[HDP Suggestion](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/hbase-data-access/content/configure-the-size-and-number-wal-files.html)

## 问题

这里的 Log 日志同步行为与 HDFS 备份行为是否是同一个行为？