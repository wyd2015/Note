# MySQL常用命令
```sql
-- 插入新数据
insert into table_name values (id, column1, column2, ...), (id, column1, column2, ...);

-- 修改符合条件的数据
update table_name set column_field = targetValue where column = conditions;

-- 删除符合条件的数据
delete from table_name where column = conditions;

-- 重命名表
alter table table_name rename t_name;

-- 添加字段
alter table sys_user add column is_read tinyint(2) default 0;

-- 修改字段属性
alter table sys_department modify dep_id int(11) auto_increment primary key;

-- 重命名列
alter table table_name change column_field_1 column_field_2 int(2);

-- 删除指定列
alter table table_name drop column column_field;

-- 添加索引
alter table table_name add index 索引名(field1, field2);

-- 删除索引
alter table table_name drop index emp_name;

-- 调用存储过程
call proc_name(param1, param2);
```