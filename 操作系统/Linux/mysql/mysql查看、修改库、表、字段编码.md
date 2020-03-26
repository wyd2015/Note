# 查看和修改 mysql库、表、字段编码
mysql的utf8编码的一个字符最多3个字节，但一个emoji表情是4字节，因此utf8不支持emoji表情，但utf8的超集utf8mb4一个字符最多可由4个字节组成，能支持emoji表情存储。

## 查看编码
```sql
-- 查看数据库编码
show variables like '%char%';

-- 查看数据表编码
show create table table_name;

-- 查看表字段编码
show full columns from table_name;
```

## 修改编码
```sql
-- 修改数据库编码
alter database db_name character set utf8mb4;

-- 修改数据表编码
alter table table_name character set utf8mb4;

-- 修改表字段编码
alter table table_name modify column column_field filed_type character set utf8mb4_unicode_ci;
-- 示例
ALTER TABLE sys_blog MODIFY COLUMN description VARCHAR(512) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
````