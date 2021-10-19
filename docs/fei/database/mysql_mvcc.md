# MySql MVCC机制

## MVCC(多版本并发控制)

`MVCC`只在`Read Committed`和`Repeatable Read`两个隔离级别下工作，其他两个隔离级别和`MVCC`不兼容，`Read Uncommitted`总是读取最新的记录行，而不是符合当前事务版本的记录行，`Serializable`则会对所有读取的记录行都加锁

## 隐藏字段

`InnoDB`存储引擎在每行数据的后面添加了三个隐藏字段

* `DB_TRX_ID`(6字节)

  表示最近一次对本记录行作修改(`insert`|`update`|`delete`)的事务ID

* `DB_ROLL_PTR`(7字节)

  回滚指针，指向当前记录行的`undo log`的信息

* `DB_ROW_ID`(6字节)

  随着新行插入而单调递增的行ID

## Read View(snapshot)

`Read View`主要是用来做可见性判断的，存储了当前系统中其他活跃事务的信息

* `low_limit_id`

  目前出现过的最大的事务`ID + 1`，即下一个将被分配的事务ID

* `up_limit_id`

  活跃事务列表`trx_ids`中最小的事务ID，如果`trx_ids`为空，则`up_limit_id`为`low_limit_id`

* `trx_ids`

  `Read View`创建时其他未提交的活跃事务ID列表，`Read View`中`trx_ids`的活跃事务，不包括当前事务自己和已提交的事务

* `creator_trx_id`

  当前创建事务的ID，是一个递增的编号

## Undo log

`Undo log`中存储的是老版本数据，当一个事务需要读取记录行时，如果当前记录行不可见，可以顺着`Undo log`链找到满足其可见性条件的记录行版本

* `insert undo log`

  事务对`insert`新记录时产生的`undo log`，只在事务回滚时需要，并且在事务提交后就可以立即丢弃

* `update undo log`

  事务对记录进行`delete`和`update`操作时产生的`undo log`，不仅在事务回滚时需要，快照读也需要，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被`purge`线程删除

## 可见性比较算法

在`InnoDB`中，创建一个新事务后，执行第一个`select`语句的时候，`InnoDB`会创建一个快照(`Read View`)，快照中会保存系统当前不应该被本事务看到的其他活跃事务的事务ID列表(即`trx_ids`)，当用户在这个事务中要读取某个记录行的时候，`InnoDB`会将该记录行的`DB_TRX_ID`与该`Read View`中的一些变量进行比较，判断是否满足可见性条件

设读取的记录行的`DB_TRX_ID`为`trx_id`

* `trx_id`<`up_limit_id`，记录行可见
* `trx_id`>=`low_limit_id`，记录行不可见
* `up_limit_id`<=`trx_id`<`low_limit_id`
  * `trx_ids`包含`trx_id`，记录行不可见
  * `trx_ids`不包含`trx_id`，记录行可见

当前读取的记录行不可见时，在该记录行的`DB_ROLL_PTR`指针所指向的`undo log`回滚中取出最新的旧事务ID(`DB_TRX_ID`)，将它赋值给`trx_id`，然后重新进行可见性判断

## 当前读和快照读

* 快照读(`snapshot read`)

  普通的`select`语句(不包括`select ... lock in share mode`，`select ... for update`)

* 当前读(`current read`)

  `select ... lock in share mode`，`select ... for update`，`insert`，`update`，`delete`等语句

## 实验(mysql 5.7)

* RC

  A修改数据未提交事务，B快照读保证提交读，B当前读会被阻塞

  A修改数据并提交事务，B快照读和当前读都出现不可重复读、幻读

* RR

  A修改数据未提交事务，B快照读保证提交读，B当前读会被阻塞

  A修改数据并提交事务，B快照读保证可重复读、防止幻读，B当前读会出现不可重复读、幻读

### RR和RC创建Read View时的区别

在`InnoDB`中的`Repeatable Read`级别，只有事务在`begin`之后，执行第一条`select`(读操作)时，才会创建第一个快照(`read view`)，将当前系统中活跃的其他事务记录起来，并且事务之后都使用这个快照，不会重新创建，直到事务结束

在`InnoDB`中的`Read Committed`级别，事务在`begin`之后，执行每条`select`(读操作)语句时，快照都会被重置，即会重新创建一个快照(`read view`)
