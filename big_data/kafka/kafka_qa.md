## QA

### Kafka Offset 无法进行重置

**现象**

![kafka_qa_001](assets/kafka_qa_001.png)

**原因**

kafka需要进行多节点数据的同步

**解决方案** 

停止相应的consumer组，同时等待一段时间之后，再次执行该命令