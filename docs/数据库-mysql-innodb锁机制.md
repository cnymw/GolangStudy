# InnoDB 锁

## 共享锁和排他锁(Shared and Exclusive Locks)

InnoDB 实现了两种行锁，共享锁和排他锁：

- 共享锁允许事务拥有读一个记录数据的锁（holds the lock to read a raw）
  
- 排他锁允许事务拥有更新/删除一个记录数据的锁(holds the lock to update or delete a row)

如果事务一在记录上面获得了共享锁，那么事务二的锁会如下操作：

- 事务二会立即获得共享锁，最终事务一和事务二同时在记录上面获得共享锁
  
- 事务二不会立即获得排他锁，需要等事务一释放共享锁之后，事务二才能获得排他锁

如果事务一在记录上面获得了排他锁，那么事务二不会立即获得任意一种锁，除非事务一释放排他锁。

## 意向锁（Intention Locks）

InnoDB 支持多粒度锁，允许行锁和表锁共存。

为了支持多粒度锁，InnoDB 使用了意向锁，意向锁是表级锁，指示事务需要对表中的记录使用哪种类型的锁（共享或排他），有两种类型的意向锁：

- 意向共享锁表示事务打算在表上的记录设置共享锁
  
- 意向排他锁表示事务打算在表上的记录设置排他锁

例如，*SELECT ... LOCK IN SHARE MODE* 会设置意向共享锁， *SELECT ... FOR UPDATE* 会设置意向排他锁

意向锁的原则如下所示：

- 在事务获取记录共享锁之前，事务必须首先获得一个表的意向共享锁或者意向排他锁
  
- 在事务获取记录排他锁之前，事务必须首先获得一个表的意向排他锁

```sql
	X	        IX	        S	        IS
X	Conflict	Conflict	Conflict	Conflict
IX	Conflict	Compatible	Conflict	Compatible
S	Conflict	Conflict	Compatible	Compatible
IS	Conflict	Compatible	Compatible	Compatible
```

如果和现有的锁可以兼容，那么会给事务授予锁。如果和现有的锁冲突了，则不会授予锁。

如果一个请求锁和现有锁冲突了，并且不能被授予，因为这将导致一个死锁，会发生错误。

除了全表请求（例如，*LOCK TABLES ... WRITE*）意向锁不会阻塞任何请求，意向锁的主要意图是通知其他请求，有人正在锁一条记录或者准备去锁一条记录。

意向锁的事务显示如下所示：

```go
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

## 记录锁(Record Locks)

记录锁是用于锁一个索引记录。

例如，*SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;* 语句会阻止其他事务对 *t.c1=10* 记录做增删改操作。

即使表没有索引，记录锁也会锁索引记录，针对于这种情况，InnoDB 创建一个隐藏的聚集索引，并将这个索引用于记录锁。

记录锁的事务显示如下：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` 
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
2: len 7; hex b60000019d0110; asc        ;;
```

## 间隙锁(Gap Locks)

间隙锁是索引记录中间范围的锁，也就是第一个和最后一个索引记录之间上的锁。例如，*SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;* 语句阻止其他事务插入数据 15 到 *t.c1* 列。

也就是说，*t.c1* 列的 10-20 之间的所有记录全部被间隙锁锁定，其他事务无法操作增删改操作。

间隙锁可以跨域单个，多个，甚至空的索引值。

间隙锁是为了权衡性能和并发行而产生的，仅用于某些事务隔离级别。

使用唯一索引搜索单个记录不会使用间隙锁（不包括搜索条件包含多列唯一索引的情况，在这种情况下，会使用到间隙索引）。例如，如果 *id* 列具有唯一索引，下面的语句只会在 *id=100* 记录上使用一个索引记录锁（index-record lock），
而不会在 *id<100* 这个范围内加间隙锁。

```go
SELECT * FROM child WHERE id = 100;
```

然而，如果 *id* 没有唯一索引，那么上面语句会将 *id<100* 范围加上间隙锁。

这里需要注意的是，不同的事务可以在间隙上拥有冲突的锁，例如，事务 A 可以在间隙上持有共享间隙锁(gap S-lock)，同时，事务 B 可以在同一个间隙上持有一个排他间隙锁（gap X-lock）。间隙锁允许冲突的原因是，如果从索引中清除记录，
那么被不同事务所持有的间隙锁必须合并。

在 InnoDB 中，间隙锁的作用是用于"单纯禁止"的目的，他们存在的意义在于阻止其他事务插入数据到间隙中。间隙锁可以共存。一个事务的间隙锁不能阻止另一个事务在同一个间隙上面也持有间隙锁。共享间隙锁和排他间隙锁没有区别。他们相互不会
冲突，他们起到同样的作用。

可以显式的禁用间隙锁。如果将事务隔离级别更改为 *READ COMMITTED* 或启用 *innodb_locks_unsafe_for_binlog* 系统变量，则会禁用间隙锁。在这种情况下，在搜索和索引扫描中会禁用间隙锁，间隙锁仅会用于外键约束检查和重复键检查。

使用 *READ COMMITTED* 或启用 *innodb_locks_unsafe_for_binlog* 系统变量还有其他影响。在 MySQL 计算 WHERE 条件只会，会释放非匹配行的记录锁。对于 UPDATE 语句，InnoDB 执行半一致读（semi-consistent read），
以便能够将最新提交的最新版本返回给 MySQL，这样可以使 MySQL 确定行是否于最新的 WHERE 条件匹配。

总结一下：MySQL 通过间隙锁来锁住目标记录可能出现的位置，如果检索条件有索引，可以通过索引锁住目标位置，如果索引是 unique 则不用锁间隙，因为不会出现间隙，如果没有索引会锁住全表。

## Next-Key 锁（Next-Key Locks）

Next-Key 锁是索引记录上的 Record Lock 和在索引记录之前的Gap Lock 的组合。

InnoDb 执行行锁的方式是这样的，当它搜索或扫描表索引时，它会对遇到的索引记录设置共享或独占锁。因此，行级锁实际上是索引记录锁。索引记录上的 Next-Key 锁也会影响该索引记录之前的 gap。
也就是说，Next-Key 锁是一个索引记录锁加上索引记录前面的间隙上的 Gap Lock。如果一个会话对索引中的记录 R 具有共享或独占锁，那么另一个会话无法在紧靠 R 的索引顺序的间隙中插入新的索引记录。

假设一个索引包含值 10，11，13 和 20，那么这个索引的 Next-Key 锁可能包含以下间隔，其中圆括号表示排除间隔端点，方括号表示包含端点：

```bash
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```

对于最后一个间隔，Next-Key 锁锁定索引中最大值上方的间隙，以及值高于索引中任何实际值的"上界"伪记录。上界值不是真正的索引记录，因此，Next-Key 锁只锁定最大索引值后面的间隙。

默认情况下，InnoDB 在 REPEATABLE READ 事务隔离级别运行。在这种情况下，InnoDB 使用 Next-Key 锁进行搜索和索引扫描，这样可以防止幻读。

在输入 *SHOW ENGINE INNODB STATUS* 后，Next-Key 锁的事务数据如下所示：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10080 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

## 插入意向锁（Insert Intention Locks）

插入意向锁是 INSERT 操作在插入行之前设置的一种间隙锁。这个锁的意图在于，在插入到同一索引间隙中的多个事务如果不是在间隙内的同一位置插入，那么不需要等待彼此。假设有值为 4 和 7 的索引记录，分别插入值 5 和 6 的独立事务。
在获得插入行的排他锁之前，每个事务都使用插入意向锁锁定 4 和 7 之间的间隙，但不会因为行不冲突而阻塞彼此。

下面的示例演示了一个事务，它在获得插入记录的排他锁之前使用了插入意向锁。这个例子涉及两个客户机，A 和 B。

A 创建一个包含两个索引记录（90 和 102）的表，然后启动一个事务，该事务对 ID 大于 100 的索引记录进行独占锁定。独占锁包含记录 102 之前的间隙锁：

```sql
mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
mysql> INSERT INTO child (id) values (90),(102);

mysql> START TRANSACTION;
mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
+-----+
| id  |
+-----+
| 102 |
+-----+
```

B 开始一个事务，将一个记录插入到间隙中。事务在等待获得排他锁时接受插入意向锁。

```sql
mysql> START TRANSACTION;
mysql> INSERT INTO child (id) VALUES (101);
```

输入*SHOW ENGINE INNODB STATUS*后，插入意向锁的事务数据如下所示：

```sql
RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
trx id 8731 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000066; asc    f;;
 1: len 6; hex 000000002215; asc     " ;;
 2: len 7; hex 9000000172011c; asc     r  ;;...
```

## 自增锁(AUTO-INC lock)

自增锁是一种特殊的表级锁，它由事务插入具有*AUTO_INCREMENT*列的表中获得。在最简单的情况下，如果一个事务正在向表中插入值，则任何其他事务都必须等待，以便第一个事务插入的行接收连续的主键值。

*innodb_autoinc_lock_mode*配置控制自增锁定的算法，它允许你选择在可预测的自动递增序列和插入操作的最大并发性之间进行权衡。

## 空间索引谓词锁（Predicate Locks for Spatial Indexes）

InnoDB 支持对包含空间列的列进行空间索引。

要处理涉及空间索引的操作的锁定，Next-Key 锁不能很好的支持*REPEATABLE READ*或*SERIALIZABLE*事务隔离级别，多维数据中没有绝对的排序概念，因此不清楚哪个是下一个键。

为了支持具有空间索引的表的隔离级别，InnoDB 使用谓词锁。空间索引包含最小边界矩形（MBR）值，因此 InnoDB 通过在用于查询的 MBR 值上设置谓词锁来保持制索引的一致性。其他事务无法插入或修改与查询条件匹配的行。

# 参考资料
- [MySQL 8.0 Reference Manual:InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)