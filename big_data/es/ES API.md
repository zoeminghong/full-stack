# ElasticSeach API

[TOC]

## Cluster

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

#### 查看集群配置 `_cluster/settings`

```shell
GET /_cluster/settings
GET /_cluster/settings?include_defaults=true
```

#### 修改集群配置 `_cluster/settings`

```shell
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" 
    }
}
```

## Node

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

#### 查看指定节点信息

```
GET /_nodes/es7_01,es7_02
```

## Cat

[Cat API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html)

#### 显示 shard 视图 `/_cat/shards`

https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html

```shell
 curl -XGET  -H "Content-Type:application/json" --insecure -u admin:admin   "https://localhost:9200/_cat/shards"
 
dop:dop_user_label                         0 r STARTED    216  193.9kb 10.200.131.187 dmp-node1
dop:dop_user_label                         0 p STARTED    216  193.9kb 10.200.131.189 dmp-node3
```

#### 查看集群状态 `_cat/health?v`

```
GET _cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1563072306 02:45:06  elasticsearch green           4         4    542 270    0    0        0             0                  -                100.0%
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m
```

#### 查看所有索引信息 `_cat/indices?v`

```shell
curl -X GET 'https://admin:admin@10.200.131.184:9200/_cat/indices?v'
 
health status index                                uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   dmp:yyy_customer_contacts            eUA40wuLTnuirgYI-WXyiw   5   1        515            0    515.8kb        257.9kb
green  open   dop:dop_device_unique_month          DmdLO59QSbiFxK1hydrKlw   5   1       5860           33      6.2mb          3.1mb
```

## Index

#### 修改索引 `[index]/_doc/[id]`

```shell
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

#### 创建索引  `/[index]?pretty`

```
PUT /customer?pretty
```

#### 添加/修改文档 `/[index]/_doc/[id]`  （全量更新）

```shell
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
# 未指定 ID
POST /customer/_doc?pretty
{
  "name": "Jane Doe"
}
```

#### 查看文档 `/[index]/_doc/[id]`

```shell
GET /customer/_doc/1?pretty
```

#### 删除索引  `/[index]`

```
DELETE /customer?pretty
```

#### 修改文档 `/[index]]/_doc/[id]/_update` （局部更新）

```shell
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe" }
}

POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

#### 删除文档 `/[index]/_doc/[id]`

```
DELETE /customer/_doc/2?pretty
```

#### 批量操作` /[index]/_doc/_bulk`

```shell
POST /customer/_doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

## Search

#### 集群上所有索引 `/_search` 

#### 具体索引查询 `/[index]/_search` 

```shell
# customer_first_name 字段值为 Eddie 的数据
GET kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie
```

```shell
#REQUEST Body
POST kibana_sample_data_ecommerce/_search
{
	"profile": true,
	"query": {
		"match_all": {}
	}
}
```

#### 多个索引查询 `/[index1,index2]/_search` 

#### 模糊索引匹配查询 `/[index*]/_search`

```
GET kibana*/_search?q=customer_first_name:Eddie
```

参数说明

**query:** 查询语句，Query String Syntax 语法

**df:** 默认字段，不指定时，会对所有字段进行查询

**size:** 返回数据条数

**from:** 指定从哪条开始返回数据，用于分页

**sort:** 排序

**profile:** 可以查看查询是如何被执行的

```
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
```

```
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}

GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```

### Query String Syntax 

#### 指定字段 VS 泛查询

q=title:2012 / q=2012

#### Term VS Phrase

Beautiful Mind 等效于 Beautiful OR Mind

“Beautiful Mind” 等效于 Beautiful AND Mind ，还要求前后顺序保持一致

#### 分组与引号

title:(Beautiful AND Mind)  使用括号标识分组

title="Beautiful Mind"

```shell
#使用引号，Phrase查询
GET /movies/_search?q=title:"Beautiful Mind"
{
	"profile":"true"
}

#分组，Bool查询
GET /movies/_search?q=title:(Beautiful Mind)
{
	"profile":"true"
}
```

布尔操作

AND / OR / NOT 或者 && / || / !

- 必须大写
- title:(matrix NOT reloaded)

分组

- `+` 表示 must  `%2`
- `-` 表示 must_not 
- title:(+matrix - reloaded)

```
#布尔操作符
# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful AND Mind)
{
	"profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful NOT Mind)
{
	"profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful %2BMind)
{
	"profile":"true"
}

```

范围查询

区间表示：[] 闭区间，{} 开区间

- year:{2019 TO 2018}
- year:[* TO 2018]

算数符号

- Year:>2010
- Year:{>2010 && <=2018}
- Year:{+>2010 +<=2018}

```
#范围查询 ,区间写法
GET /movies/_search?q=title:beautiful AND year:[2002 TO 2018%7D
{
	"profile":"true"
}
```

通配符查询（查询效率低，占用内存大，不建议使用，特别是放在最前面）

- ？代表一个字符，* 代表 0 或多个字符
  - Title:mi?d
  - Title:be*

正则表达式

- title:[bt]oy

模糊匹配与近似查询

- title:beatifl~1
- title:"lord rings"~2

```
#通配符查询
GET /movies/_search?q=title:b*
{
	"profile":"true"
}

//模糊匹配&近似度匹配
GET /movies/_search?q=title:beautifl~1
{
	"profile":"true"
}

GET /movies/_search?q=title:"Lord Rings"~2
{
	"profile":"true"
}
```

`_source` : 查询结果字段

```
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```

```shell
GET /bank/_search
{
# 查询 account_number 值为 20
  "query": { "match": { "account_number": 20 } }
}
```

```shell
GET /bank/_search
{
# 查询 address 中包含 mill 或者 lane
  "query": { "match": { "address": "mill lane" } }
}
```

```shell
GET /bank/_search
{
# 查询 address 中包含 mill lane 的值
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

```shell
GET /bank/_search
{
  "query": {
  # 查询 address 中含有 mill 和 lane 的数据
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

```shell
GET /bank/_search
{
  "query": {
  # 查询 address 中包含 mill 或者 lane
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

```shell
GET /bank/_search
{
# 没有 mill 和 lane 的address 值
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

```shell
GET /bank/_search
{
  "query": {
  # age 值为 40 但 state 不含有 ID
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

```shell
GET /bank/_search
{
# balance 大于等于 20000 小于 30000
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

```shell
GET /bank/_search
{
# 按照字段state分组
# 其中size设置为0意思是不返回文档内容，仅返回聚合结果
# state.keyword表示字段精确匹配，因为使用模糊匹配性能很低，所以不支持
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

```shell
GET /bank/_search
{
# 我们可以在聚合的基础上再进行聚合，例如求和、求平均值等等。
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      # balance 求平均数
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

```shell
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        # 平均数 倒序
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

```shell
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```

```shell
GET /bank/_search
{
  "query": {
        "match" : {
            "address" : "Putnam"
        }
    }
}

GET /bank/_search
{
  "query": {
        "match" : {
            "address.keyword" : "Putnam"
        }
    }
}

GET /bank/_search
{
  "query": {
        "term" : {
            "address" : "Putnam"
        }
    }
}
# 结果：
1、第一个能匹配到数据，因为会分词查询。
2、第二个不能匹配到数据，因为不分词的话没有该条数据。
3、结果不确定。需要看实际是怎么分词的。
term query 查询的是倒排索引中确切的term
match query 会对filed进行分词操作，然后再查询
```

#### 搜索数据 ` /[index]/_search`

```shell
GET /bank/_search?q=*&sort=account_number:asc&pretty
# q 查询条件
# sort 查询结果排序
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
结果
{
  "took" : 63, # 花费时间毫秒数
  "timed_out" : false, #是否超时
  "_shards" : {  # 告诉我们有多少分片被查询，多少成功，多少失败
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : { 
    "total" : 1000, # 匹配数据总数
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],  # 排序字段
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

### 分词

#### 查看具体字段分词情况 `[index]/[type]/[id]/_termvectors?fields=[field]`

```
GET /bank/account/25/_termvectors?fields=address
```

## 倒排索引

倒排索引：单词到文档ID的关系![IMG_6393](assets/IMG_6393.PNG)

```shell
POST _analyze
{
  "analyzer": "standard",
  "text": "Mastering Elasticsearch"
}

POST _analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch Server"
}

POST _analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch Essentials"
}

```

## Analysis 与 Analyzer

![IMG_6394](assets/IMG_6394.PNG)

Analyzer 由三部分组成

- Character Filters 针对原始文本处理
- Tokenizer 按照规则切分单词
- Token Filter 对切分后的单词进行加工，小写，删除 stopword等

内置分词器

- Standard Analyzer – 默认分词器，按词切分，小写处理
- Simple Analyzer – 按照非字母切分（符号被过滤），小写处理
- Stop Analyzer – 小写处理，停用词过滤（the，a，is）
- Whitespace Analyzer – 按照空格切分，不转小写
- Keyword Analyzer – 不分词，直接将输入当作输出
- Patter Analyzer – 正则表达式，默认 \W+ (非字符分隔)
- Language – 提供了30多种常见语言的分词器

中文分词器

- ICU Analyzer  – 提供了 Unicode 的支持，更好的支持亚洲语言，`elasticsearch-plugin install analysis-icu` 
- IK – 支持自定义词库，支持热更新分词字典
- THULAC – 清华大学自然语言处理提供的分词器