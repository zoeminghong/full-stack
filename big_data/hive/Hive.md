# Hive

## 操作

```shell
# shell命令
hive

# 查看数据库
show databases;
# 正则方式
show databases like 'h.*';

# 使用指定数据库
use test;

# 查看该数据库中的所有表
show tables; 

# 创建表
creat database database_name location '路径'; 

# 查看table的存储路径
show create table tablename;  

# 创建表
create table test(userid string);
# 载入数据
LOAD DATA INPATH '/tmp/p10pco2a.dat' INTO TABLE weather.weather_everydate_detail;

# 查看表有哪些分区 
show partitions t1;

# 修改表名
alter table table_name rename to another_name;   

# 删除空的数据库
drop database if exists database_name; 

# 表结构
desc test;

# 查询
select * from test.test_1;

```

