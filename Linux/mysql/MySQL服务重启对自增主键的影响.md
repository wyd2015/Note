---
title: '''MySQL服务重启对自增主键的影响'''
date: 2019-03-12 17:39:51
categories: MySQL
tags: MySQL
---
## mysql数据库的user表（id为自增主键）中有5条数据，删除主键ID最大的两条，a,b两种情况下重新插入一条新数据，这条新数据的主键ID是多少？
- a. 删除后直接插入新数据；
- b. 重启MySQL服务后再插入新数据。
```sql
mysql> select * from user;
+----+------+--------+--------+
| id | age  | name   | status |
+----+------+--------+--------+
|  1 |   18 | 甯冧竵 |      0 |
|  2 |   18 | 閰歌彍 |      0 |
|  3 |   18 | 鍖呭瓙 |      0 |
|  4 |   18 | M4     |   NULL |
|  5 |   18 | AK47   |      0 |
+----+------+--------+--------+
5 rows in set (0.00 sec)

mysql> delete from user where id > 3;
Query OK, 2 rows affected (0.24 sec)

mysql> select * from user;
+----+------+--------+--------+
| id | age  | name   | status |
+----+------+--------+--------+
|  1 |   18 | 甯冧竵 |      0 |
|  2 |   18 | 閰歌彍 |      0 |
|  3 |   18 | 鍖呭瓙 |      0 |
+----+------+--------+--------+
3 rows in set (0.00 sec)

mysql> quit
Bye
```

## 针对情况a：删除后直接插入数据，主键ID会接着已经被删除的最大主键ID值自增1后的值入库；
```sql
mysql> insert into user(age,name,status) values(12,'AK47',1);
Query OK, 1 row affected (0.14 sec)

mysql> select * from user;
+----+------+--------+--------+
| id | age  | name   | status |
+----+------+--------+--------+
|  1 |   18 | 甯冧竵 |      0 |
|  2 |   18 | 閰歌彍 |      0 |
|  3 |   18 | 鍖呭瓙 |      0 |
|  5 |   12 | AK47   |      1 |
+----+------+--------+--------+
4 rows in set (0.00 sec)
```
## 针对情况b：重启MySQL服务后，主键ID会根据当前表中最大的id值自增1后的值入库。
1. 重启MySQL服务：
```cmd
λ net stop mysql57
MySQL57 服务正在停止..
MySQL57 服务已成功停止。

λ net start mysql57
MySQL57 服务正在启动 ..
MySQL57 服务已经启动成功。
```
2. 插入新数据：
```sql
mysql> insert into user(age,name,status) values(12,'AK47',1);
Query OK, 1 row affected (0.14 sec)

mysql> select * from user;
+----+------+--------+--------+
| id | age  | name   | status |
+----+------+--------+--------+
|  1 |   18 | 甯冧竵 |      0 |
|  2 |   18 | 閰歌彍 |      0 |
|  3 |   18 | 鍖呭瓙 |      0 |
|  4 |   12 | AK47   |      1 |
+----+------+--------+--------+
4 rows in set (0.00 sec)
```
# 重启服务后，数据库表的自增主键会被重置成当前主键的最大值！！！
```sql
-- mysql重启后，该函数取得的值会重置为当前表现有数据中的最大主键值
-- MySQL不重启，则主键值正常自增。
select LAST_INSERT_ID()
```