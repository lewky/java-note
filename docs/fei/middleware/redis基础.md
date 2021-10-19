# Redis基础

传统数据库将数据存储在磁盘，而非关系型数据库`Redis`将数据存储在内存，因此`Redis`读取性能高

## 线程模型

`Redis`内部使用文件事件处理器`file event handler`，而这个文件事件处理器是单线程的，它采用IO多路复用机制同时监听多个`Socket`，根据`Socket`上的事件来选择对应的事件处理器进行处理

## 数据类型

* 键`key`
  * `strings`

* 值`value`
  * `strings`
  * `lists`
  * `hashes`
  * `sets`
  * `sorted sets`

## 持久化

`snapshotting`，`RDB`，在一段时间内有n个key发生变化才会进行快照，因此系统发生崩溃，`Redis`会丢失最近一次快照之后的所有新存储的数据

`append-only file`，`AOF`，每一条改变`Redis`的命令都会持久化到`AOF`文件中，但是生成的`AOF`文件体积大

> 重写/压缩`AOF`文件，移除`AOF`中冗余的命令以减少`AOF`的体积，可以用`redis.conf`中的`auto-aof-rewrite-percentage`和`auto-aof-rewrite-min-size`来控制何时自动进行`AOF`重写/压缩

## 事务

命令打包请求，命令依次执行，`redis`将事务中的命令执行完毕后才会执行其他客户端的命令请求

## 过期时间

定时删除，每100ms随机检查设置过期时间的`key`是否过期，过期则删除

惰性删除，查询`key`，如果`key`已过期则删除

> 如果定期删除导致很多过期的`key`没有被删除，而又没有被查询，则会有大量过期的`key`堆积在内存里，导致`redis`内存耗尽

## 内存淘汰机制

`redis.conf`中的`maxmemory-policy`提供若干种数据淘汰策略，来解决大量`key`堆积在内存中导致内存耗尽问题

## 缓存雪崩

`redis`中存储的`key-value`值大面积失效，请求直接落在数据库，造成数据库短时间内承受大量请求导致数据库崩溃

## 缓存穿透

大量请求的`key`根本不存在，导致请求直接落到数据库上
