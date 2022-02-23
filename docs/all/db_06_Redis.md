<!--
date: 2022-02-22T22:34:12+08:00
lastmod: 2022-02-22T22:34:12+08:00
-->
## Redis

Redis是非关系型数据库（NoSQL）的一种，本质上是一个Key-Value类型的内存数据库（使用C语言编写的）。由于是纯内存操作，因此速度很快。

Redis的键类型只能是String，值类型则支持五种数据类型。

Redis支持很多特性，如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。

## 数据类型

### String

字符串，是Redis最基本的类型，且是二进制安全的，即可以存储任何数据，如字符串、整数、浮点数、JPG图片或者序列化的对象，最大能存储512MB。

* [二进制安全字符串](https://javanote.doc.lewky.cn/#/all/basic_02_String?id=%e4%ba%8c%e8%bf%9b%e5%88%b6%e5%ae%89%e5%85%a8%e5%ad%97%e7%ac%a6%e4%b8%b2%ef%bc%88binary-safe-strings%ef%bc%89)

支持如下操作：

1）对整个字符串或者字符串的其中一部分执行操作<br>
2）对整数和浮点数执行自增或者自减操作

客户端命令如下：

```
> set hello world
OK
> get hello
"world"
> del hello
(integer) 1
> get hello
(nil)
```

### Hash

哈希（散列表），是一个String类型的键值对的无序集合。

支持如下操作：

1）添加、获取、移除单个键值对<br>
2）获取所有键值对<br>
3）检查某个键是否存在

客户端命令如下：

```
> hset hash-key sub-key1 value1
(integer) 1
> hset hash-key sub-key2 value2
(integer) 1
> hset hash-key sub-key1 value1
(integer) 0

> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"

> hdel hash-key sub-key2
(integer) 1
> hdel hash-key sub-key2
(integer) 0

> hget hash-key sub-key1
"value1"

> hgetall hash-key
1) "sub-key1"
2) "value1"

> HMSET runoob field1 "Hello" field2 "World"
"OK"
> HGET runoob field1
"Hello"
> HGET runoob field2
"World"
```

### List

String类型的列表，按照插入顺序排序。

支持如下操作：

1）从两端压入或者弹出元素<br>
2）对单个或者多个元素进行修剪<br>
3）只保留一个范围内的元素

客户端命令如下：

```
> rpush list-key item
(integer) 1
> rpush list-key item2
(integer) 2
> rpush list-key item
(integer) 3

> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"

> lindex list-key 1
"item2"

> lpop list-key
"item"

> lrange list-key 0 -1
1) "item2"
2) "item"
```

### Set

String类型的无序集合，通过哈希表实现，所以添加，删除，查找的复杂度都是O(1)。不允许重复元素。

支持如下操作：

1）添加、获取、移除单个元素<br>
2）检查一个元素是否存在于集合中<br>
3）计算交集、并集、差集<br>
4）从集合里面随机获取元素

客户端命令如下：

```
> sadd set-key item
(integer) 1
> sadd set-key item2
(integer) 1
> sadd set-key item3
(integer) 1
> sadd set-key item
(integer) 0

> smembers set-key
1) "item"
2) "item2"
3) "item3"

> sismember set-key item4
(integer) 0
> sismember set-key item
(integer) 1

> srem set-key item2
(integer) 1
> srem set-key item2
(integer) 0

> smembers set-key
1) "item"
2) "item3"
```

### zset

即sorted set，String类型的有序集合。通过哈希表实现，所以添加，删除，查找的复杂度都是O(1)。不允许重复元素，每个元素都会关联一个double类型的分数（score），以此作为元素从小到大的排序。分数值允许重复。

支持如下操作：

1）添加、获取、删除元素<br>
2）根据分值范围或者成员来获取元素<br>
3）计算一个键的排名

客户端命令如下：

```
> zadd zset-key 728 member1
(integer) 1
> zadd zset-key 982 member0
(integer) 1
> zadd zset-key 982 member0
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"

> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"

> zrem zset-key member1
(integer) 1
> zrem zset-key member1
(integer) 0

> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```

## 数据结构

### 字典

### 跳跃表


## 参考链接

* [Redis](http://www.cyc2018.xyz/%E6%95%B0%E6%8D%AE%E5%BA%93/Redis.html)
* [Redis 教程](https://www.runoob.com/redis/redis-tutorial.html)