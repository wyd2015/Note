# MySQL由比较规则collation引起的执行异常
## 现场还原

项目中在执行存储过程时报错：

```yml
1267 - Illegal mix of collations (utf8mb4_general_ci,IMPLICIT) and (utf8mb4_unicode_ci,IMPLICIT) for operation '='
```
存储过程如下：  
```sql
DROP PROCEDURE IF EXISTS `proc_clean_cfg_table`;
delimiter ;;
CREATE PROCEDURE `proc_clean_cfg_table`()
BEGIN
DECLARE num int(11);
DECLARE cfgGroupId int(11);
DECLARE countNum int(11);
DECLARE done INT DEFAULT 0;
DECLARE tableName VARCHAR(100) DEFAULT '';
DECLARE sql_clean_builder VARCHAR(10000) DEFAULT '';
DECLARE cur_cfgGroupId CURSOR FOR select t.cfg_group_id from alert_config_group t;
DECLARE cur_tableName CURSOR FOR select t.TABLE_NAME FROM information_schema.TABLES AS t WHERE t.TABLE_NAME REGEXP 'monitor_alert_[0-9]' OR t.TABLE_NAME REGEXP 'monitor_alert_sens_[0-9]' OR t.TABLE_NAME REGEXP 'monitor_topic_doc_[0-9]';
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
SELECT count(*) into num from alert_config_group t;
SELECT count(*) into countNum from information_schema.TABLES AS t WHERE t.TABLE_NAME REGEXP 'monitor_alert_[0-9]' OR t.TABLE_NAME REGEXP 'monitor_alert_sens_[0-9]' OR t.TABLE_NAME REGEXP 'monitor_topic_doc_[0-9]';
															
OPEN cur_tableName;
	out_loop: LOOP
	FETCH cur_tableName INTO tableName;

	IF done THEN
		LEAVE out_loop;
	END IF;
	SET @countName1 = 0;
	SET @countName2 = 0;
	
	OPEN cur_cfgGroupId;
		SET done = 0;
		REPEAT
		FETCH cur_cfgGroupId INTO cfgGroupId;
		
 		IF CONCAT('monitor_alert_', cfgGroupId) = tableName THEN
 			SET @countName1 = 1;
 		END IF;
		IF CONCAT('monitor_alert_sens_', cfgGroupId) = tableName THEN
			SET @countName2 = 1;
		END IF;
		IF CONCAT('monitor_topic_doc_', cfgGroupId) = tableName THEN
			SET @countName2 = 1;
		END IF;
		IF num = 1 THEN
			set done =1;
		END IF;
		SET num = num - 1;
		UNTIL done END REPEAT;
	CLOSE cur_cfgGroupId;
	
	IF @countName1 = 0 AND @countName2 = 0 THEN
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
;;
delimiter ;
```

在数据库查看数据库比较规则如下：  
```sql
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

## 问题出现原因

存储过程的字符序设置不一致导致。

#### 数据库对象的字符集的指定有如下继承关系

Server -> Database -> Table ->Column

也就是说，如果后者没有显示指定字符集，那么将采用前者的字符集。

## 解决思路

既然是mysql中`collation`的设置不一致，那就设置一直不就行了吗？

1. 我尝试时依次将对应数据库的`collation`相关项都设置为：`utf8mb4_general_ci`，没用！

   ```sql
   mysql> show variables like 'collation_%';
   +----------------------+--------------------+
   | Variable_name        | Value              |
   +----------------------+--------------------+
   | collation_connection | utf8mb4_general_ci |
   | collation_database   | utf8mb4_general_ci |
   | collation_server     | utf8mb4_general_ci |
   +----------------------+--------------------+
   ```

2. 后来使用命令查看了这个存储过程的信息才发现一个重要问题：存储过程需要单独设置字符序（collation）!

   ```sql
   mysql> show create procedure proc_clean_cfg_table;
   -- 长度限制，删除了用不到的列，只显示需要的两列
   +---------------------+-----------------------+---------------------+
   | Procedure           | collation_connection  | Database Collation  |
   +---------------------+-----------------------+---------------------+
   | proc_clean_cfg_table|  utf8mb4_general_ci   | utf8mb4_unicode_ci  |
   +-----------+-------------------------------------------------------+
   ```

3. 修改存储过程的sql，添加`SET collation_database=utf8mb4_general_ci;`单独设置具体存储过程的collation，添加位置如下：

   ```sql
   -- ----------------------------
   -- Procedure structure for proc_clean_cfg_table
   -- ----------------------------
   DROP PROCEDURE IF EXISTS `proc_clean_cfg_table`;
   delimiter ;;
   SET collation_database=utf8mb4_general_ci;  --在这里添加就行了！
   CREATE PROCEDURE `proc_clean_cfg_table`()
   BEGIN
    --***
   END
   ;;
   delimiter ;
   ```

4. root登录mysql账户，重新执行该存储过程的sql：`source /home/proc_clean_cfg_table.sql`，再去执行存储过程不再报错！最终字符序也一致了！

   ```sql
   mysql> show create procedure proc_clean_cfg_table;
   +---------------------+-----------------------+---------------------+
   | Procedure           | collation_connection  | Database Collation  |
   +---------------------+-----------------------+---------------------+
   | proc_clean_cfg_table|  utf8mb4_general_ci   | utf8mb4_general_ci  |
   +-----------+-------------------------------------------------------+
   ```

   