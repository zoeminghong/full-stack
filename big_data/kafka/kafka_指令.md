分区重分配

```shell
kafka-reasign-partitions.sh --zookeeper localhost:2181/kafka --generate --topics-to-move-json-file reason.json --broker-list 0,2

kafka-reasign-partitions.sh --zookeeper localhost:2181/kafka --execute --reassignment-json-file reason.json
```

优先副本选举

```shell
kafka-preferred-repalica-election.sh --zookeeper localhost:2181/kafka --path-to-json-file election.json
```

