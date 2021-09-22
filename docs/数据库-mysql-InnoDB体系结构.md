# mysql InnoDB 体系结构

下图展示了组成 InnoDB 存储引擎体系中的内存和磁盘结构：

![数据库-mysql-InnoDB体系结构.png](https://cnymw.github.io/GolangStudy/docs/img/数据库-mysql-InnoDB体系结构.png)

## InnoDB 内存结构 In-Memory Structures

本节介绍 InnoDB 内存结构和相关主题。

### 缓冲池 Buffer Pool

缓冲池是主内存中的一个区域，InnoDB 在访问表和索引数据时将数据缓存到缓冲池。缓冲池允许直接从内存访问经常使用的数据，从而加快处理速度。在专用服务器上，高达 80% 的物理内存通常分配给缓冲池。

为了提高大容量读取的效率，缓冲池被划分为尽可能容纳多行的页面。为了提高缓存管理的效率，缓冲池被实现为页面的链表，很少使用的数据会使用 LRU 变体算法从缓冲中失效。

了解如何利用缓冲池将频繁访问的数据保存在内存中是 Mysql 调优的一个重要方向。

#### 缓冲池 LRU 算法

缓冲池使用 LRU 变体算法来管理链表。当需要空间将新的页面添加到缓冲池时，将放逐（evict）最近使用最少的页，并将新的页面添加到链表的中间。这个中点插入策略将链表视为两个子链表：

- head，是最近访问的新（年轻）页面的子链表
- tail，是最近访问较少的旧页面的子链表

![数据库-mysql-InnoDB缓冲池链表.png](https://cnymw.github.io/GolangStudy/docs/img/数据库-mysql-InnoDB缓冲池链表.png)

该算法将经常使用的页面保留在新的子链表中，旧的子链表包含使用频率较低的页面，这些页面是候选被驱逐的页面。

默认情况下，算法的操作如下：

- 缓冲池的 3/8 用于旧的子链表
- 链表的中点是新子链表的尾部与旧子链表的头部相交的边界
- 当 InnoDB 将一个页面读入缓冲池时，它首先将其插入到中点（旧子链表的头部）。一个页面被读取是因为它是用户执行的操作（如 SQL 查询）所必须的，或者是 InnoDB 自动执行的预读操作的一部分。
- 访问旧子链表中的页面会使其"年轻"，并将其移动到新子链表的开头。如果由于用户执行操作的需要而读取该 面，则会立即进行第一次访问，并使该页面处于年轻状态。如果页面是由于预读操作而读取的，则第一次访问不会立即发生，并且可能在页面被驱逐之前根本不会发生。
- 随着数据库的运行，缓冲池中未被访问的页面会向链表的尾部移动，从而"老化"。新旧子链表中的页面都会随着其他页面的更新而老化。在中点插入页面时，旧子链表中的页面也会老化。最终，一个一直未使用的页面到达旧子链表的尾部并被逐出。

默认情况下，查询读取的页面会立即移动到新的子链表中，这意味着他们在缓冲池中停留的时间更长。例如，为 mysqldump 操作或不带 `WHERE` 的 `SELECT` 语句执行的表扫描可以将大量数据带入缓冲池，并驱逐出等量的旧数据，即使新数据不再使用。

类似地，由预读（read-ahead）后台线程加载且仅访问一次的页面将移动到新链表的开头。这些情况可能会将经常使用的页面推到旧的子链表中，并将经常使用的页面在旧子链表中逐出。

InnoDB 标准监视器输出在缓冲池和内存部分包含几个字段，这些字段与缓冲池 LRU 算法的操作有关。

#### 使用 InnoDB 标准监视器监视缓冲池

InnoDB 标准监视器输出可以使用`SHOW ENGINE INNODB STATUS`访问，它提供了有关缓冲池操作的指标。缓冲池指标输出如下：

```mysql
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 2198863872
Dictionary memory allocated 776332
Buffer pool size   131072
Free buffers       124908
Database pages     5720
Old database pages 2071
Modified db pages  910
Pending reads 0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 4, not young 0
0.10 youngs/s, 0.00 non-youngs/s
Pages read 197, created 5523, written 5060
0.00 reads/s, 190.89 creates/s, 244.94 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not
0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read
ahead 0.00/s
LRU len: 5720, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
```

### 写缓冲区 Change Buffer

写缓冲区是一种特殊的数据结构，当二级索引页不在缓冲池时，写缓冲区会缓存这些页的更改。缓存的更改可能由插入，更新或删除操作（DML）导致，稍后当页面通过其他读取操作加载到缓冲池时，会合并这些更改。

![数据库-mysql-InnoDB写缓冲.png](https://cnymw.github.io/GolangStudy/docs/img/数据库-mysql-InnoDB写缓冲.png)

与聚集索引不同，二级索引通常是非唯一的，二级索引的插入顺序相对随机。类似的，删除和更新可能会影响索引树中不相邻的二级索引页。当其他操作将受影响的页面读入缓存时，在未来合并缓存的更改时，可以避免从磁盘将二级索引页面读入缓存池所需的大量随机访问 IO。

在系统大部分空闲或缓慢关机时运行的清洗操作会定期将更新的索引页写入磁盘。与立即将每个值写入磁盘相比，清洗操作可以更有效地将一系列索引值写入磁盘块。

当有许多受影响的行和许多索引需要更新时，合并写缓冲区可能需要几个小时。在此期间，磁盘 IO 会增加，这可能会导致查询磁盘数据的速度显著降低。提交事务后，甚至在服务器关闭并重新启动后，合并写缓冲区也可能继续执行。

在内存中，写缓冲区占用缓冲池的一部分。在磁盘上，写缓冲区是系统表空间的一部分，当数据库服务器关闭时，索引的更改将在这里进行缓存。

缓存在写缓冲区中的数据类型由 `innodb_change_buffering` 变量控制，你还可以配置最大写缓冲区的大小。

如果二级索引包含降序索引或主键中包含降序索引列，那么这类索引不支持写缓冲区。

#### 监视写缓冲区

以下选项可用于监视写缓冲区：

- InnoDB 标准监视器输出包括写缓冲状态信息：

```mysql
mysql> SHOW ENGINE INNODB STATUS\G

-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
    -------------------------------------
    Ibuf: size 1, free list len 0, seg size 2, 0 merges
    merged operations:
insert 0, delete mark 0, delete 0
    discarded operations:
insert 0, delete mark 0, delete 0
    Hash table size 4425293, used cells 32, node heap has 1 buffer(s)
    13577.57 hash searches/s, 202.47 non-hash searches/s

```

- `INFORMATION_SCHEMA.INNODB_METRICS` 表提供了 InnoDB 标准监视器输出中大多数指标以及其他指标：

```mysql
mysql> SELECT NAME, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME LIKE '%ibuf%'\G
```

- `INFORMATION_SCHEMA.INNODB_BUFFER_PAGE` 表提供了缓冲池中每个页面的元数据，包括写缓冲区索引和位图（bitmap）页面。写缓冲区页面由页面类型标识。`PAGE_TYPE.IBUF_INDEX` 是写缓冲区索引页面的页面类型。`IBUF_BITMAP` 是写缓冲区位图页面的页面类型。

```mysql
mysql> SELECT (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
       WHERE PAGE_TYPE LIKE 'IBUF%') AS change_buffer_pages,
       (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE) AS total_pages,
       (SELECT ((change_buffer_pages/total_pages)*100))
       AS change_buffer_page_percentage;
+---------------------+-------------+-------------------------------+
| change_buffer_pages | total_pages | change_buffer_page_percentage |
+---------------------+-------------+-------------------------------+
|                  25 |        8192 |                        0.3052 |
+---------------------+-------------+-------------------------------+
```

- 性能模式为高级性能监控提供了写缓冲区互斥等待检测，要查看写缓冲区检测，执行以下查询：

```mysql
mysql> SELECT * FROM performance_schema.setup_instruments
       WHERE NAME LIKE '%wait/synch/mutex/innodb/ibuf%';
+-------------------------------------------------------+---------+-------+
| NAME                                                  | ENABLED | TIMED |
+-------------------------------------------------------+---------+-------+
| wait/synch/mutex/innodb/ibuf_bitmap_mutex             | YES     | YES   |
| wait/synch/mutex/innodb/ibuf_mutex                    | YES     | YES   |
| wait/synch/mutex/innodb/ibuf_pessimistic_insert_mutex | YES     | YES   |
+-------------------------------------------------------+---------+-------+
```

### 自适应哈希索引 Adaptive Hash Index

自适应哈希索引使 InnoDB 能够在具有适当的工作负载环境和足够的缓冲池内存的系统上执行更像内存（in-memory）数据库，而不会牺牲事务功能或可靠性。自适应哈希索引由 `innodb_adaptive_hash_index` 变量启用，或在服务器启动时通过 `--skip-innodb-adaptive-hash-index` 命令关闭。

根据搜索的观察模式，哈希索引使用索引键的前缀构建。前缀可以是任意长度，并且可能只有 B 树（B-tree）中的某些值出现在哈希索引中。哈希索引是根据需要为经常访问的索引页面构建的。

如果一个表几乎完全位于主内存中，则哈希索引通过允许直接查找任何元素来加速查询，从而将索引值转换为某种指针。InnoDB 有一种监视索引搜索的机制，如果 InnoDB 注意到查询可以从构建哈希索引中获益，它会自动这样做。

对于某些工作负载，哈希索引查找的速度会大大超过了监视索引查找和维护哈希索引结构的额外工作。在繁重的工作负载下（例如多个并发连接），对自适应哈希索引的访问有时会成为冲突的来源。使用 LIKE 运算符和 % 通配符的查询也往往不会受益。对于不受益于自适应哈希索引的工作负载，关闭它可以减少不必要的性能开销。因为很难预测自适应哈希索引是否适用于特定的系统和工作负载，所以请考虑启用和禁用的运行基准。

对自适应哈希索引功能进行分区，每个索引都绑定到一个特定的分区，每个分区都由一个单独的锁保护。分区由 `innodb_adaptive_hash_index_parts` 变量控制，默认设置为 8，最大为 512。

可以使用 `SHOW ENGINE INNODB STATUS` 监视自适应哈希索引的使用和争用。

```mysql
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 719810
OS WAIT ARRAY INFO: signal count 689746
RW-shared spins 0, rounds 1080088, OS waits 528310
RW-excl spins 0, rounds 277881, OS waits 3440
RW-sx spins 4026, rounds 80960, OS waits 1140
Spin rounds per wait: 1080088.00 RW-shared, 277881.00 RW-excl, 20.11 RW-sx
```

### 日志缓冲区 Log Buffer

日志缓冲区是存储要写入磁盘上日志文件数据的内存区域。日志缓冲区大小由 `innodb_log_buffer_size` 变量定义。默认大小为 16MB。日志缓冲区的内容定期刷新到磁盘。大型日志缓冲区使大型事务能够运行，而无需在事务提交之前将重做日志（redo log）数据写入磁盘。因此，如果有更新，插入或删除多行的事务，增加日志缓冲区的大小可以节省磁盘 IO。

`innodb_flush_log_at_trx_commit` 变量控制如何将日志缓冲区的内容写入并刷新到磁盘。

`innodb_flush_log_at_timeout` 变量控制日志刷新频率。

## InnoDB 磁盘结构 On-Disk Structures

本节介绍 InnoDB 磁盘结构和相关主题。

### 数据库表结构

#### 创建 InnoDB 数据库表

使用`CREATE TABLE`语句创建 InnoDB 表，例如：

```mysql
CREATE TABLE t1 (a INT, b CHAR (20), PRIMARY KEY (a)) ENGINE=InnoDB;
```

当默认存储引擎指定为 InnoDB 时，`ENGINE=InnoDB`不是必须的。但是，如果 InnoDB 不是默认引擎或未知的其他 Mysql 服务器实例上执行`CREATE TABLE`语句，`ENGINE`语句非常有用。可以通过以下语句来确定 Mysql 服务器实例上的默认存储引擎：

```mysql
mysql> SELECT @@default_storage_engine;
+--------------------------+
| @@default_storage_engine |
+--------------------------+
| InnoDB                   |
+--------------------------+
```

默认情况下，InnoDB 表是在每个表的文件表空间中创建的。要在 InnoDB 系统表空间中创建 InnoDB 表，请在创建表之前禁用 innodb_file_per_table 变量。要在常规表空间中创建表，请使用`CREATE TABLE ... TABLESPACE`语法。

##### 行格式 Row Formats

InnoDB 表的行格式决定该行在磁盘上的物理存储方式。InnoDB 支持四种行格式，每种格式具有不同的存储特性。支持的行格式包括：`REDUNDANT`，`COMPACT`，`DYNAMIC`，`COMPRESSED`。默认为 `DYNAMIC`。

`innodb_default_row_format`定义默认的行格式，也可以使用`CREATE TABLE`或`ALTER TABLE`语句中的`ROW_FORMAT`显式定义表的行格式。

##### 主键 Primary Keys

建议为创建的每个表定义一个主键。选择主键列时，请选择具有如下特征的列：

- 由最重要的查询引用的列
- 从不留空的列
- 从不具有重复值的列
- 插入后基本不更改值的列

例如，在包含人员信息的表中，你不会在（姓，名）上创建主键，因为多个人员可以具有相同的姓名，姓名列可能留空，有时人员会更改姓名。因此可以创建一个整数 ID 的列作为主键，可以声明自增序列，以便插入的时候自动填充升序值：

```mysql
# The value of ID can act like a pointer between related items in different tables.
CREATE TABLE t5 (id INT AUTO_INCREMENT, b CHAR (20), PRIMARY KEY (id));

# The primary key can consist of more than one column. Any autoinc column must come first.
CREATE TABLE t6 (id INT AUTO_INCREMENT, a INT, b CHAR (20), PRIMARY KEY (id,a));
```

虽然表在没有定义主键的情况下可以正常工作，但主键涉及性能的许多方面，对于任何大型或常用表来说，它都是一个至关重要的设计。建议在`CREATE TABLE`语句中指定主键。如果创建表，加载数据之后，再来执行`ALTER TABLE`来添加主键，那么这个速度比创建表的时候定义要慢得多。

##### 查看 InnoDB 表属性

执行`SHOW TABLE STATUS`来查看 InnoDB 表属性：

```mysql
mysql> SHOW TABLE STATUS FROM test LIKE 't%' \G;
*************************** 1. row ***************************
           Name: t1
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2021-02-18 12:18:28
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment:
```

你也可以通过查询 InnoDB Information Schema 系统表来访问 InnoDB 表属性：

```mysql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME='test/t1' \G
*************************** 1. row ***************************
     TABLE_ID: 1144
         NAME: test/t1
         FLAG: 33
       N_COLS: 5
        SPACE: 30
   ROW_FORMAT: Dynamic
ZIP_PAGE_SIZE: 0
   SPACE_TYPE: Single
 INSTANT_COLS: 0
```

#### 从外部创建数据库表

从外部创建表的意思是，在数据库目录之外创建表，从外部创建 InnoDB 表有不同的原因，例如，需要优化空间管理，IO 优化或在具有特定性能或容量特征的存储设备上放置表。

InnoDB 支持以下外部创建表的方法：

- 使用`DATA DIRECTORY`语句
- 使用`CREATE TABLE ... TABLESPACE`语句
- 在外部常规表空间中创建表

##### 使用`DATA DIRECTORY`语句

通过在`CREATE TABLE`语句中指定`DATA DIRECTORY`可以在外部目录中创建 InnoDB 表：

```mysql
CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '/external/directory';
```

`DATA DIRECTORY`语句适用于在每个表的文件表空间中创建的表。当`innodb_file_per_table`变量处于启用状态（默认情况）下，表将隐式地在文件表空间中创建。

```mysql
mysql> SELECT @@innodb_file_per_table;
+-------------------------+
| @@innodb_file_per_table |
+-------------------------+
|                       1 |
+-------------------------+
```

当你在`CREATE TABLE`语句中指定`DATA DIRECTORY`时，将在指定目录下创建表的数据文件(table_name.ibd)

下面的示例演示如何使用`DATA DIRECTORY`语句在外部目录中创建表，假设`innodb_file_per_table`已经启用，并且 InnoDB 知道该目录：

```mysql
mysql> USE test;
Database changed

mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '/external/directory';

# MySQL creates the table's data file in a schema directory
# under the external directory

$> cd /external/directory/test
$> ls
t1.ibd
```

##### 使用`CREATE TABLE ... TABLESPACE`语句

`CREATE TABLE ... TABLESPACE`语句可以和`DATA DIRECTORY`结合使用，用于在外部目录中创建表。为此，需要指定`innodb_file_per_table`作为表空间名称。

```mysql
mysql> CREATE TABLE t2 (c1 INT PRIMARY KEY) TABLESPACE = innodb_file_per_table
       DATA DIRECTORY = '/external/directory';
```



# 参考资料

- [InnoDB Architecture](https://dev.mysql.com/doc/refman/8.0/en/innodb-architecture.html)