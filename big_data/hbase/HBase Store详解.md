MemStore由一个可写的Segment，以及一个或多个不可写的Segments构成。

将一部分合并工作上移到内存。减少Hfile生成数量（4次-->1次），减少写的次数。改善IO放大问题。频繁的Compaction对IO资源的抢占，其实也是导致HBase查询时延大毛刺的罪魁祸首之一

- 那为何不干脆调大MemStore的大小？这里的本质原因在于，ConcurrentSkipListMap在存储的数据量达到一定大小以后，写入性能将会出现显著的恶化。

问答

**一个 Region下 相同列族会有多个 store 吗？**

不会，同一个 Region 下一个列族对应有且只有一个 Store。我们知道 HBase 是通过 Region 的 StartKey 和 EndKey 确定当前 RowKey 存储在哪个 Region 下，同时，HFile 文件合并时，只是对当前的 Store 下的 HFile 集合合并为一个新的 HFile，删除被合并的文件。假设一个列族存在多个 Store ，根据合并规则无法保证已被删除的数据合并正确，可能会出现，一个 Store 下已经被删除，另一个 Store 下未被删除。（HDFS 文件只支持 Append 操作，HBase 的数据修改新增删除操作都是一次 Append 操作）

