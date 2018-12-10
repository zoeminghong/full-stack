## 延迟操作组件

Kafka将一些**不立即执行，需要满足一定条件之后才触发**的操作称为延迟操作。

### DelayedOperation

DelayedOperation 就是实现延迟操作的抽象类。DelayedOperation 是一个有失效时间的TimerTask。TimerTask 实现了 Runable 接口，维护了一个 TimerTaskEntity 对象，TimerTaskEntity 被添加到 TimerTaskList 中，而 TimerTaskList 是一个环形双向链表，按失效时间排序。

![IMG_5138 (assets/IMG_5138 (1).JPG)](assets/IMG_5138 (1).JPG)

DelayedOperation 中只有一个 AtomicBoolean 类型的 compeleted 属性。用于保证延迟操作（onComplete()）只被执行一次。

tryComplete方法：负责检测执行条件是否满足。

forceComplete方法：在条件满足时，检测延迟任务是否未被执行。若未被执行，则先调用TimerTask.cancle() 方法解除该延迟操作与 TimerTaskEntity 的绑定，将该延迟操作从 TimerTaskList 链表中移除，然后调用 onComplete() 方法让延迟操作执行完成。通过completed 的 CAS 原子操作，可以保证只有第一个调用线程可以顺利调用 onComplete() 完成延迟操作，其他线程 compeleted=false。

onComplete方法：实际的业务逻辑。

safeTryComplete方法：以synchronized 同步锁调用 onComplete方法，供外部调用。

onExpiration方法：由子类来实现当延迟操作已达失效时间的相应逻辑处理。

![IMG_5139 (assets/IMG_5139 (1).JPG)](assets/IMG_5139 (1).JPG)

### DelayedOperationPurgatory

DelayedOperationPurgatory 是一个作为对 DelayedOperation 管理的辅助类。其内部有一个 watchersForKey 的对象，是一个 Pool[Any,Watcher] 类型，Watcher 是 DelayedOperationPurgatory 的内部类，其内部存在一个 ConcurrentLinkedQueue 队列 operations 属性，用于保存 DelayedOperation 。同时，SystemTask 中也存在 DelayedOperation。也就是说，同一个 DelayedOperation 会存在于两个地方，一个是 DelayedOperationPurgatory 的 Watcher 和 SystemTask。

> SystemTask 是 Kafka 实现的底层基于**层级时间轮和 DelayQueue 定时器**。

**DelayedOperationPurgatory 是如何保证 DelayedOperation 的执行呢？**

会多次调用 `DelayedOperation.safeTryComplete()` 方法判断是否任务已经完成。若完成则返回true，从 SystemTask 和 watchersForKey 中移除，如果没有完成，将其再次放入 SystemTask 中。

**如何清理 watchersForKey 中的 key？**

DelayedOperationPurgatory 在启动时会同时启动一个 ExpiredOperationReaper 线程，该线程除了推进时间轮的指针外还会定期清理 watchersForKey 已完成的 DelayedOperation。

### DelayedProduce

DelayedProduce 消息写入 Leader 副本时使用。帮助 ReplicaManager 完成延迟操作。

> ReplicaManager 负责将生产者发送的消息写入 Leader 副本、管理 Follower 副本与 Leader 副本之间的同步以及副本的角色转换。

**就是协助副本管理器在 acks 为-1的场景时，延迟回调 responseCallback 向生产者做出相应。**

#### ProduceMetadata

- ProduceRequiredAcks：记录本次 ProduceRequest 的 ack 信息
- ProduceParitionStatus：对应分区对消息追加处理结果信息
  - requiredOffset：本次追加消息的最大偏移量
  - ParitionResponse：分区处理结果。
    - errorCode：结果码
    - baseOffset：消息写入日志段的基准偏移量
    - timestamp：消息追加的时间戳
  - acksPending：标识是否还在进行数据同步的Boolean类型，副本同步完成后值为false。

#### 发生场景

- 写操作异常，更新 errorCode 和 acksPending=false
- 当分区 Leader 副本发生迁移。更新 ProduceParitionStatus 和 acksPending=false
- ISR 副本同步完成，Leader 副本的 HW 已大于 requireOffset

### DelayedFetch

DelayedFetch 在 FetchRequest 处理时进行的延迟操作。在Kafka中只有消费者或是 Follower 副本会发起 FetchRequest。

#### FetchMetadata

- 本次拉取操作获取的最小字节数字段
- 本次拉取操作获取的最大字节数字段
- 是否只从 Leader 副本读取
- 是否只读 HW 之前的数据的标志字段
- replicaId：用来标识是消费者还是Follower 副本
- fetchParitionsStatus：记录本次从每个分区拉取结果

**之所以拉取消息时需要延迟操作，是为了让本次拉取消息获取足够的数据。**

#### 发生场景

- 发生异常，Leader 副本发生迁移
- 拉取消息的分区不存在了
- 日志段发生了切割，请求拉取的消息偏移量已不在活跃段内，同时 Leader 副本没有处在限流处理的状态
- 累积拉取的消息数已超过了最小字节数限制

### DelayedJoin

在组协调器的 prepareRebalance 方法中会创建一个 DelayedJoin 对象。

之所以需要 DelayedJoin 是为了让组协调器等待当前消费组下所有的消费者都请求加入消费组。

#### 发生场景

所有的消费者均已申请加入消费组

### DelayedHeartbeat

用于协助消费者与组协调器心跳检测相关的延迟操作。

#### MemberMetadata

- 心跳 Session 超时时间
- 上一次更新心跳的时间戳
- 成员所支持的协议
- 组的状态信息

#### 发生场景

- `memeber.awaitingJoinCallback` 不为空。awaitingJoinCallback 不为空，则表示消费者已经发出 JoinGroupRequest ，正在等待组协调器返回结果。
- `memeber.awaitingSyncCallback` 不为空，表示正在进行 SyncGroupRequest 处理
- 上一次更新心跳的时间戳与 `member.sessionTimeoutMS` 之和大于 heartbeatDeadline。
- 消费者已离开消费组。

### DelayedCreateTopic

创建主题时，等待该主题的所有分区副本分配到 Leader 或是等待超时后调用回调函数返回给客户端。







