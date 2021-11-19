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