Region 应该被理解为分片的概念进行理解会比较好。

参考文章：

http://bigdata-star.com/archives/1197

## Region的寻址

Region的寻址就是怎么去找到Region在哪个RegionServer上。
大数据不管什么框架都存在一个元信息管理，**在HBASE中元信息管理是meta表**。
而之前讲架构的时候说过，**ZK是元信息的入口**。
我们首先应该知道，META表在哪里？因此第一步：Client请求ZK获取META所在的RegionServer的地址。

```
[zk: localhost:2181(CONNECTED) 0] get /hbase/meta-region-server
regionserver:60020
master
```

由此我们得到:META元信息表在master机器上。知道了META表在哪，就可以去查找对应的元信息了。
第二步：去master机器上查找META表，获得我们需要的信息。

```
hbase(main):010:0> scan 'hbase:meta'
ROW                                                   COLUMN+CELL                             
DMP:dop_fdn_v2_cpl_orders,4999999999999998,1560998224746.ed0e7236291bfdca93a77aa4c506568
 5.  column=info:serverstartcode, timestamp=1562808909123, value=1562808826219
                                                                                     
```

META表告诉了我们：你要的的那个 rowkey 是属于哪个Region范围内？这个Region是在哪个RegionServer上？
第三步：去对应的RegionServer上，读取你需要的数据。
第四步：客户端会把meta信息缓存起来，下次操作就不需要去找hbase:meta了。

## 分裂

直接参见：http://hbasefly.com/2017/08/27/hbase-split/?pwfsbk=l13b81

步骤：

1. 第一件事是寻找切分点－splitpoint （整个region中最大store中的最大文件中最中心的一个block的首个rowkey）

2. HBase将整个切分过程包装成了一个事务，意图能够保证切分事务的原子性。整个分裂事务过程分为三个阶段：prepare – execute – (rollback) 
   1. prepare阶段：在内存中初始化两个子region，具体是生成两个HRegionInfo对象，包含tableName、regionName、startkey、endkey等。同时会生成一个transaction journal，这个对象用来记录切分的进展，具体见rollback阶段。
   2. execute阶段
      1. regionserver 更改ZK节点 /region-in-transition 中该region的状态为SPLITING。
      2. master通过watch节点/region-in-transition检测到region状态改变，并修改内存中region的状态，在master页面RIT模块就可以看到region执行split的状态信息。
      3. 在父存储目录下新建临时文件夹.split保存split后的daughter region信息。
      4. 关闭parent region：parent region关闭数据写入并触发flush操作，将写入region的数据全部持久化到磁盘。此后短时间内客户端落在父region上的请求都会抛出异常NotServingRegionException。
      5. 核心分裂步骤：在.split文件夹下新建两个子文件夹，称之为daughter A、daughter B，并在文件夹中生成reference文件，分别指向父region中对应文件。这个步骤是所有步骤中最核心的一个环节。
      6. 父region分裂为两个子region后，将daughter A、daughter B拷贝到HBase根目录下，形成两个新的region
      7. parent region通知修改 hbase.meta 表后下线，不再提供服务。下线后parent region在meta表中的信息并不会马上删除，而是标注split列、offline列为true，并记录两个子region。
      8. 开启daughter A、daughter B两个子region。通知修改 hbase.meta 表，正式对外提供服务。
      9. rollback阶段：如果execute阶段出现异常，则执行rollback操作。为了实现回滚，整个切分过程被分为很多子阶段，回滚程序会根据当前进展到哪个子阶段清理对应的垃圾数据。



## 合并

MemStore 每次 Flush 时都会创建一个新 HFile 文件，当HFile变多，回到值读取数据磁头寻址缓慢，因为HFile都分散在不同的位置，为了防止寻址动作过多，数据读取效率相应降低，适当的减少碎片文件，就需要合并HFile。

### HFile 合并触发条件

**1.Memstore Flush:**

应该说compaction操作的源头就来自flush操作，memstore flush会产生HFile文件，文件越来越多就需要compact。因此在每次执行完Flush操作之后，都会对当前Store中的文件数进行判断，一旦文件数大于配置，就会触发compaction。需要说明的是，compaction都是以Store为单位进行的，而在Flush触发条件下，整个Region的所有Store都会执行compact，所以会在短时间内执行多次compaction。

**2.后台线程周期性检查：**

后台线程定期触发检查是否需要执行compaction，检查周期可配置。线程先检查文件数是否大于配置，一旦大于就会触发compaction。如果不满足，它会接着检查是否满足major compaction条件，简单来说，如果当前store中hfile的最早更新时间早于某个值mcTime，就会触发major compaction（默认7天触发一次，可配置手动触发），HBase预想通过这种机制定期删除过期数据。

**3.手动触发：**

一般来讲，手动触发compaction通常是为了执行major compaction，一般有这些情况需要手动触发合并

是因为很多业务担心自动major compaction影响读写性能，因此会选择低峰期手动触发；

也有可能是用户在执行完alter操作之后希望立刻生效，执行手动触发major compaction；

是HBase管理员发现硬盘容量不够的情况下手动触发major compaction删除大量过期数据；

### HFile 合并策略

合并操作主要是在一个Store里边找到需要合并的HFile，然后把它们合并起来，合并在大体意义上有两大类Minor Compation和Major Compaction：

- Minor Compaction：将Store中多个HFile合并为一个HFile，这个过程中，**达到TTL（记录保留时间）会被移除**，但是有墓碑标记的记录不会被移除，因为墓碑标记可能存储在不同HFile中，合并可能会跨过部分墓碑标记。这种合并的触发频率很高
- Major Compaction：**合并Store中所有的HFile为一个HFile**（并不是把一个Region中的HFile合并为一个），这个过程有墓碑标记的几率会被真正移除，同时超过单元格maxVersion的版本记录也会被删除。合并频率比较低，默认7天执行一次，并且性能消耗非常大，最后手动控制进行合并，防止出现在业务高峰期。

需要注意的是，有资料说只有Major合并才会删数据，其实Major合并删除的是带墓碑标记的，而Minor合并直接就不读取TTL过期文件，所以也相当于删除了。

具体步骤为： 

- 获取需要合并的HFile列表 
- 由列表创建出StoreFileScanner 
- 把数据从这些HFile中读出，并放到tmp目录（临时文件夹） 
- 用合并后的新HFile来替换合并前的那些HFile

#### minor Compaction

第一步：获取需要合并的HFile列表 
获取列表的时候需要排除掉带锁的HFile，分为写锁（write lock）和读锁（read lock），当HFile正在进行如下操作时候会进行上锁：

- 用户正在进行scan查询->上Region读锁（region read lock）
- Region正在切分（split）：此时Region会先关闭，然后上Region写锁（region write lock）
- Region关闭->上region写锁（region write lock）
- Region批量导入->Region写锁（region write lock）

第二步：由列表创建出StoreFileScanner 
HRegion会创建出一个Scanner，用Scanner读取所有需要合并的HFile上的数据 
第三步：把数据从这些HFile中读出，并放到tmp（临时目录） 
HBase会在临时目录中创建新的HFile，并把Scanner读取到的数据放入新的HFile，数据过期（达到了TTL时间）不会被读出： 
第四步：用合并后的HFile来替换合并前的那些HFile 
最后用那个临时文件夹合并后的新HFile来替换掉之前那些HFile文件，过期的数据由于没有被读取出来，所以就永远消失了。

HBase 会将队列中的storefile 按照文件年龄排序（older to younger），minor compaction总是从older store file开始选择。

（1）如果该文件小于`hbase.hstore.compaction.min.size`（为memestoreFlushSize）则一定会被添加到合并队列中。

（2）如果该文件大于`hbase.hstore.compaction.max.size（Long.MAX_VALUE）`则一定会被排除，这个值很大，一般不会有。

（3）如果该文件的size 小于它后面`hbase.hstore.compaction.max`（默认为10） 个store file size 之和乘以一个ratio（配置项是hbase.hstore.compaction.ratio，默认为1.2），则该storefile 也将加入到minor compaction 中。当然，如果他后面不足10个文件，那么也就是取他后面几个文件总和*ratio了。

如此，最终选择下来的文件就将进入Minor合并。

#### Major Compaction

HBase中并没有一种合并策略叫做Major Compaction，Major Compaction目的是增加读性能，在Minor Compaction的基础上可以实现删除掉带墓碑标记的数据。因为有时候，用户删除的数据，墓碑标记和原始数据这两个KeyValue在不同的HFile上。在Scanner阶段，会将所有HFile数据查看一遍，如果数据有墓碑标记，那么就直接不Scan这条数据。 
所以之前介绍的每一种合并策略，都有可能升级变为majorCompaction，如果本次Minor Compaction包含了当前Store所有的HFile，并且达到了足够的时间间隔，则会被升级为Major Compaction。这两个阀值的配置项为：

- hbase.hregion.majorcompaction：Major Compaction发生的周期，单位毫秒，默认7天
- hbase.hregion.majorcompaction.jitter：Major Compaction周期抖动参数，0-1.0，让MajorCompaction发生时间更加灵活，默认0.5

建议关闭自动 Major Compaction，将base.hregion.majorcompaction设置为0。在合适的时间，手动执行 Major Compaction。

##### 老版本的合并策略（0.96之前）RationBasedCompactionPolicy

基于固定因子的合并策略，从旧到新扫描HFile文件，当扫描到某个文件满足条件：

```
该文件大小 < 比它更新的所有文件大小综合 *hbase.store.compaction.ratio
```

就把该HFile和比它更新的所有HFile合并为一个HFile。 
但是实际上，Memstore很多情况下会有不同的刷写情况，所以每次的HFile不一定一样大，比如最后一次导入数据需要关闭Region，强制刷写导致数据非常少。实际情况下RatioBasedCompactionPolicy算法效果很差，经常应发大面积合并，合并就不能写入数据，影响IO。

##### 新版本合并策略 ExploringCompactionPolicy

是新版本的默认算法，不再是强顺序遍历，而是整体遍历一遍然后综合考虑，算法模型是：

```
该文件大小 < (所有文件大小综合 - 该文件大小) * 比例因子
```

如果HFile小于minCompactionSize，则不需要套用公式，直接进入待合并列表，如果没有配置该项，那么使用hbase.hregion.memstore.flush.size。

```
#合并因子hbase.store.compaction.ratio#合并策略执行最小的HFile数量hbase.hstore.compaction.min.size#合并策略执行最大的HFile数量hbase.hstore.compaction.max.size#memstore刷写值，hbase.hregion.memstore.flush.size#参与合并最小的HFile大小minCompactionSize
```

所以一个HFile是否进行参与合并：

- 小于minCompactionSize，直接进入合并列表
- 大于minCompactionSize，用公式决定是否参与合并
- 穷举出所有可执行合并列表，要求数量要>hbase.hstore.compaction.min.size，且<=hbase.hstore.compaction.max.size
- 比较所有组，选出其中数量最多
- 在数量最多且一样的组中，选择总大小最小的一组

##### FIFOCompactionPolicy

先进先出合并策略，最简单的合并算法，甚至可以说是一种删除策略。minor、major合并一定会发生但是频率不同，但是有时候没必要执行合并：

- TTL特别短，比如中间表，只是在计算中间暂存一些数据，很容易出现一整个HFile都过去
- BlockCache非常大，可以把整个RegionServer上的数据都放进去，就没必要合并了，因为所有数据都可以走缓存

因为合并会把TTL超时数据删掉，两个合并就回导致其实把HFile都删除了，而FIFO策略在合并时候就会跳过含有未过期数据的HFile，直接删除所有单元格都过期的块，所以结果就是过期的快上整个被删除，而没过期的完全没有操作。而过程对CPU、IO完全没有压力。但是这个策略不能用于：

- 表没有设置TTL，或者TTL=ForEver
- 表设置了Min_Versions，并且大于0

因为如果设置了min_versions，那么TTL就会失效，芮然达到了TTL时间，但是因为有最小版本数的限制，依旧不能删除。但是如果手动进行delete依旧可以删到小于最小版本数，min_versions只能约束TTL，所以FIFO不适用设置了min_versions的情况。

##### DateTieredCompactionPolicy

日期迭代合并删除策略，主要考虑了一个比较重要的点，最新的数据可能被读取的次数是最多的，比如电商中用户总是会查看最近的订单；随着时间的推移，用户会逐渐减缓对老数据的点击；非常古老的记录鲜有人问津。 
电商订单这种数据作为历史记录一定没有TTL，那么FIFO合并策略不合适；单纯的看大小进行合并也不太有效，如果把新数据和老数据合并了，反而不好，所以Exploring也不友好。Ratio就更别考虑了，每次合并都卷入最新的HFile。 
所以DateTieredCompactionPolicy的特点是：

- 为了新数据的读取性能，将新旧文件分开处理，新的和新的合并，旧的和旧的合并
- 不能只把数据分为新旧两种，因为读取频率是连续的，所以可以分为多个阶段，分的越细越好
- 太早的文件就不合并了，没什么人读取，合不合并没有太大差别，合并了还消耗性能

涉及到的配置项有有：

```
#初始化时间窗口时长，默认6小时，最新的数据只要在6小时内都会在初始化时间窗口内hbase.hstore.compaction.date.tiered.base.window.millis# 层次增长倍数，用于分层。越老的时间窗口越长，如果增长倍数是2，那么时间窗口为6、12、24小时，以此类推。如果时间窗口内的HFile数量达到了最小合并数量（hbase.hstore.compaction.min）就执行合并，并不是简单合并成一个，默认是使用ExploringCompactionPolicy（策略套策略），使用hbase.hstore.compaction.date.tiered.window.policy.class所定义hbase.hstore.compaction.date.tiered.windows.per.tier#最老的层次时间，如果文件太老超过了该时间定义的范围（默认天为单位）就直接不合并了。hbase.hstore.compaction.date.tiered.max.tier.age.millis#最小合并数量hbase.hstore.compaction.min
```

有一个确定就是，如果Store中的某个HFile太老了，但是有没有超过TTL，且大于了最老层次时间，那么这个HFile在超时被删除之前都不会被删除，不会发生Major合并，用户手动删除数据也不会真正被删除，而是一直占用着空间。 
其实归根到底，合并的最终策略是每个时间涌口中的HFile数量达到了最小合并数量，那么就回进行合并。另外，当一个HFile跨了时间线的时候，将其定义为下一个时间窗口（更老的更长的）。 
**适用场景：**

- 适用于经常读写最近数据的系统，专注于新数据
- 因为哪个缺点的原因可能引发不了Major合并，没法删除手动删除的信息，更适合基本不删数据的系统
- 根据时间排序的存储也适合
- 如果数据修改频率较低，只修改最近的数据，也推荐使用
- 如果数据改动平凡，老数据也修改，经常边读边写数据，那么就不太适合了。

##### StripeCompactionPolicy

简单来说就是将一个Store里边的数据分为多层，数据从Memstore刷写到HFile先落到level 0，当level 0大小超过一定的阀值时候会引发一次合并，会将level 0读取出来插入到level 1的HFile中去，而level 1的块是根据建委范围划分的，最早是分为多层的，后来感觉太复杂，将level 0改名为L0，而level 1-N合并成一个层叫做strips层。依旧是按照键位来划分块。 
这种策略的好处是：

- 通过增加L0层，给合并操作增加了一层缓冲，让合并操作更加缓和；
- 严格按照键位来划分Strips，碎玉读取虽然不能提高太多速度，但是可以提高查询速度的稳定性，执行scan时候，跨越的HFile数量保持在了一个比较稳定的数值
- Major合并本来是牵涉到一个store中的所有HFile，现在可以按照子Strip执行了，因为Major合并一直都会存在因为牵涉HFile太多导致的IO不稳定，而该策略一次只是牵涉到一个Strip中的文件，所以克服了IO不稳定的缺点

Stripe合并策略对于读取的优化要好于写的优化，所以很难说会提高多少IO性能，最大的好处就是稳定。那么什么场景适合StripeCompactionPolicy呢？

- Region要足够大，如果Region小于2GB那么就不适合该策略，因为小Region用strips会细分为多个stripe，反而增加IO负担
- RowKey要具有统一的格式，能够均匀分布，如果使用timestamp来做rowkey，那么数据就没法均匀分布了，而用26个首字母就比较合适

详细策略方案：https://juejin.im/post/5bfe978af265da61417147f2

### HFile合并的吞吐量限制参数

在使用过程中，如果突然发生IO降低，十有八九是发生compaction了，但是compaction又是不可获取的，所以就可以通过限制compaction的吞吐量来限制其占用的性能。 
由于HBase是分布式系统，吞吐量的概念是磁盘IO+网络IO的笼统概念，因为没办法具体判断哪一个的IO的限制更大，HBase的吞吐量是通过要合并的HFile的文件大小/处理时间得出的。未发生合并之前就没法测了，只能通过上一次的合并信息进行简单预测。具体的设置参数有；

```
#要限制的类型对象的类名，有两种
控制合并相关指标：PressureAwareCompactionThroughputController
控制刷写相关指标：PressureAwareFlushThroughputContollerhbase.regionserver.throughput.controller
#当StoreFile数量达到该值，阻塞刷写动作，默认是7
hbase.hstore.blockingStoreFile
#合并占用吞吐量下限
hbase.hstore.compaction.throughput.lower.bound
#合并占用吞吐量上限
hbase.hstore.compaction.throughput.higher.bound
```

hbase.hstore.blockingStoreFile 的设置比较讲究，如果设置不合理，当HFile数量达到该值之后，会导致Memstore占用的内存急剧上升，很快就达到了Memstore写入上限，导致memstore阻塞，一点都写不进去。所以Memstore达到阻塞值的时候，先不要急着调大Memstore的阻塞阀值，要综合考虑HFile的合并阻塞值，可以适当调大，20、30、50都不算多，HFile多，只是读取性能下降而已，但是达到阻塞值不只是慢的问题了，是直接写不进去了。

### 合并/刷写吞吐量限制机制

HBase会将合并和刷写总的吞吐量做计算，如果总吞吐量太大，那么进行适当休眠，因为这两个参数会限制合并时候占用的吞吐量，也会限制刷写时候占用的吞吐量。保证业务的响应流畅、保障系统的稳定性。限制会区分高峰时段和非高峰时段，通过如下两个参数：

```
#每天非高峰的起始时间，取值为0-23的整数hbase.offpeak.start.hour#每天非高峰的结束时间，取值为0-23的整数hbase.offpeak.end.hour
```

通过设置非高峰，其他时段就是高峰时段了。在非高峰期是不会进行限速，只有在高峰期占用了太大的吞吐量才会休眠，主要看一个阀值：

```
lowerBound+(upperBound-lowerBound)*pressureRatio
#合并占用吞吐量的下限，取值是10MB/sec
lowerBound:hbase.hstore.compaction.throughput.lower.bound
#合并占用吞吐量的上限，取值是20MB/sec
upperBound:hbase.hstore.compaction.throughput.higher.bound
#压力比，限制合并时候，该参数就是合并压力（compactionpressure），限制刷写时候，该参数刷写压力（flushPressure），为0-1.0
pressureRatio
```

### 压力比

压力比（pressureRatio）越大，代表HFile堆积的越多，或者即将产生越多的HFile，一旦达到HFile的阻塞阀值，那么久无法写入数据了，所以合并压力比越大，合并的需求变得迫在眉睫。压力比越大，吞吐量的阀值越高，意味着合并线程可以占用更多的吞吐量来进行合并。 
压力比有两种： 
**合并压力（compactionPressure）**：

```
(storefileCount-minFilesToCompact) / (blockingFileCount-minFilesToCompact)
storefileCount:当前StoreFile数量
minFilesToCompact：单词合并文件的数量下限，即hbase.hstore.compaction.min
blockingFileCount：就是hbase.hstore.blockingStoreFiles
```

当前的StoreFile越大，或者阻塞上限越小，合并压力越大，更可能发生阻塞 
**刷写压力(flusthPressure)**：

```
globalMemstoreSize /  memstoreLowerLimitSize
globalMemstore :当前Msmstore大小
memstoreLowerLimitSize：Memstore刷写的下限，当全局memstore达到这个内存占用数量的时候就会开始刷写
```

如果当前的Memstore占用内存越大，或者触发条件越小，越有可能引发刷写，刷写后HFile增多，就有可能发生HFile过多阻塞。

## HBase Meta表简介

### Meta表作用

我们知道HBase的表是会分割为多个Region的，不同Region分布到不同RegionServer上。Region 是 HBase中分布式存储和负载均衡的最小单元。

所以当我们从客户端读取，写入数据的时候，我们就需要知道我么数据的 Rowkey是在哪个Region的范围以及我们需要的Region是在哪个RegionServer上。

而这正是HBase Meta表所记录的信息。

### Meta表的Rowkey

region所在的 **表名+region的StartKey+时间戳**。而这三者的MD5值也是HBase在HDFS上存储的region的名字。

### Meta表的列族和列

表中最主要的Family：info

info里面包含三个Column：regioninfo, server, serverstartcode。

其中regioninfo就是Region的详细信息，包括StartKey, EndKey 以及每个Family的信息等。server存储的就是管理这个Region的RegionServer的地址。

所以当Region被拆分、合并或者重新分配的时候，都需要来修改这张表的内容。

### Region的定位

第一次读取： 步骤1：读取ZooKeeper中META表的位置。 步骤2：读取.META表中用户表的位置。 步骤3：读取数据。

如果已经读取过一次，则root表和.META都会缓存到本地，直接去用户表的位置读取数据。