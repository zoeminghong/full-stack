## ElasticSeach在大数据中的实践应用

## ElasticSeach API

### 集群信息

[Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster.html)

#### 查看集群健康状态 `/_cluster/health`

```shell
 curl -XGET  -H "Content-Type:application/json" --insecure -u admin:admin   "https://localhost:9200/_cluster/health?pretty"
 
 {
  "cluster_name" : "elasticsearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 643,
  "active_shards" : 1287,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

#### 查看集群节点配置信息 `/_nodes/process`

```shell
curl -XGET  -H "Content-Type:application/json" --insecure -u admin:admin   "https://localhost:9200/_nodes/process?pretty"

{
  "_nodes" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "cluster_name" : "elasticsearch",
  "nodes" : {
    "wtoVlGOwS0yTd2x57yszig" : {
      "name" : "dmp-node3",
      "transport_address" : "localhost1:9300",
      "host" : "localhost1",
      "ip" : "localhost1",
      "version" : "6.5.4",
      "build_flavor" : "oss",
      "build_type" : "rpm",
      "build_hash" : "d2ef93d",
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 13235,
        "mlockall" : false
      }
    },
    "fpFgEKSjRH2kYwvp5D7Adw" : {
      "name" : "dmp-node1",
      "transport_address" : "localhost2:9300",
      "host" : "localhost2",
      "ip" : "localhost2",
      "version" : "6.5.4",
      "build_flavor" : "oss",
      "build_type" : "rpm",
      "build_hash" : "d2ef93d",
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 22193,
        "mlockall" : false
      }
    },
    "ubJ_mj36SzCFktvF2gzFnA" : {
      "name" : "dmp-node2",
      "transport_address" : "localhost3:9300",
      "host" : "localhost3",
      "ip" : "localhost3",
      "version" : "6.5.4",
      "build_flavor" : "oss",
      "build_type" : "rpm",
      "build_hash" : "d2ef93d",
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 783,
        "mlockall" : false
      }
    }
  }
}
```

## 命令行下 Cat 的结果

[Cat API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html)

#### 显示 shard 视图 `/_cat/shards`

https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html

```shell
 curl -XGET  -H "Content-Type:application/json" --insecure -u admin:admin   "https://localhost:9200/_cat/shards"
 
dop:dop_user_label                         0 r STARTED    216  193.9kb 10.200.131.187 dmp-node1
dop:dop_user_label                         0 p STARTED    216  193.9kb 10.200.131.189 dmp-node3
```



### 拓展

[elasticsearch Doc](https://www.elastic.co/guide/en/elasticsearch/hadoop/master/spark.html#spark-sql-streaming)

[elasticsearch Configuration](https://www.elastic.co/guide/en/elasticsearch/hadoop/master/configuration.html)

[Elasticsearch史上最全最常用工具清单](https://cloud.tencent.com/developer/article/1166460)

[干货 | Elasticsearch多表关联设计指南](https://www.javazhiyin.com/35154.html)

[Canal2ES](https://github.com/alibaba/canal/wiki/Sync-ES)

[kibana 配置项](https://www.elastic.co/guide/en/kibana/current/settings.html)

[Elasticsearch索引的性能注意事项](https://www.elastic.co/cn/blog/performance-considerations-elasticsearch-indexing)

## opendistro

[SQL](https://opendistro.github.io/for-elasticsearch-docs/docs/sql/)

