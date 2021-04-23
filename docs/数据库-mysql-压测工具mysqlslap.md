# mysql 性能压测工具-mysqlslap 详解

mysqlslap 是一个诊断程序，用于模拟 MySQL 服务器的客户端负载，并报告每个阶段的时间。

它的工作原理就像多个客户端正在访问服务器一样。

像如下来调用 mysqlslap：

```bash
mysqlslap [options]
```

有一些选项例如 --create 或 --query，使你能够指定包含 SQL 语句的字符串或包含语句的文件。

如果指定文件，默认情况下，它必须每行包含一条语句（即隐式语句分隔符或换行符）。

使用 --delimiter 选项指定不同的分隔符，这样可以指定跨多行的语句或将多个语句放在一行上。

文件中不能包含注释，mysqlslap 无法理解他们。

mysqlslap 运行分为三个阶段：

1. 创建用于测试的 schema，table 和任何存储程序或数据，此阶段使用单个客户端连接
2. 运行负载测试，此阶段可以使用多个客户端连接
3. 清理（断开连接，如果指定的话，会删除表），此阶段使用单个客户端连接

## 例子

提供你自己的 create 和 query SQL，其中有 50 个客户端进行查询，每个客户端进行 200 次 select（在一行中输入命令）。

```bash
mysqlslap --delimiter=";"
  --create="CREATE TABLE a (b int);INSERT INTO a VALUES (23)"
  --query="SELECT * FROM a" --concurrency=50 --iterations=200
```

让 mysqlslap 用一个包含两个 INT 列和三个 VARCHAR 列的表来构建 query SQL 语句。使用 5 个客户端，每个客户端查询 20 次。不要创建表或插入数据（即使用上一个测试的模式和数据）。

```bash
mysqlslap --concurrency=5 --iterations=20
  --number-int-cols=2 --number-char-cols=3
  --auto-generate-sql
```

告诉程序从指定的文件加载 create，insert 和 query SQL 语句，其中 create.sql 文件有多个以";"分割的表创建语句以及多个由";"分割的 insert 语句。

--query 文件应包含多个以";"分割的查询。运行所有的负载语句，然后使用五个客户端（每个客户端五次）运行查询文件中的所有查询。

```bash
mysqlslap --concurrency=5
  --iterations=5 --query=query.sql --create=create.sql
  --delimiter=";"
```

## 选项

选项名称|描述
---|:--:
--auto-generate-sql|当文件或命令选项中未提供 SQL 语句时，自动生成 SQL 语句
--auto-generate-sql-add-autoincrement|将自动增量列添加到自动生成的表中
--auto-generate-sql-execute-number|指定要自动生成的查询数
--auto-generate-sql-write-number|在每个线程上执行多少行插入
--commit|提交前要执行多少条语句
--concurrency|发出 SELECT 语句时要模拟的客户端数
--create|包含用于创建表的语句的文件或字符串
--create-schema|运行测试的 schema
--delimiter|在 SQL 语句中使用的分隔符
--host|MySQL 服务器所在主机
--iterations|运行测试的次数
--number-char-cols|指定--auto-generate-sql时要使用的 VARCHAR 列数
--number-int-cols|指定了--auto-generate-sql时要使用的 INT 列数
--number-of-queries|将每个客户端限制为大约此数量的查询
--password|连接到服务器时使用的密码
--port|连接的 TCP/IP 端口号
--query|包含用于检索数据的 SELECT 语句的文件或字符串
--user|连接到服务器时使用的 MySQL 用户名
