## 分区副本分配

分区副本可以在创建主题的时候指定分配方案，也可以采用Kafka默认的分区副本分配策略创建副本分配方案。

分配策略通过随机数方式来确定第一个副本的位置，以及第二次轮询相对前一次分配的移位量是为了尽可能的把分区副本均匀分布。实现各代理负载均衡的效果。

客户端创建分区副本方案

```shell
./kafka-topic.sh --create --zookeeper server-1:2181,server-2:2181,server-3:2181 --replication-factor 3 --partitions 6 --topic replica-topic
```

通过在zookeeper中使用如下命令获取分区分配方案

```shell
get /brokers/topics/<topic名>
```

每个分区中的首个副本，被称为优先副本。Kafka会保证优先副本会被均匀分布到集群所有的代理节点上，刚创建的主题一般会选择优先副本作为分区的Leader，这样一个主题的所有分区的Leader被均匀的分布到集群中了。

因而推荐`paritionNum % brokerNum=0`进行配置，同时副本数要小于节点数（brokers）

