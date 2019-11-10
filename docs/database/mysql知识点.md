# 1、mysql简介
## 1.1 什么是MySQL?
- MySQL 是一种关系型数据库
- MySQL的默认端口号是3306

# 2、mysql存储引擎
## 3.1 常用命令
- 查看MySQL提供的所有存储引擎

```mysql
mysql> show engines;
```
查看MySQL当前默认的存储引擎

```mysql
mysql> show variables like '%storage_engine%';
```

查看表的存储引擎

```mysql
show table status like "table_name" ;
```