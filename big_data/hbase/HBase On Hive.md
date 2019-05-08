# HBase On Hive

## 性能

查询性能比较 

query1:

```
select count(1) from on_hdfs;
select count(1) from on_hbase;
```

query2(根据key过滤)

```
select * from on_hdfs
where key = ‘13400000064_1388056783_460095106148962′;
select * from on_hbase
where key = ‘13400000064_1388056783_460095106148962′;
```

query3(根据value过滤)

```
select * from on_hdfs where value = ‘XXX';
select * from on_hbase where value = ‘XXX';
```

on_hdfs (20万记录，150M，TextFile on HDFS) 
on_hbase(20万记录，160M，HFile on HDFS) 

![image-20190423155620701](assets/image-20190423155620701.png)

on_hdfs (2500万记录，2.7G，TextFile on HDFS) 
on_hbase(2500万记录，3G，HFile on HDFS) 

![image-20190423155648834](assets/image-20190423155648834.png)

> 对于全表扫描，hive_on_hbase查询时候如果不设置catching，性能远远不及hive_on_hdfs； 根据rowkey过滤，hive_on_hbase性能上略好于hive_on_hdfs，特别是数据量大的时候； 设置了caching之后，尽管比不设caching好很多，但还是略逊于hive_on_hdfs；

https://www.iteblog.com/archives/1718.html

https://www.cnblogs.com/xuwujing/p/8059079.html

https://blog.csdn.net/ymf827311945/article/details/73690927

