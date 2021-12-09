# InnoDB 存储引擎

InnoDB 是事务安全的 MySQL 存储引擎，设计上采用了类似于 Oracle 数据库的结构。

## InnoDB 后台线程

InnoDB 后台线程的主要作用是负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据。

此外将已修改的数据文件刷新到磁盘文件，保证在数据库发生异常的情况下 InnoDB 能回复到正常运行状态。

### 1. Master Thread

主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新，合并插入缓冲（INSERT BUFFER），UNDO 页的回收等。

### 2. IO Thread

InnoDB 存储引擎大量使用 AIO（Async IO）来处理写 IO 请求，这样可以极大提高数据库的性能。

IO Thread 的工作主要是负责这些 IO 请求的回调（call back）处理。

### 3. Purge Thread

事务被提交后，其所使用的 undolog 可能不再需要，因此需要 PurgeThread 来回收已经使用并分配的 undo 页。

### 4. Page Cleaner Thread

作用是将之前版本中脏页的刷新操作都放入到单独的线程中来完成。目的是为了减轻 Master Thread 的工作及对于用户查询线程的阻塞。

## 内存

### 1. 缓冲池

缓冲池就是一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。

在数据库中进行读取页的操作，首页判断页是否在缓冲池中，若在缓冲池中，称该页在缓冲池中被命中，否则，读取磁盘上的页。

对于数据库中页的修改操作，首页修改在缓冲池中的页，然后以一定的频率刷新到磁盘上。这里通过一种称为 *Checkpoint* 的机制刷新回磁盘。

缓冲池中缓存的数据页类型有：索引页，数据页，undo 页，插入缓冲页（insert buffer），自适应哈希索引（adaptive hash index），InnoDB 存储的锁信息（lock info），数据字典信息（data
dictionary）

### 2. LRU List，Free List 和 Flush List

在 InnoDB 引擎中，缓冲池对传统 LRU 算法做了一些优化，使用了称为 midpoint insertion strategy 的算法来管理页。

新读到的页，虽然是最新访问的页，但并不是直接放入到 LRU 列表的首部，而是放入到 LRU 列表的 midpoint 位置。