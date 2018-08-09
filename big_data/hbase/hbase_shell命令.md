规约

> 存在Namespace，则使用`<namespace>:<tb>`

> tb:表名，cf:列族名，rk:行键，value:值，ts:时间戳

表结构信息查看

```shell
describe '<tb>' 
```

创建表

```shell
# create 表名 列族名...
create '<tb>' 'cf'
```

添加数据

```shell
put '<tb>', '<rk>', '<cf>', '<value>', <ts> 
```

获取数据

```shell
# 顺序就是操作关键词后跟表名，行名，列名这样的一个顺序，如果有其他条件再用花括号加上
get <'tb'>, <'rk'> 
get <'tb'>, <'rk'>, {TIMERANGE => [ts1, ts2]} 
get <'tb'>, <'rk'>, {COLUMN => <'cf'>} 
get <'tb'>, <'rk'>, {COLUMN => ['cf', 'cf', 'cf']} 
get <'tb'>, <'rk'>, {COLUMN => <'cf'>, TIMESTAMP => ts1} 
get <'tb'>, <'rk'>, {COLUMN => <'cf'>, TIMERANGE => [ts1, ts2], VERSIONS => 4} 
get <'tb'>, <'rk'>, {COLUMN => <'cf'>, TIMESTAMP => ts1, VERSIONS => 4} 
get <'tb'>, <'rk'>, <'cf'> 
get <'tb'>, <'rk'>, <'cf'>, <'cf'> 
get <'tb'>, <'rk'>, ['cf', 'cf']
```

扫描所有数据

```shell
# 可以指定一些修饰词：TIMERANGE, FILTER, LIMIT, STARTROW, STOPROW, TIMESTAMP, MAXLENGTH,or COLUMNS
scan '<tb>'
```

删除数据

```shell
delete '<tb>' '<rk>' '<c1>' '<ts>'
```

删除表

```shell
disable '<tb>'
drop '<tb>'
```

修改表

```shell
disable '<tb>'
alter '<tb>',NAME=>'<cf>'
enable '<tb>'
```

