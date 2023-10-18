<!--
date: 2023-10-18T22:34:12+08:00
lastmod: 2023-10-18T22:34:12+08:00
-->
## 索引优化

### 单列索引

查询时索引列不能参与表达式运算（包括类型转换），也不能是函数的参数，否则索引失效。

```sql
-- 索引失效
SELECT name FROM student WHERE age + 1 = 15;
```

### 多列索引

查询时多个查询条件可以建立联合索引，查询效率比起单列索引要更好。且使用联合索引时，必须使用到前面的索引列来查询，否则索引失效。

```sql
-- 创建了(name-data)的联合索引

-- 能使用到联合索引
select * from test where name = 'fjh' and data = 'text';
select * from test where name = 'fjh';
select * from test where data = 'text' and name = 'fjh';

-- 使用不到联合索引
select * from test where data = 'text';
```

### 索引列顺序

创建联合索引时将区分度更高的索引列排在前面，可以提高查询效率。

比如学生表里的性别和姓名两个字段如果要创建联合索引，那么姓名字段明显区分度更高，而性别字段只有两种值，因此创建联合索引时姓名字段应该在性别前面。

### 前缀索引

对于 BLOB、TEXT 和 VARCHAR 类型的列，必须使用前缀索引，只索引开始的部分字符。

前缀长度的选取需要根据索引选择性来确定。

### 覆盖索引

索引包含所有需要查询的字段，这样可以通过减少回表次数来提高查询效率。

## 查询性能优化

### 使用Explain分析查询性能

Explain 用来分析 SELECT 查询语句，开发人员可以通过分析 Explain 结果来优化查询语句。

比较重要的字段有：

```html
select_type : 查询类型，有简单查询、联合查询、子查询等
key : 使用的索引
rows : 扫描的行数
```

### 降低数据访问量

* 只返回必要的数据列：避免使用`select *`
* 只返回必要的数据行：使用`limit`限制返回的行数
* 缓存重复查询的数据：使用缓存避免或减少数据库的查询行为，可以明显提升查询性能。
* 使用索引来覆盖查询，可以避免全表扫描。

### 重构查询方式

**1.切分大查询**

一个大查询如果一次性执行的话，可能会一次性锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但重要的查询。

比如删除三个月内的数据，可以切分成批量删除数据，每次删除一万条数据：

```sql
-- 切分前：
DELETE FROM messages WHERE create < DATE_SUB(NOW(), INTERVAL 3 MONTH);

-- 切分后：
rows_affected = 0
do {
    rows_affected = do_query(
    "DELETE FROM messages WHERE create  < DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000")
} while rows_affected > 0
```

**2.分解大连接查询**

将一个大连接查询分解成对每一个表进行一次单表查询，然后在应用程序中进行关联，这样做的好处有：

* 让缓存更高效。对于连接查询，如果其中一个表发生变化，那么整个查询缓存就无法使用。而分解后的多个查询，即使其中一个表发生变化，对其它表的查询缓存依然可以使用。
* 分解成多个单表查询，这些单表查询的缓存结果更可能被其它查询使用到，从而减少冗余记录的查询。
* 减少锁竞争。
* 在应用层进行连接，可以更容易对数据库进行拆分，从而更容易做到高性能和可伸缩。
* 查询本身效率也可能会有所提升。例如下面的例子中，使用 IN() 代替连接查询，可以让 MySQL 按照 ID 顺序进行查询，这可能比随机的连接要更高效。

```sql
-- 分解前：
SELECT * FROM tag
JOIN tag_post ON tag_post.tag_id=tag.id
JOIN post ON tag_post.post_id=post.id
WHERE tag.tag='mysql';

-- 分解后：
SELECT * FROM tag WHERE tag='mysql';
SELECT * FROM tag_post WHERE tag_id=1234;
SELECT * FROM post WHERE post.id IN (123,456,567,9098,8904);
```

### 其他优化

* 慎用`union`，其会自动去重导致额外的性能损耗，改用`union all`。
* 小表驱动大表，即用小表查询出来的数据去查询大表中的数据，这样可以有效降低数据表扫描的行数。
* like模糊匹配避免使用左侧模糊匹配。

## 参考链接

* [MySQL](http://www.cyc2018.xyz/%E6%95%B0%E6%8D%AE%E5%BA%93/MySQL.html)
* [SQL优化的15个小技巧，纯干货分享！](https://blog.csdn.net/HJW_233/article/details/131636552)