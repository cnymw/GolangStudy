# mysql 慢查询工具-mysqldumpslow 详解

Mysql 慢查询日志包含有关执行时间较长的查询的信息。mysqldumpslow 解析 Mysql 慢查询日志文件并总结其内容。

通常，mysqldumpslow 聚集类似的查询，但数字和字符串数据值的特定值除外。它在显示摘要输出时将这些值"抽象"为 N 和 S，要修改值抽象行为，请使用 -a 和 -n 选项。

调用 mysqldumpslow，如下所示：

```bash
mysqldumpslow [options] [log_file ...]
```

未使用选项的输出示例：

```bash
Reading mysql slow query log from /usr/local/mysql/data/mysqld57-slow.log
Count: 1  Time=4.32s (4s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t2 select * from t1

Count: 3  Time=2.53s (7s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t2 select * from t1 limit N

Count: 3  Time=2.13s (6s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t1 select * from t1
```

## mysqldumpslow 选项

mysqldumpslow 支持以下选项：

选项名称|描述
---|:--:
-a|不要将所有数字抽象为 N，不要将字符串抽象为 S
-n|指定需要查询的最少数量
--debug|输出 debug 信息
-g|只输出符合指定正则规则的语句，后边可以写正则来匹配输出
--help|打印帮助信息
-h|指定在慢查询日志里的服务器 host
-i|服务器实例的名称
-l|不要从总时间中减去锁定时间
-r|反转排序顺序
-s|指定如何排序输出
-t|仅显示前 n 个查询，top n的意思
--verbose|详细模式

-s 支持以下 sort_type：

- t，at：按查询时间或平均查询时间排序
- l，al：按锁定时间或平均锁定时间排序
- r，ar：按返回的条目数或平均返回的条目数排序
- c：按 sql 数量排序，也就是该 sql 执行的总的次数

例如，要得到返回记录最多的 10 个 sql，可以执行以下命令：

```bash
mysqldumpslow -s r -t 10 mysql06_slow.log
```

得到 sql 执行次数最多的10个 sql：

```bash
mysqldumpslow -s c -t 10 mysql06_slow.log
```

得到按照总时间排序的前 10 条里面含有左连接的查询语句。

```bash
mysqldumpslow -s t -t 10 -g "left join" mysql06_slow.log
```
