# mysql InnoDB Locking Reads 锁定读

如果在同一事务中插入或更新相关数据，则常规 SELECT 语句无法提供足够的保护。其他事务可以更新或删除你刚才查询的数据行。InnoDB 支持两种提供额外安全性的锁定读：

- SELECT ... FOR SHARE：在读取的任意数据行上设置共享锁。其他会话可以读取数据行，但在事务提交之前无法修改他们。如果这些数据行中的任何一行被另一个尚未提交的事务更改，则查询将等待该事务结束，然后使用最新的值。

> `SELECT ... FOR SHARE` 是 `SELECT ... LOCK IN SHARE MODE` 的替代方案，但 `LOCK IN SHARE MODE` 仍支持向后兼容，这些语句都是等价的。

- SELECT ... FOR UPDATE：就好像对这些行执行了 `UPDATE` 操作一样，该 sql 会锁定行和任何关联的索引项。其他事务被阻止更新这些行，无法执行 `SELECT ... FOR SHARE`，或从某个事务隔离级别读取数据。隔离级别一致性读会忽略在读视图中存在的记录上设置的任何锁。

提交或回滚事务的时候，将释放`FOR SHARE`和`FOR UPDATE`查询设置的所有锁。

> 锁定读仅在禁用自动提交时才可以生效（使用 START TRANSACTION 或将 autocommit 设置为 0）

外部语句中的锁定读不会锁定嵌套子查询中的行，除非在子查询中也指定了锁定读子句，例如，下面的语句不会锁定 t2 中的行：

```mysql
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2) FOR UPDATE;
```

要锁定 t2 中的行，需要向子查询添加一个锁定读子句：

```mysql
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2 FOR UPDATE) FOR UPDATE;
```

# 锁定读案例

假设要将一条数据插入到表 CHILD，并确保 child 数据在表 PARENT 中有一个父 parent 数据。应用程序代码可以在整个操作中确保引用完整性。

首先，使用一致性读查询表 PARENT 并验证数据 parent 是否存在。确认你能安全地将数据 child 插入表 CHILD 吗？答案是不可以的，因为其他会话可能会在 SELECT 和 INSERT 之间删除数据 parent，而你不会意识到这一点。

如果要避免这个潜在的问题，可以执行`SELECT FOR SHARE`:

```mysql
SELECT * FROM parent WHERE NAME = 'Jones' FOR SHARE;
```

`SELECT FOR SHARE`查询返回 parent `Jones`后，你可以安全地将数据 child 添加到表 CHILD 并提交事务。任何试图对表 PARENT 中该行数据获取独占锁的事务都会等待，直到完成插入到表 CHILD 完成，也就是说，直到所有表中的数据处于一致性状态为止。

另一个例子为，表 CHILD_CODES 有一个整数计数器字段，给每一个添加到表 CHILD 的对象分配唯一标识符。这个整数计数器的当前值不能使用一致性读或共享模式读取，因为数据库的两个用户可以看到计数器相同的值，如果两个事务试图向表 CHILD 添加具有相同标识符的行，则会发生重复主键（duplicate-key）错误。

在这里，`FOR SHARE`不是一个好的解决方案，因为如果两个用户同时读取计数器，那么当它尝试更新计数器时，其中至少有一个用户会陷入死锁。

要实现读取和递增计数器，首先使用`FOR UPDATE`对计数器执行锁定读，然后递增计数器，例如：

```mysql
SELECT counter_field FROM child_codes FOR UPDATE;
UPDATE child_codes SET counter_field = counter_field + 1;
```

`SELECT ... FOR UPDATE`读取最新的可用数据，在它读取的每一行上设置独占锁。因此，它查询时设置的锁和更新时的锁是相同的。

前面描述的仅仅是`SELECT ... FOR UPDATE`如何工作的案例。在 MYSQL 中，生成唯一标识符的任务实际上只需对表进行一次访问即可完成：

```mysql
UPDATE child_codes SET counter_field = LAST_INSERT_ID(counter_field + 1);
SELECT LAST_INSERT_ID();
```

SELECT 语句仅检索标识符信息（仅针对于当前连接）。它不访问任何表。

# 参考资料

- [Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)