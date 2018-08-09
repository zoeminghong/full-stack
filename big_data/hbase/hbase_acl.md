HBase ACL 可以实现不同的用户、Group与Namespace、Table、ColumnFamily层级的数据权限控制

## 基本概念

**某个范围(Scope)的资源** 

| 范围         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| Superuser    | 超级账号可以进行任何操作，运行HBase服务的账号默认是Superuser。也可以通过在hbase-site.xml中配置hbase.superuser的值可以添加超级账号 |
| Global       | Global Scope拥有集群所有table的Admin权限                     |
| Namespace    | 在Namespace Scope进行相关权限控制                            |
| Table        | 在Table Scope进行相关权限控制                                |
| ColumnFamily | 在ColumnFamily Scope进行相关权限控制                         |
| Cell         | 在Cell Scope进行相关权限控制                                 |

**操作权限**

| 操作          | 说明                                            |
| ------------- | ----------------------------------------------- |
| Read ( R )    | 读取某个Scope资源的数据                         |
| Write ( W )   | 写数据到某个Scope的资源                         |
| Execute ( X ) | 在某个Scope执行协处理器                         |
| Create ( C )  | 在某个Scope创建/删除表等操作                    |
| Admin ( A )   | 在某个Scope进行集群相关操作，如balance/assign等 |

**实体** 

| 实体  | 说明             |
| ----- | ---------------- |
| User  | 对某个用户授权   |
| GROUP | 对某个用户组授权 |

## 基本操作

- HBase

```shell
# hbase shell
# 查看存在哪些表
list

# 创建表
create '表名称', '列名称1','列名称2','列名称N'

# 添加记录
put '表名称', '行名称', '列名称:', '值'

# 查看记录
get '表名称', '行名称'

# 查看表中的记录总数
count '表名称'

# 删除记录
delete '表名' ,'行名称' , '列名称'

# 删除一张表
先要屏蔽该表，才能对该表进行删除，第一步 disable '表名称' 第二步 drop '表名称'

# 查看所有记录
scan "表名称"

# 查看某个表某个列中所有数据
scan "表名称" , ['列名称:']

# 更新记录
就是重写一遍进行覆

# 退出 HBASE SHELL环境
exit
```

- Namespace

```shell
# hbase shell
# 创建namespace
create_namespace 'ns'

# 删除namespace
drop_namespace 'ns'

# 查看namespace
describe_namespace 'ns'

# 列出所有namespace
list_namespace

# 在namespace下创建表
create 'ns:test_table', 'fm1'

# 查看namespace的表
list_namespace_tables 'ns'
```

## Quick Start

- HBase服务配置

**hbase-site.xml**

```xml
<property>
     <name>hbase.security.authorization</name>
     <value>true</value>
</property>
<property>
     <name>hbase.coprocessor.master.classes</name>
     <value>org.apache.hadoop.hbase.security.access.AccessController</value>
</property>
<property>
     <name>hbase.coprocessor.region.classes</name>
 <value>org.apache.hadoop.hbase.security.token.TokenProvider,org.apache.hadoop.hbase.security.access.AccessController</value>
</property>
<property>
  <name>hbase.coprocessor.regionserver.classes</name>
  <value>org.apache.hadoop.hbase.security.access.AccessController</value>
</property>
```

> HBase的所有节点服务的配置都要进行添加

- 重启HBase服务
- 服务器用户配置

```shell
# 创建用户
adduser <userName>
# 复制hbase-config.sh文件
cp bin/hbase-config.sh /home/<userName>/hbase-config.sh
```

> 下方使用test用户名进行讲解说明

- 设置权限

```shell
# 进入hbase环境
hbase shell

# 创建namespace
create_namespace 'ns'

# 创建表
create 'ns:test_table', 'fm1'

# 给 test 用户授于ns:test_table表RW权限
grant 'test','RW','ns:test_table' (1)

# 查看权限
user_permission 'ns:test_table'
```

1. 单纯的namespace需要添加@前缀，user/group的授权方式一样，group需要加一个前缀@

```shell
grant <user> <permissions> [<@namespace> [<table> [<column family> [<column qualifier>]]]

# 权限回收所有
revoke 'test'

# 权限回收指定内容
revoke 'test','ns:test_table'
```

- 校验权限

```shell
# 进入hbase环境
# sudo -u test hbase shell
su - test hbase shell

# 查看HBase表列表
list

# 或者使用test账号登录服务器进行查看
```

## 深入讲解

HBase ACL是基于Linux环境的用户体系，进行对HBase环境权限的控制。上方提到的 `test` 用户其实就是Linux系统中的用户。HBase管理员为某个租户提供资源权限的时候， 需要先为其创建一个Linux系统的账号，再用管理员的权限为该账号进行权限操作。

## 可能会出现的问题

- `hbase-config.sh:No such file or directory`

只需将 `hbase-config.sh` 文件copy一份到提示的目录下即可

- `sudo -u` 命令无法执行

使用 `su -` 方式

- 配置 `hbase-site.xml` 不起作用

检查是否HBase服务没有重启，或者是否HBase所有节点都进行了配置

## 参考文献

<http://zhangxiong0301.iteye.com/blog/2244570>

<https://www.cloudera.com/documentation/enterprise/5-4-x/topics/cdh_sg_hbase_authorization.html#concept_enm_hhx_yp>

<http://debugo.com/hbase-access-control/>

<https://community.pivotal.io/s/article/How-to-control-user-s-access-to-HBASE>

https://www.alibabacloud.com/help/zh/doc-detail/62705.htm