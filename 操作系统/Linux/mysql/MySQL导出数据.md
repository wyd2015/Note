# MySQL导出数据
```sql
-- 导出多张表的结构和数据到指定文件, 在--tables后依次写上表名，用空格隔开即可
mysqldump -uroot -p db_name --tables sys_organ sys_user sys_role sys_role_user sys_department sys_dep_user > ~/sqldata/user_relates.sql

-- 导出符合条件的表数据到指定文件
mysqldump -u用户名 -p密码 -h mysql主机  --default-character-set=指定编码  数据库名称  表名称  --where=" 查询条件 " > 导出文件名.sql

-- 导出存储过程到指定文件
mysqldump -uroot -p -hlocalhost -P3306 -n -d -t -R db_name > procedure_name.sql

mysqldump -u{username} -p{password} --routines --no-data --no-create-d --skip-opt --no-create-info {database} > output_file.sql
```