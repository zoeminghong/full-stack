15课

[https://github.com/onebirdrocks/geektime-ELK/blob/master/part-1/3.7-URISearch%E8%AF%A6%E8%A7%A3/README.md](https://github.com/onebirdrocks/geektime-ELK/blob/master/part-1/3.7-URISearch详解/README.md)

# Search

Search API

- URI Search：在 URL 中使用查询参数
- Request Body Search：使用 ES 提供的，基于 `JSON` 格式的更加完备的 `DSL` 语法。

查询索引

```
/index1/_search
/index1,index2/_search
/index*/_search
```

URI 查询

```
# q 表示查询内容
# kv方式表示查询条件
index1/_search?q=customer_id:xha234
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s

GET /movies/_search?q=2012&df=title
{
	"profile":"true"
}
```

- q 指定查询语句
- df 默认字段，不指定时，会对所有字段进行查询，用于限定查询
- Sort 排序 
- from 和 size 用于分页
- Profile 可以查看查询是如何被执行的

Query String Syntax

指定字段 VS 泛查询

```
q=title:2012 # 指定字段
q=2012 #泛查询

# 查找美丽心灵, title:Beautiful，但Mind为泛查询
GET /movies/_search?q=title:Beautiful Mind
{
	"profile":"true"
}
```

Term VS Phrase

- Beautiful Mind 等效于 Beautiful OR Mind
- “Beautiful Mind” 等效于 Beautiful AND Mind，包括顺序

分组与引号

- title:(Beautiful AND Mind)

- title="Beautiful Mind"

```
#分组，Bool查询
GET /movies/_search?q=title:(Beautiful Mind)
{
	"profile":"true"
}
```

布尔操作

- AND / OR / NOT 或者 && / || / !
  - 必须大写
  - title:(matrix NOT reloaded)

分组

- `+` 表示 must
- `-` 表示 must_not
- title:(+matrix -reloaded)

范围查询

- 区间查询：[] 闭区间，{} 开区间
  - year:{2019 TO 2018}
  - year:[* TO 2018]

算术符号

- year:>2010
- year:(>2010 && <=2018)
- year:(+>2010 +<=2018)

通配符查询（占用内存大，不建议使用。特别是放在最前面）

- ? 代表 1 个字符，* 代表 0 或多个字符
  - Title:mi?d

正则表达式

- title:[bt]oy

模糊匹配与近似查询

- title:befutiful~1

```
//模糊匹配&近似度匹配
GET /movies/_search?q=title:beautifl~1
{
	"profile":"true"
}
```

Request Body

```
POST /movies,404_idx/_search?ignore_unavailable=true
{
  "profile": true,
	"query": {
		"match_all": {}
	}
}
```

分页

```
POST /kibana_sample_data_ecommerce/_search
{
  "from":10,  #页码
  "size":20, # 页Size
  "query":{
    "match_all": {}
  }
}

```

_source filter

只关心自己希望查询出来的字段值。使用 `_source` 即可。

- 支持通配符，`"_source":["order_dat*"]`

```
#source filtering
POST kibana_sample_data_ecommerce/_search
{
  "_source":["order_date"],
  "query":{
    "match_all": {}
  }
}
```

排序

```
#对日期排序
POST kibana_sample_data_ecommerce/_search
{
  "sort":[{"order_date":"desc"}],
  "query":{
    "match_all": {}
  }

}
```

脚本字段

在 ES 中通过 painless 语句从已有字段值中根据一定的规则，生成一个新的字段。

```
#脚本字段
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'_hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
}
```

使用查询表达式— Match

```
POST movies/_search
{
  "query": {
    "match": {
      "title": "last christmas"  # last 或者 christmas
    }
  }
}

POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "last christmas",  # last 且 christmas
        "operator": "and"
      }
    }
  }
}
```

Match Phrase

查询条件要求顺序都是一样的。

- slop 允许中间有一个其他的字符

```
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love"

      }
    }
  }
}

POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love",
        "slop": 1  # 允许有中间有一个其他的字符

      }
    }
  }
}
```

## ES 相关性

ES 5.0 之前相关性算分采用 TF-IDF，5.0之后采用 BM25。

<img src="assets/image-20191011222500753.png" alt="image-20191011222500753" style="zoom:30%;" />

<img src="assets/image-20191011222556061.png" alt="image-20191011222556061" style="zoom:30%;" />

```shell
POST /testscore/_search
{
  //"explain": true,
  "query": {
    "match": {
      "content":"you"
      //"content": "elasticsearch"
      //"content":"the"
      //"content": "the elasticsearch"
    }
  }
}
```

开启 `explain=true` 可以知道是怎么打分的，默认相关度高的数据在搜索结果中排在前面。

```shell
POST testscore/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "content" : "elasticsearch"
                }
            },
            "negative" : {
                 "term" : {
                     "content" : "like"
                }
            },
            "negative_boost" : 0.2
        }
    }
}
```

Boost 是控制相关度的一种手段

- 索引，字段或查询子条件

参数 Boost 的含义

- 大于 1 ，打分相关度相对性提升
- 大于 0 小于 1，打分权重相关性降低
- 小于 0 ，贡献负分 