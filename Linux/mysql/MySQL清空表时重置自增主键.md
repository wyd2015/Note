# MySQL清空表时重置自增主键
有两种方法实现：
```sql
-- 1、使用truncate命令，此命令需要高权限才能执行
truncate table_name

-- 2、使用delete命令后，再使用alter
delete from table_name;
alter table table_name anto_increment=1;
```