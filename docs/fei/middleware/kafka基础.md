# kafka基础

## 基本概念

* 消息`record`
* 批次`batch`
* 主题`topic`
* 分区`partition`
* 生产者`producer`
* 消费者`consumer`
* 消费群组`consumer group`
* 偏移量`consumer offset`
* 独立的`kafka`服务器`broker`
* 副本`replica`
* 重平衡`rebalance`

## `producer`发送消息到`broker`

1. 构建`record`
2. 序列化器，序列化`record`中的键值对
3. 分区器，决定将`record`发往指定`topic`的某个`partition`中，并缓存在同`topic`同`partition`的缓冲区上
4. 由独立线程将缓冲区上的`record`分`batch`发到`broker`中

## 发送方式

* 简单发送(不在意发送结果)
* 同步发送
* 异步发送

## 分区机制

* 顺序轮询(默认策略)
* 随机轮询
* 按照`key`进行分区，同一个`key`能分到同一个分区

## 消费方式

* 点对点模式(消息队列)，一个消费群组消费一个主题中的消息
* 发布订阅模式，一个主题中的消息被多个消费群组共同消费

## `consumer`消费`broker`中的消息

* 一个群组中的消费者订阅的都是相同的主题，每个消费者消费主题一部分分区的消息
* 相同主题的一个分区在一个消费群组中只能由一个消费者消费，多余的消费者会闲置
* 每个消费群组都能从`broker`中读取全量消息，在同一个消费群组中添加消费者，能增加该群组的消费能力

## 重平衡机制

* 对一个主题中的分区应该被消费群组中的哪个消费者消费进行重新分配

> 重平衡会导致整个消费群组都停止消费

## 提交与偏移量

消费者把分区的偏移量提交到特殊主题`_consumer_offset`中，用于重平衡后，即每个消费者分配到了新的分区也能够继续正确消费消息

* 提交给`_consumer_offset`的偏移量`<`消费者最后消费的偏移量，那么重平衡后两个偏移量之间的消息就会被`重复消费`
* 提交给`_consumer_offset`的偏移量`>`消费者最后消费的偏移量，那么重平衡后两个偏移量之间的消息就将会`丢失`
