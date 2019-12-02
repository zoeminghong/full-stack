# JVM 调优

## JVM调优参数简介

### JVM参数简介

-XX 参数被称为不稳定参数，之所以这么叫是因为此类参数的设置很容易引起JVM 性能上的差异，使JVM 存在极大的不稳定性。如果此类参数设置合理将大大提高JVM 的性能及稳定性。

不稳定参数语法规则：

1.布尔类型参数值
```
-XX:+<option> '+'表示启用该选项
-XX:-<option> '-'表示关闭该选项
```
2.数字类型参数值：
  `-XX:<option>=<number>` 给选项设置一个数字类型值，可跟随单位，例如：'m'或'M'表示兆字节;'k'或'K'千字节;'g'或'G'千兆字节。32K与32768是相同大小的。
3.字符串类型参数值：
`-XX:<option>=<string>` 给选项设置一个字符串类型值，通常用于指定一个文件、路径或一系列命令列表。
例如：`-XX:HeapDumpPath=./dump.core`

### JVM参数示例

配置：` -Xmx4g –Xms4g –Xmn1200m –Xss512k -XX:NewRatio=4 -XX:SurvivorRatio=8 -XX:PermSize=100m`

`-XX:MaxPermSize=256m -XX:MaxTenuringThreshold=15`

解析：
`  -Xmx4g`：堆内存最大值为4GB。

`-Xms4g`：初始化堆内存大小为4GB 。

`-Xmn1200m`：设置年轻代大小为1200MB。增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。

`-Xss512k`：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1MB，以前每个线程堆栈大小为256K。应根据应用线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

`-XX:NewRatio=4`：设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5

`-XX:SurvivorRatio=8`：设置年轻代中Eden区与Survivor区的大小比值。设置为8，则两个Survivor区与一个Eden区的比值为2:8，一个Survivor区占整个年轻代的1/10

`-XX:PermSize=100m`：初始化永久代大小为100MB。

`-XX:MaxPermSize=256m`：设置持久代大小为256MB。

`-XX:MaxTenuringThreshold=15`：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。

## JVM调优目标

1. 何时需要做jvm调优？
      1. heap 内存（老年代）持续上涨达到设置的最大内存值；
      2. Full GC 次数频繁；
      3. GC 停顿时间过长（超过1秒）；
      4. 应用出现 OutOfMemory 等内存异常；
      5. 应用中有使用本地缓存且占用大量内存空间；
      6. 系统吞吐量与响应性能不高或下降。

2. JVM调优原则

      1. 多数的Java应用不需要在服务器上进行JVM优化；

      2. 多数导致GC问题的Java应用，都不是因为我们参数设置错误，而是代码问题；

      3. 在应用上线之前，先考虑将机器的JVM参数设置到最优（最适合）；

      4. 减少创建对象的数量；

      5. 减少使用全局变量和大对象；

      6. JVM优化是到最后不得已才采用的手段；

      7. 在实际使用中，分析GC情况优化代码比优化JVM参数更好；

3. JVM调优目标

      1. GC低停顿；

      2. GC低频率；

      3. 低内存占用； 

      4. 高吞吐量;

JVM调优量化目标（示例）：

  1. Heap 内存使用率 <= 70%;

  2. Old generation内存使用率<= 70%;

  3. avgpause <= 1秒; 

  4. Full gc 次数0 或 avg pause interval >= 24小时 ;

  注意：不同应用，其JVM调优量化目标是不一样的。

## JVM调优经验

### JVM调优经验总结

**JVM调优的一般步骤为：**

​      第1步：分析GC日志及dump文件，判断是否需要优化，确定瓶颈问题点；

​      第2步：确定JVM调优量化目标；

​      第3步：确定JVM调优参数（根据历史JVM参数来调整）；

​      第4步：调优一台服务器，对比观察调优前后的差异；

​      第5步：不断的分析和调整，直到找到合适的JVM参数配置；

​      第6步：找到最合适的参数，将这些参数应用到所有服务器，并进行后续跟踪。

###  JVM调优重要参数解析

注意：不同应用，其JVM最佳稳定参数配置是不一样的。

```shell
配置： -server  

-Xms12g -Xmx12g -XX:PermSize=500m -XX:MaxPermSize=1000m -Xmn2400m -XX:SurvivorRatio=1 -Xss512k  -XX:MaxDirectMemorySize=1G 

-XX:+DisableExplicitGC -XX:CompileThreshold=8000 -XX:+UseConcMarkSweepGC  -XX:+UseParNewGC

-XX:+UseCompressedOops -XX:CMSInitiatingOccupancyFraction=60  -XX:ConcGCThreads=4

-XX:MaxTenuringThreshold=10  -XX:ParallelGCThreads=8

-XX:+ParallelRefProcEnabled  -XX:+CMSClassUnloadingEnabled  -XX:+CMSParallelRemarkEnabled

-XX:CMSMaxAbortablePrecleanTime=500 -XX:CMSFullGCsBeforeCompaction=4 

XX:+UseCMSInitiatingOccupancyOnly -XX:+UseCMSCompactAtFullCollection 

-XX:+HeapDumpOnOutOfMemoryError  -verbose:gc  -XX:+PrintGCDetails  -XX:+PrintGCDateStamps  -Xloggc:/weblogic/gc/gc_$$.log

重要参数（可调优）解析：

-Xms12g：初始化堆内存大小为12GB。

-Xmx12g：堆内存最大值为12GB 。

-Xmn2400m：新生代大小为2400MB，包括 Eden区与2个Survivor区。

-XX:SurvivorRatio=1：Eden区与一个Survivor区比值为1:1。

-XX:MaxDirectMemorySize=1G：直接内存。报java.lang.OutOfMemoryError: Direct buffer memory 异常可以上调这个值。

-XX:+DisableExplicitGC：禁止运行期显式地调用 System.gc() 来触发fulll GC。

注意: Java RMI的定时GC触发机制可通过配置-Dsun.rmi.dgc.server.gcInterval=86400来控制触发的时间。

-XX:CMSInitiatingOccupancyFraction=60：老年代内存回收阈值，默认值为68。

-XX:ConcGCThreads=4：CMS垃圾回收器并行线程线，推荐值为CPU核心数。

-XX:ParallelGCThreads=8：新生代并行收集器的线程数。

-XX:MaxTenuringThreshold=10：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。

-XX:CMSFullGCsBeforeCompaction=4：指定进行多少次fullGC之后，进行tenured区 内存空间压缩。

-XX:CMSMaxAbortablePrecleanTime=500：当abortable-preclean预清理阶段执行达到这个时间时就会结束。
```

### 触发Full GC的场景及应对策略

年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC，对老年代GC称为MajorGC，而Full GC是对整个堆来说的，在最近几个版本的JDK里默认包括了对永生带即方法区的回收（JDK8中无永生带了），出现Full GC的时候经常伴随至少一次的Minor GC,但非绝对的。MajorGC的速度一般会比Minor GC慢10倍以上。

触发Full GC的场景及应对策略： 

1. System.gc()方法的调用，应对策略：通过-XX:+DisableExplicitGC来禁止调用System.gc ;

2. 老年代空间不足，应对策略：让对象在Minor GC阶段被回收，让对象在新生代多存活一段时间，不要创建过大的对象及数组;
3. 3. 永生区空间不足，应对策略：增大PermGen空间

4. GC时出现promotionfailed和concurrent mode failure，应对策略：增大survivor space
5. Minor GC后晋升到旧生代的对象大小大于老年代的剩余空间，应对策略：增大Tenured space 或下调CMSInitiatingOccupancyFraction=60

6. 内存持续增涨达到上限导致Full GC  ，应对策略：通过dumpheap 分析是否存在内存泄漏

### Gc日志分析工具

借助GCViewer日志分析工具，可以非常直观地分析出待调优点。

可从以下几方面来分析：

​      1.Memory,分析Totalheap、Tenuredheap、Youngheap内存占用率及其他指标，理论上内存占用率越小越好；

​      2.Pause  ,分析Gc pause、Fullgc pause、Total pause三个大项中各指标，理论上GC次数越少越好，GC时长越小越好；

### MAT 堆内存分析工具

EclipseMemory Analysis Tools (MAT) 是一个分析Java堆数据的专业工具，用它可以定位内存泄漏的原因。

## MaxGCPauseMillis调优

前面介绍过使用GC的最基本的参数：
`-XX:+UseG1GC -Xmx32g -XX:MaxGCPauseMillis=200`

前面2个参数都好理解，后面这个MaxGCPauseMillis参数该怎么配置呢？这个参数从字面的意思上看，就是允许的GC最大的暂停时间。G1尽量确保每次GC暂停的时间都在设置的MaxGCPauseMillis范围内。 那G1是如何做到最大暂停时间的呢？这涉及到另一个概念，CSet(collection set)。它的意思是在一次垃圾收集器中被收集的区域集合。
Young GC：选定所有新生代里的region。通过控制新生代的region个数来控制young GC的开销。
Mixed GC：选定所有新生代里的region，外加根据global concurrent marking统计得出收集收益高的若干老年代region。在用户指定的开销目标范围内尽可能选择收益高的老年代region。

在理解了这些后，我们再设置最大暂停时间就好办了。 首先，我们能容忍的最大暂停时间是有一个限度的，我们需要在这个限度范围内设置。但是应该设置的值是多少呢？我们需要在吞吐量跟MaxGCPauseMillis之间做一个平衡。如果MaxGCPauseMillis设置的过小，那么GC就会频繁，吞吐量就会下降。如果MaxGCPauseMillis设置的过大，应用程序暂停时间就会变长。G1的默认暂停时间是200毫秒，我们可以从这里入手，调整合适的时间。

## 其他调优参数

-XX:G1HeapRegionSize=n
设置的 G1 区域的大小。值是 2 的幂，范围是 1 MB 到 32 MB 之间。目标是根据最小的 Java 堆大小划分出约 2048 个区域。
-XX:ParallelGCThreads=n
设置 STW 工作线程数的值。将 n 的值设置为逻辑处理器的数量。n 的值与逻辑处理器的数量相同，最多为 8。
如果逻辑处理器不止八个，则将 n 的值设置为逻辑处理器数的 5/8 左右。这适用于大多数情况，除非是较大的 SPARC 系统，其中 n 的值可以是逻辑处理器数的 5/16 左右。
-XX:ConcGCThreads=n
设置并行标记的线程数。将 n 设置为并行垃圾回收线程数 (ParallelGCThreads) 的 1/4 左右。
-XX:InitiatingHeapOccupancyPercent=45
设置触发标记周期的 Java 堆占用率阈值。默认占用率是整个 Java 堆的 45%。
避免使用以下参数：
避免使用 -Xmn 选项或 -XX:NewRatio 等其他相关选项显式设置年轻代大小。固定年轻代的大小会覆盖暂停时间目标。

## 触发Full GC

在某些情况下，G1触发了Full GC，这时G1会退化使用Serial收集器来完成垃圾的清理工作，它仅仅使用单线程来完成GC工作，GC暂停时间将达到秒级别的。整个应用处于假死状态，不能处理任何请求，我们的程序当然不希望看到这些。那么发生Full GC的情况有哪些呢？
并发模式失败

G1启动标记周期，但在Mix GC之前，老年代就被填满，这时候G1会放弃标记周期。这种情形下，需要增加堆大小，或者调整周期（例如增加线程数-XX:ConcGCThreads等）。
晋升失败或者疏散失败

G1在进行GC的时候没有足够的内存供存活对象或晋升对象使用，由此触发了Full GC。可以在日志中看到(to-space exhausted)或者（to-space overflow）。解决这种问题的方式是：
a,增加 -XX:G1ReservePercent 选项的值（并相应增加总的堆大小），为“目标空间”增加预留内存量。
b,通过减少 -XX:InitiatingHeapOccupancyPercent 提前启动标记周期。
c,也可以通过增加 -XX:ConcGCThreads 选项的值来增加并行标记线程的数目。
巨型对象分配失败

当巨型对象找不到合适的空间进行分配时，就会启动Full GC，来释放空间。这种情况下，应该避免分配大量的巨型对象，增加内存或者增大-XX:G1HeapRegionSize，使巨型对象不再是巨型对象。



## 问题

1、导致 OutOfMemory 原因？

2、常见调优参数有哪些？

3、如何获取服务器堆栈信息？

4、如何对堆栈信息进行分析？

5、GC 停顿时间过长，原因有哪些？