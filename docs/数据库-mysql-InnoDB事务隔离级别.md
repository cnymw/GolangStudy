# InnoDB 事务隔离级别

事务隔离是数据库处理的基础之一。隔离是 `ACID` 中的 `I`。隔离级别是一种设置，当多个事务同时进行更改和执行查询时，它可以调整性能与可靠性，一致性和可复现性之间的平衡。

InnoDB 提供了 `SQL:1992` 标准描述的四个事务隔离级别：

- 读未提交（READ UNCOMMITTED）

- 不可重复读（READ COMMITTED）

- 可重复读（REPEATABLE READ）

- 序列化（SERIALIZABLE）

InnoDB 默认隔离级别为可重复读。

用户可以使用 `SET TRANSACTION` 语句更改单个会话或所有后续连接的隔离级别。要为所有连接设置服务器的默认隔离级别，请在命令行或选项文件中使用`--transaction-isolation`选项。

InnoDB 使用不同的锁策略来支持不同的事务隔离级别。

对于关键数据上的操作，需要符合 ACID 的规范，那么可以使用默认隔离级别——`REPEATABLE READ`来保持数据的高度一致性。

或者，如果精确的一致性和可重复的结果不像最大限度地减少锁定开销那么重要，则可以通过`READ COMMITTED`或甚至`READ UNCOMMITTED`来放松一致性原则。

`SERIALIZABLE`执行比`REPEATABLE READ`更严格的规则，主要用于特殊情况，如 XA 事务，以及解决并发和死锁问题。

## 读未提交 READ UNCOMMITTED

`SELECT`语句以未锁定方式执行，但可能读取到该记录的早期版本。因此，使用此隔离级别会导致读取是不一致的，这也被称为脏读。否则，此隔离级别的工作方式类似于`READ COMMITTED`。

## 不可重复读 READ COMMITTED

即使在同一事务中，每一个一致性读都会写入并读取自己的最新版本。

对于锁定读语句`SELECT ... FOR SHARE`和`SELECT ... FOR UPDATE`，UPDATE 语句，DELETE 语句，InnoDB 只锁定索引记录，而不锁定它们之间的间隙，因此允许在锁定记录旁边自由插入新纪录。间隙锁定仅用于外键约束检查和重复键检查。

由于禁用了间隙锁`Gap Locks`，可能会出现幻读问题，因为其他会话可能会在间隙中插入新行。

`READ COMMITTED`仅支持基于行的二进制日志记录。如果使用`binlog_format=MIXED`参数，服务器将自动使用基于行的日志记录。

使用`READ COMMITTED`还有其他效果：

- 对于 UPDATE 或 DELETE 语句，InnoDB 仅为更新或删除的行持有锁。Mysql 评估 WHERE 条件后，将释放不匹配行的记录锁`Record Locks`。这大大降低了死锁的概率，但死锁仍然可能发生。

- 对于 UPDATE 语句，如果一条记录已经被锁定，InnoDB 将执行半一致性读`semi-consistent read`，将最新提交的版本返回给 Mysql，以便 Mysql 可以确定该行是否匹配更新的 WHERE 条件。如果记录匹配（已经被更新），Mysql 将再次读取该记录，这次 InnoDB 将锁定该行或等待锁定该行。

