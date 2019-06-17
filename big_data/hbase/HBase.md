# HBase

[了解HBase](https://mp.weixin.qq.com/s/XNeOceRbpPAUz5_IWLRsyw)

## HBase Master

HMaster服务器控制HBase集群。您可以启动最多9个备用HMaster服务器，这使得10个HMaster服务器成为主服务器。要启动备份HMaster，请使用`local-master-backup.sh`。对于要启动的每个备份主站，添加一个表示该主站的端口偏移量的参数。每个HMaster使用两个端口（默认为16000和16010）。端口偏移量将添加到这些端口，因此使用偏移量2，备份HMaster将使用端口16002和16012.以下命令使用端口16002 / 16012,16003 / 16013和16005/16015启动3个备份服务器。

```shell
./bin/local-master-backup.sh start 2 3 5
```

文件打开数

```
(StoreFiles per ColumnFamily) x (regions per RegionServer)
```

在大多数情况下都可以通过减小StoreFiles的大小来提高性能，从而减少I / O.

HBase使用[wal](https://hbase.apache.org/book.html#wal)来恢复在RS出现故障时尚未刷新到磁盘的memstore数据。这些WAL文件应配置为略小于HDFS块（默认情况下，HDFS块为64Mb，WAL文件为~60Mb）。

## 锁表恢复

1.获取表的状态
get 'hbase:meta','DMP:DS_TRMALL_ORDER_GOOD','table:state'
2.disable
 put 'hbase:meta','DMP:DS_TRMALL_ORDER_GOOD','table:state',"\b\1"
3.确保表锁了
 is_disabled 'DMP:DS_TRMALL_ORDER_GOOD'
4.备份表
 snapshot 'DMP:DS_TRMALL_ORDER_GOOD', 'cata_tableSnapshot'
5.查看快照，确保有快照内容
 list_snapshots
6.还原表
 restore_snapshot 'cata_tableSnapshot'
7.重启master
8.enbale 表
enable 'DMP:DS_TRMALL_ORDER_GOOD'