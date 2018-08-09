## 配置文件

### config

`server.properties`

包含broker.id，port，zookeeper、日志文件路径等信息内容。

```shell
port=9092
host.name=10.200.162.16
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/data/kafka/logs
num.partitions=5
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=3
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=test1.org:2181,test2.org:2181,test3.org:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
auto.create.topics.enable=true
delete.topic.enable=true
```

### log

`meta.properties`

记录与当前Kafka版本对应的verison，如果是当前版本则固定值为0，还记录了当前代理的broker.id

```shell
version=0
broker.id=1
```

## 应用

### 不改变log路径配置，而修改代理的broker.id时

1. 修改代理对应的`server.properties`文件的broker.id的值
2. 修改log路径目录下`meta.properties`文件的broker.id值。若log路径存在多个，则要分别修改log目录下的`meta.properties`文件的broker.id

### 主题分区副本分配方案记录

```shell
/brokers/topics/<主题>/paritions/<分区编号>/state

/brokers/topics/kafka-topic/paritions/0/state
```

### 主题命名规则

长度不超过249个字母、数字、着重号(.)、下划线、`连接号(_)`的字符组成。正则表达式 `[a-zA-Z0-9\\._\\-]+`。

不允许只有着重号组成

> 主题最好不要有着重号与下划线，防止与kafka一些指标字段冲突

### 客户端创建分区副本方案

```shell
./kafka-topic.sh --create --zookeeper server-1:2181,server-2:2181,server-3:2181 --replication-factor 3 --partitions 6 --topic replica-topic
```



