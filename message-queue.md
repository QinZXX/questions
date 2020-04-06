# 消息队列

## 消息队列的用途

1. 解耦
2. 冗余，因为避免数据处理失败而导致数据丢失的风险
3. 扩展性
4. 异步
5. 削峰

## 主流的消息队列

1. kafka
2. rocketmq
3. redis
4. zeromq

# kafka

参考：

1. http://www.jasongj.com/2015/03/10/KafkaColumn1/
2. https://www.cnblogs.com/huxi2b/p/6223228.html

## kafka 是什么

kafka 是一个分布式的，基于发布订阅的消息系统，主要设计目标为

1. 以常数时间复杂度提供消息持久化能力，即使对 TB 级以上的数据也能保证常数时间访问。
2. 高吞吐率
3. 支持 kafka server 间的消息分区，及分布式消费，同时保证每个 partition 内的消息顺序传输
4. 支持在线水平扩展

## kafka 架构

- producer 消息生产者
- broker 传播者
  - topic
  - partition
- zookeeper 保存元数据
- consumer

## topic 和 partition 的关系

topic 在逻辑上可以被认为是一个 queue，每条消费必须指定她的 topic，为了使 kafka 的吞吐率可以线性提高，物理上把 topic 分成一个或多个 partition，每个 partition 物理上对应一个文件夹，该文件夹下存储这个 partition 的所有消息和索引文件。

## kafka 为什么性能高

1. 顺序写磁盘
2. broker 不保存消息消费的状态，因此访问 broker 不需要加锁，没有竞争

## kafka 删除策略

1. 基于时间删除
2. 基于 partition 文件大小删除

## kafka 中消息 key 的作用，如果不指定 key 会发生什么情况

决定消息存放在哪一个 partition 中。

## 消息怎么被消费

同一个 topic 的一条消息只能被同一个 consumer group 内的同一个 consumer 消费，但多个 consumer 可同时消费这一消息。

## kafka 是 consumer group 是怎么避免 consumer 重复消费的

## 对消息安全，kafka 做了哪些保证

1. `at most once` 消息可能会丢失，但不会重复消费
2. `at least once` 消息绝不会丢，但可能重复
3. `exactly once` 每条消息肯定会被传输且仅传输一次

## kafka 怎么实现高可用的

producer 和 consumer 只同 leader 交互，leader 会将消息通过复制的方式备份到 replication 中，当 leader 崩溃，replication 通过选举的方式升级为新的 leader。

## kafka 分配 replica 算法

1. 将 broker (假设共有 n 个 borker) 和待分配的 partition 排序，
2. 将第 i 个 partition 分配到第 (i mod n) 个 broker 上
3. 将第 i 个 partition 的第 j 个 replica 分配到 (i + j) mod n 个 boker 上

## kafka 消息发送的流程

1. producer 通过 metadata 找到 leader 节点，并向其发送信息
2. leader 接受到消息，并将其写入本地 log，同时每个 replica 会向 leader pull 数据，follower 在收到消息并写入其 log 后，向 leader 发送 ack。一旦 leader 收到 ISR 中所有 replica 的 ACK，该消息就被认为 commit 完毕，leader 将增加 HW，并且向 producer 发送 ACK。

## kafka 的 leader 选举算法

## 所有的 broker 节点都挂掉，之后的恢复策略


