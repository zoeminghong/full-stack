# Spark with ES

[Support SPNEGO/Kerberos auth to Elasticsearch · Issue #1175 · elastic/elasticsearch-hadoop · GitHub](https://github.com/elastic/elasticsearch-hadoop/issues/1175)
[Elastic Stack 笔记（十）Elasticsearch5.6 For Hadoop - 沐小悠 - 博客园](https://www.cnblogs.com/cnjavahome/p/9217025.html)
[Spark 2.0 保存数据到Elasticsearch 6 - baifanwudi的专栏 - CSDN博客](https://blog.csdn.net/baifanwudi/article/details/80258663)
[Elasticsearch Java API - 客户端连接(TransportClient，PreBuiltXPackTransportClient)（一） - Elastic 中文社区](https://elasticsearch.cn/article/380)
[spark读写ES数据 - 努力中国 - 博客园](https://www.cnblogs.com/kaiwen1/p/9138245.html)
[elasticsearch-spark更新文档 - 简书](https://www.jianshu.com/p/6dedf0e4620f)
[Configuration        | Elasticsearch for Apache Hadoop master      | Elastic](https://www.elastic.co/guide/en/elasticsearch/hadoop/master/configuration.html)

ES-HBase
[HBase 实现数据同步 ElasticSearch - 蛮-com | 醉里挑灯看剑](http://www.niuchaoqun.com/14969255996574.html)

[Elasticsearch+Hbase实现海量数据秒回查询](https://www.bbsmax.com/A/Ae5Rgl18dQ/)
[Apache Spark support        | Elasticsearch for Apache Hadoop master      | Elastic](https://www.elastic.co/guide/en/elasticsearch/hadoop/master/spark.html#spark-sql-streaming)

spark
[Spark大数据之DataFrame和Dataset - 知乎](https://zhuanlan.zhihu.com/p/29830732)
[scala - How to use from_json with Kafka connect 0.10 and Spark Structured Streaming? - Stack Overflow](https://stackoverflow.com/questions/42506801/how-to-use-from-json-with-kafka-connect-0-10-and-spark-structured-streaming/42514979)
https://stackoverflow.com/questions/34069282/how-to-query-json-data-column-using-spark-dataframes
[Split 1 column into 3 columns in spark scala - Stack Overflow](https://stackoverflow.com/questions/39255973/split-1-column-into-3-columns-in-spark-scala)
Col 可以使用 $ 代替
```
.withColumn("body", col("value").cast(StringType))
.select($"value" cast "string" as "json")
```
[Spark SQL操作JSON字段小Tips - 简书](https://www.jianshu.com/p/2d21f0de6230)


https://blog.cloudera.com/blog/2017/12/hadoop-delegation-tokens-explained/



<https://help.aliyun.com/document_detail/93204.html?spm=5176.10695662.1996646101.searchclickresult.fcc6adb7ZCmPRA>

如果数据是 json 格式的，多层级的，需要以内部层级字段作为参数进行调用