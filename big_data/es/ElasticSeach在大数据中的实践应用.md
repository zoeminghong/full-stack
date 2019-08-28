## ElasticSeach在大数据中的实践应用

数据格式多变，弹性拓展字段，一旦一个错误字段同步到es，可能就会导致需要删除索引重新同步，同时，如果数据是由多方汇集的，获取数据难度系数比较高，就会存在问题。

## ElasticSearch SQL

API：https://admin:admin@10.200.131.184:9200/_opendistro/_sql

```shell
POST/GET
{
  "query":"select * from dop:dop_visit_event_info where  _id =  '2019-06-27 10:43:00.388|000001287429205000000141040411000002067845529'"
}

```

[语法](https://juejin.im/entry/5c137dca5188257e2a7b4e93)

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-sql.html)

[sql-function-conditional](https://www.elastic.co/guide/en/elasticsearch/reference/master/sql-functions-conditional.html)

### 不等于

```shell
script('!Objects.equals(doc["DOP_VISIT_FAKE_STAYTIME"].value,-1)')
```

https://github.com/NLPchina/elasticsearch-sql/wiki/ScriptQuery

## 关于 ElasticSearch 所有列和部分列数据查询性能记录

> 说明：在没有关闭 `_all` 情况下测试，字段 100 多个，数据体量 200多万条。

结论：`select *` 与指定字段查询时间差异不大。

![92951AC8-6047-4440-AF6F-CB8D6C4FC072](assets/92951AC8-6047-4440-AF6F-CB8D6C4FC072.png)

![AED75ABF-56D7-4647-A949-0C79B8D7042C](assets/AED75ABF-56D7-4647-A949-0C79B8D7042C.png)

![59B2AC80-3CD2-409b-8E81-B67044371CDD](assets/59B2AC80-3CD2-409b-8E81-B67044371CDD.png)拓展

[elasticsearch Doc](https://www.elastic.co/guide/en/elasticsearch/hadoop/master/spark.html#spark-sql-streaming)

[elasticsearch Configuration](https://www.elastic.co/guide/en/elasticsearch/hadoop/master/configuration.html)

[Elasticsearch史上最全最常用工具清单](https://cloud.tencent.com/developer/article/1166460)

[干货 | Elasticsearch多表关联设计指南](https://www.javazhiyin.com/35154.html)

[Canal2ES](https://github.com/alibaba/canal/wiki/Sync-ES)

[kibana 配置项](https://www.elastic.co/guide/en/kibana/current/settings.html)

[Elasticsearch索引的性能注意事项](https://www.elastic.co/cn/blog/performance-considerations-elasticsearch-indexing)

[性能优化](https://www.easyice.cn/archives/207)

[性能优化](https://blog.csdn.net/laoyang360/article/details/85109769)

## opendistro

[SQL](https://opendistro.github.io/for-elasticsearch-docs/docs/sql/)

