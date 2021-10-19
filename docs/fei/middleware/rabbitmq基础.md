# RabbitMQ基础

## 处理流程

1. `Producer`将`Message`传给`Exchange`，这时会指定一个`routing key`
2. `Exchange`跟`Queue`的连接中，会有`binging key`
3. `Exchange`会根据`Exchange Type`的匹配规则将`binging key`与`routing key`进行匹配，从而将符合匹配规则的`Message`加入到指定的`Queue`中
4. `Consumer`将消费`Queue`中的`Message`，处理完成后会返回`ack`，如果中途断开连接，`Queue`就会再把该`Message`给`Consumer`进行处理

## `Exchange Type`的四种类型

* `fanout`
* `direct`
* `topic`
* `headers`
