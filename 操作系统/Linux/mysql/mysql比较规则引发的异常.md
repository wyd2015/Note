# MySQL由比较规则collation引起的执行异常
项目中在使用存储过程新建表时报错：  
`Procedure execution failed
```yml
1267 - Illegal mix of collations (utf8mb4_general_ci,IMPLICIT) and (utf8mb4_unicode_ci,IMPLICIT) for operation '='
```
存储过程如下：  
```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_clean_table`()
BEGIN
DECLARE num int(11);
DECLARE cfgGroupId int(11);
DECLARE countNum int(11);
DECLARE done INT DEFAULT 0;
DECLARE tableName VARCHAR(100) DEFAULT '';
DECLARE sql_clean_builder VARCHAR(10000) DEFAULT '';
DECLARE cur_cfgGroupId CURSOR FOR SELECT t.cfg_group_id FROM alert_config_group t;
DECLARE cur_tableName CURSOR FOR SELECT t.TABLE_NAME FROM information_schema.TABLES AS t WHERE t.table_schema='db_area' AND (t.TABLE_NAME REGEXP '^monitor_[a-zA-Z]*_[0-9]*$' OR t.TABLE_NAME REGEXP '^monitor_[a-zA-Z]*_[a-zA-Z]*_[0-9]*$');
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
SELECT count(*) into num from alert_config_group t;
SELECT count(*) into countNum from information_schema.TABLES AS t WHERE t.table_schema='db_area' AND (t.TABLE_NAME REGEXP '^monitor_[a-zA-Z]*_[0-9]*$' OR t.TABLE_NAME REGEXP '^monitor_[a-zA-Z]*_[a-zA-Z]*_[0-9]*$');
OPEN cur_tableName;
  out_loop: LOOP
  FETCH cur_tableName INTO tableName;

  IF done THEN
    LEAVE out_loop;
  END IF;

  SET @countName1 = 0;
  OPEN cur_cfgGroupId;
    SET done = 0;
    REPEAT
    FETCH cur_cfgGroupId INTO cfgGroupId;
    SET @t_name_1 = CONCAT('monitor_alert_', cfgGroupId);

    ---------- 在此处比较时报错 ---------
    IF @t_name_1 = tableName THEN
      SET @countName1 = 1;
    END IF;
    -----------------------------------

    IF num = 1 THEN
      set done =1;
    END IF;
    SET num = num - 1;
    UNTIL done END REPEAT;
  CLOSE cur_cfgGroupId;

  IF @countName1 = 0 THEN
    SET sql_clean_builder = CONCAT('DROP TABLE IF EXISTS `', tableName, '`;');
    SET @_sql = sql_clean_builder;
    PREPARE sql_builder from @_sql;
    EXECUTE sql_builder;
  END IF;
  SET done = 0;

  SET countNum = countNum - 1;
  END LOOP out_loop;
  CLOSE cur_tableName;
END
```

在数据库查看数据库比较规则如下：  
```sql
mysql> show variables like 'collation_%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8_general_ci    |
| collation_database   | utf8mb4_unicode_ci |
| collation_server     | utf8mb4_unicode_ci |
+----------------------+--------------------+

mysql> show procedure status \G;
*************************** 1. row ***************************
                  Db: db_area
                Name: proc_clean_cfg_table
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2019-03-05 10:31:47
             Created: 2019-03-05 10:31:47
       Security_type: DEFINER
             Comment: 
character_set_client: utf8mb4
collation_connection: utf8mb4_general_ci
  Database Collation: utf8mb4_unicode_ci
```

此时的想法：
>唯独collation_connection是utf8mb4_general_ci，其他两个数据库级别collation_database和collation_server均为utf8mb4_unicode_ci，
想着是不是这里比较规则不一致引起的问题