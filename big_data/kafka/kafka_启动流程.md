## Kafka启动

启动kafka使用其自带的kafka-server-start.sh脚本。该脚本调用kafka.Kafka类

KafkaServer启动的工作时由KafkaServer.startup()来完成的，在此过程中将完成任务调度器、日志管理器、网络通信服务器、副本管理器、控制器、组协调器。动态配置管理器以及kafka健康状况检查等服务的初始化。

### 初始化流程

首先实例化用于限流的QuotaManagers，这些Quota会在后续其他组件实例化时作为入参注入当中，代理状态为starting，即开始代理。

| 状态名                        | 状态值 | 描述                                                         |
| ----------------------------- | ------ | ------------------------------------------------------------ |
| NotRunning                    | 0      | 代理未启动                                                   |
| Starting                      | 1      | 代理正在启动中                                               |
| RecoveringFromUncleanShutdown | 2      | 代理非正常关闭，在${log.dir}配置的每一个路径下存在.kafka_cleanshutdown文件 |
| RunningAsBroker               | 3      | 代理已正常启动                                               |
| PendingControllerShutdown     | 6      | KafkaController被关闭                                        |
| BrokerShuttingDown            | 7      | 代理正在准备关闭                                             |

启动任务调度器KafkaSchedule，基于ScheduledThreadPoolExecutor实现的，在KafkaServer启动时，会构造一个线程总数为${background.threads}的线程池，该配置项默认为10，每个线程的线程名以“kafka-schedule-”为前缀，后面连接递增的序列号，这些线程在KafkaServer启动的时候运行，负责副本管理及日志管理调度等。

创建与zookeeper的连接，检查并在zookeeper中创建元数据的目录节点，如目录不存在则创建相应的目录。在KafkaServer启动时，会保证以下目录正常

| 目录路径                 | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| /consumers               | 旧版消费者启动后会在zookeeper的该节点路径下创建一个消费组的节点 |
| /brokers/seqid           | 当用户没有配置broker.id时，zookeeper会自动生成全局唯一的id   |
| /brokers/topics          | 每创建一个主题时就会在该目录下创建一个与主题同名的节点       |
| /brokers/ids             | 当Kafka每启动一个KafkaServer时会在该目录创建一个名为${broker.id}的子节点 |
| /config/topics           | 存储动态修改主题级别的配置信息                               |
| /config/clients          | 存储动态修改客户端级别的配置信息                             |
| /config/changes          | 动态修改配置时存储相应的信息                                 |
| /admin/delete_topics     | 在对主题进行删除操作时保存待删除主题信息                     |
| /cluster/id              | 保存集群id信息                                               |
| /controller              | 保存控制器对应的brokerId信息等                               |
| /isr_change_notification | 保存Kafka副本ISR列表发生变化时通知的相应路径                 |

生成一个uuid，经过base64处理得到的值作为Cluster的id，调用kafka实现的org.apache.kafka.common.ClusterResourceListener通知集群元数据信息发生变更操作。此时生成的Cluster的id信息会写入/cluster/id

实例化并启动日志管理系统，LogManger负责日志的创建、读写、检索、清理等操作

实例化并启动SocketServer服务

实例化并启动副本管理器。副本管理器负责管理分区副本，他依赖于任务调度器和日志管理器，处理副本消息的添加与读取的操作以及副本数据同步等操作

实例化并启动控制器。每个代理对应一个KafkaController实例，同时会实例化分区状态机、副本状态机和控制器选举器ZookeeperLeaderElector，在KafkaController启动后，会从KafkaController中选出一个节点作为Leader控制器，主要负责分区和副本状态的管理、分区重分配、当新创建主题时调用相关方法创建分区等

实例化并启动组协调器GroupCoordinator。Kafka会从代理中选出一个组协调器，对消费者进行管理，当消费者或者订阅的分区主题发生变化时进行平衡操作

实例权限认证组件以及Handler线程池

实例化动态配置管理器。注册监听zookeeper的/config路径下各个节点信息变化

实例化并启动kafka健康状态检查。Kafka健康检查机制主要是在zookeeper的/brokers/ids路径下创建一个与当前代理的id同名的节点，该节点也是一个临时节点。当代理离线时，该节点会被删除，其他代理或者消费者通过判断/brokers/ids路径下是否存在某个代理的brokerId来确定该代理的健康情况

向meta.properties文件中写入当前代理的id已经固定的verion=0信息

注册Kafka的metrics信息，在KafkaServer启动时将一些动态的JMX Beans进行注册，以便于对Kafka进行跟踪监控

最后将当前代理状态转为RunningAsBroker

使用jps命令可以看到名为kafka的进程