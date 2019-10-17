# Kudu

主键: 每个表都必须有一个Unique主键, 主键可以是单独字段也可以由多个字段组成, 但主键中的每个字段都应该是non null, 而且不能是boolean和浮点类型字段. 重复的PK的记录将无法Insert到Kudu表中, 而且主键值是不能修改的. Kudu主键是聚集索引, 也就是说每个tablet中的记录将按照主键排序存储, 在scan时候能利用这些排序加快记录查找。
通过Kudu API更新/删除记录必须提供主键, 通过Impala查询引擎没有这个限制。

注意点：

1. 在建表DDL语句中, 主键要放到字段清单的前段.
2. 除了主键, kudu不支持建立其他索引
3. 主键值不能被update
4. insert语句中, 主键名是大小写敏感的, 其他语句大小写不敏感.

https://blog.csdn.net/xueyao0201/article/details/80874583

