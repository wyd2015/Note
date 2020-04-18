# MySQL查看存储过程

## 执行环境

- Linux
- mysql5.7

`mysql8`由于调整了`数据字典`的结构，所以已经没有`mysql.proc`表了。所以，以下使用到`mysql.proc`的命令只适合MySQL5.7及以下版本。

## 查看存储过程

1. 查看所有数据库的存储过程/函数

   ```sql
   -- 查看存储过程
   show procedure status \G;
   
   --查看函数
   show function status \G;
   ```

2. 查看指定数据库中的存储过程和函数

   ```sql
   -- 查看指定数据库的存储过程：db为数据库名称，type固定为PROCEDURE
   select name from mysql.proc where db='test' and type='PROCEDURE';
   
   -- 查看指定数据库的存储过程：name为存储过程名称
   select name from mysql.proc where db='test' and type='PROCEDURE' and name='****';
   ```

3. 查看指定存储过程的内容

   ```sql
   -- 查看名称为proc_name的存储过程内容
   show create procedure proc_name;
   
   -- 查看名称为proc_name的函数内容
   show create function func_name;
   ```

## 查看触发器

```sql
--语法结构
show triggers [from db_name] [like expression];

--示例
SELECT * FROM triggers T WHERE trigger_name='my_trigger' \G；
```

