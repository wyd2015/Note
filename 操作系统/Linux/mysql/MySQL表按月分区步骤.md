# MySQL数据表按月自动分区实践

## 场景

对一张数据表按时间分区，每个月自动创建分区，并删除1个老分区，只保留近3个月的分区。

## 主要步骤

1. 确定分区的字段、类型；
2. 建表时，将主键和分区字段放到主键中，可以理解为作为联合主键来使用；
3. 确定分区间隔；
4. 编写存储过程实现按月分区；
5. 编写事件，自动执行分区操作。

## 说明

### 1. mysql分区原理

每个分区都有自己独立的数据、索引文件的存放目录。本质上，一个分区对应一个磁盘文件，因此，分区越多，磁盘文件也就越多。

### 2. mysql分区类型

主要由四种方式：

- 按Hash分区
- 按Range分区
- 按Key分区
- 按List分区

#### 2.1 HASH分区

基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL 中有效的、产生非负整数值的任何表达式。

要使用HASH分区来分割一个表，要在CREATE TABLE 语句上添加一个“PARTITION BY HASH (expr)”子句，其中“expr”是一个返回一个整数的表达式。它可以仅仅是字段类型为MySQL 整型的一列的名字。此外，你很可能需要在后面再添加一个“PARTITIONS num”子句，其中num 是一个非负的整数，它表示表将要被分割成分区的数量。

有两种：常规Hash和线性Hash

##### 常规HASH

基于`分区个数`取模运算（id%n），使用取模运算得到的余数来确定分区范围，并将数据进行插进去。使用常规hash方式可以让数据平均分不到每个分区，如分4个区，使用id取余时，余数的值仅在`0,1,2,3`四个数中，分散比较均匀。

但常规hash也有缺点：由于分区的规则在建表时就要确定，如果需要临时新增/减少分区，运算规则发生了变化，原来已经入库的数据，就得根据新的运算规则做迁移。针对这种问题，提出了`线性HASH`的方式：

##### 线性HASH

实际是`一致性HASH算法`，使用虚拟节点方式，解决上面常规HASH分区运算规则发送变化后的数据迁移问题。这里需要说明一下，使用线性HASH依然需要迁移数据，不过相对来说迁移的数据量很小。

线性哈希功能使用的一个线性的2的幂（powers-of-two）运算法则。

线性哈希分区和常规哈希分区在语法上的唯一区别在于，在“PARTITION BY”子句中添加“LINEAR”关键字。

##### 弊端

不能删除分区，也就不能使用`DROP PARTITION`来操作分区。

### 2.2 Range分区

基于属于一个给定连续区间的列值，把多行分配给分区。这些区间要连续且不能相互重叠，使用VALUES LESS THAN操作符来进行定义。

注意，每个分区都是按顺序进行定义，从最低到最高。这是PARTITION BY RANGE 语法的要求；在这点上，它类似于C或Java中的“switch ... case”语句。

### 2.3 Key分区

类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数，必须有一列或多列包含整数值。

### 2.4 List分区

可以把list看成是在range方式的基础上的改进版。list和range本质都是基于范围，自己控制范围，range是列出范围，比如1-2000范围算一个分区，这样是一个连续的值。

LIST分区通过使用“PARTITION BY LIST(expr)”来实现，其中“expr”是某列值或一个基于某个列值、并返回一个整数值的表达式，然后通过“VALUES IN (value_list)”的方式来定义每个分区，其中“value_list”是一个通过逗号分隔的整数列表。

### 3. 分区字段处理

#### 3.1 确定数据库字段类型：

```mysql
mysql> desc rmd_msg_sens;
+------------------+------------------+------+-----+---------+----------------+
| Field            | Type             | Null | Key | Default | Extra          |
+------------------+------------------+------+-----+---------+----------------+
| rmd_id           | int(11) unsigned | NO   | PRI | NULL    | auto_increment |
| create_time      | datetime         | NO   | PRI | NULL    |                |
| msg_key          | varchar(500)     | NO   |     | NULL    |                |
| msg_u_g_ch_key   | varchar(200)     | YES  | MUL | NULL    |                |
| msg_publish_time | datetime         | YES  | MUL | NULL    |                |
| msg_abstract     | varchar(1000)    | YES  |     | NULL    |                |
| msg_url          | varchar(500)     | YES  |     | NULL    |                |
| situation        | tinyint(4)       | YES  |     | NULL    |                |
| is_delete        | tinyint(1)       | YES  |     | 0       |                |
| msg_board        | varchar(100)     | YES  |     | NULL    |                |
| msg_website      | varchar(100)     | YES  |     |         |                |
| msg_uname        | varchar(500)     | YES  |     | NULL    |                |
| msg_reply_num    | int(11)          | YES  |     | 0       |                |
| msg_forward_num  | int(11)          | YES  |     | 0       |                |
| msg_read_num     | int(11)          | YES  |     | 0       |                |
| msg_like_num     | int(11)          | YES  |     | 0       |                |
| msg_sentiment    | int(11)          | YES  |     | 0       |                |
| website_id       | int(11)          | YES  | MUL | NULL    |                |
| msg_title        | varchar(1000)    | YES  |     | NULL    |                |
| msg_key_ix       | bigint(20)       | NO   | MUL | 0       |                |
| rule_id          | int(11)          | NO   |     | NULL    |                |
+------------------+------------------+------+-----+---------+----------------+

DROP TABLE IF EXISTS `rmd_msg_sens`;
CREATE TABLE `rmd_msg_sens`  (
  `rmd_id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `create_time` datetime(0) NOT NULL COMMENT '创建时间',
  `msg_key` varchar(500) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '通道内数据的唯一标识',
  `msg_u_g_ch_key` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '该条消息的用户标识',
  `msg_publish_time` datetime(0) NULL DEFAULT NULL COMMENT '消息生成/发布时间',
  `msg_abstract` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '文档摘要',
  `msg_url` varchar(500) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '原文链接',
  `situation` tinyint(4) NULL DEFAULT NULL COMMENT '舆论场，10：新浪，20：微信，30：新闻，40：新闻评论，多舆论场：100',
  `is_delete` tinyint(1) NULL DEFAULT 0 COMMENT '是否删除 1已删除',
  `msg_board` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '文档频道',
  `msg_website` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT '' COMMENT '文档来源网站',
  `msg_uname` varchar(500) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '昵称',
  `msg_reply_num` int(11) NULL DEFAULT 0 COMMENT '回复数',
  `msg_forward_num` int(11) NULL DEFAULT 0 COMMENT '转发数',
  `msg_read_num` int(11) NULL DEFAULT 0 COMMENT '阅读数',
  `msg_like_num` int(11) NULL DEFAULT 0 COMMENT '点赞数',
  `msg_sentiment` int(11) NULL DEFAULT 0 COMMENT '消息倾向性,取值[0,100]',
  `website_id` int(11) NULL DEFAULT NULL COMMENT '网站ID',
  `msg_title` varchar(500) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '文档标题，如果信息类型为微博的话，标题则为内容的前1000个字',
  `msg_key_ix` bigint(20) NOT NULL DEFAULT 0,
  `sens_type` tinyint(2) NULL DEFAULT NULL COMMENT '1:敏感词；2：敏感模型；4：敏感作者',
  `match_words` varchar(2000) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '命中的敏感词，最多保存500个汉字',
  `match_authors` varchar(2000) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '命中的人名，500个人，每人名字3个字',
  `is_read` tinyint(2) NULL DEFAULT 0 COMMENT '已读：1；未读：0；',
  `loc_country` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '城市',
  `loc_province` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL COMMENT '省份',
  PRIMARY KEY (`rmd_id`, `create_time`) USING BTREE,
  INDEX `ix_msg_ptime`(`msg_publish_time`) USING BTREE,
  INDEX `ix_msg_key_ix`(`msg_key_ix`) USING BTREE,
  INDEX `ix_msg_u_g_ch_key`(`msg_u_g_ch_key`) USING BTREE COMMENT '消息作者对应索引',
  INDEX `ix_website_id`(`website_id`) USING BTREE COMMENT 'msg_website为空时查询的字段对应的索引'
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci COMMENT = '选题推荐-敏感推荐' ROW_FORMAT = Compact PARTITION BY RANGE COLUMNS (`create_time`)
PARTITIONS 2
(PARTITION `p202004` ENGINE = InnoDB MAX_ROWS = 0 MIN_ROWS = 0 ,
PARTITION `p202005` ENGINE = InnoDB MAX_ROWS = 0 MIN_ROWS = 0 )
;
```

这里使用create_time字段来分区。

MySQL主键的限制，每一个分区表中的公式中的列，必须在主键/unique key 中包括。

这里注意建表语句中对主键的定义，使用的复合主键，也即id+time的方式来唯一确定一条信息。

```mysql
PRIMARY KEY (`rmd_id`, `create_time`) USING BTREE,
```

#### 3.2 分区准备

- MySQL主键的限制，每一个分区表中的公式中的列，必须在主键/unique key 中包括；
- 建表时必须先有至少一个分区，然后才能自动去创建新分区；
- 确定分区个数；
- 确定分区的名称样式，这里以`p202004`的形式作为分区名，表示2020年4月份的数据均保存在该分区下；
- 创建存储过程；
- 创建事件

### 4. 存储过程

```mysql
-- ----------------------------
-- Procedure structure for proc_handle_partition_by_month
-- ----------------------------
DROP PROCEDURE IF EXISTS `proc_handle_partition_by_month`;
delimiter ;;
-- 这条设置，保证了存储过程的字符串与数据库字符序一致，避免存储过程中数据比较时产生问题
SET collation_database=utf8mb4_0900_ai_ci;
CREATE PROCEDURE `proc_handle_partition_by_month`()
BEGIN
	DECLARE i int(11) DEFAULT 2;
	DECLARE countNum int(11);
	DECLARE partition_num INT DEFAULT 0;
	DECLARE done INT DEFAULT 0;
	DECLARE tableName VARCHAR(100) DEFAULT '';
	DECLARE sql_add_partition_i VARCHAR(1000) DEFAULT '';
	DECLARE drop_partition_name VARCHAR(255) DEFAULT '';
	DECLARE max_partition_name VARCHAR(255) DEFAULT '';
	DECLARE cur_tableName CURSOR FOR SELECT t.TABLE_NAME FROM information_schema.TABLES AS t WHERE t.TABLE_NAME REGEXP 'rmd_msg_[0-9]' OR t.TABLE_NAME REGEXP 'rmd_msg_rel_[0-9]' OR t.TABLE_NAME REGEXP 'rmd_msg_sens_*';
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;
	SELECT count(*) INTO countNum FROM information_schema.TABLES AS t WHERE t.TABLE_NAME REGEXP 'rmd_msg_[0-9]' OR t.TABLE_NAME REGEXP 'rmd_msg_rel_[0-9]' OR t.TABLE_NAME REGEXP 'rmd_msg_sens_*';
	
	OPEN cur_tableName;
		out_loop: LOOP
		FETCH cur_tableName INTO tableName;

		IF done THEN
			LEAVE out_loop;
		END IF;
		
		SELECT partition_name INTO max_partition_name FROM information_schema.partitions where table_name = tableName order by partition_description desc limit 1;
		IF ISNULL(max_partition_name)=1 THEN
			SET i=2;
			WHILE i>=0 DO
				SET @p_name_i=date_format(date_sub(curdate(), interval i month),'%Y%m');
				SET @max_date_i=date_format(date_add(curdate()-day(curdate())+1, interval -(i-1) month),'%Y%m%d');
				SET @sql_add_partition=CONCAT('ALTER TABLE ',tableName,' PARTITION BY RANGE COLUMNS(create_time)(','PARTITION p',@p_name_i,' VALUES LESS THAN (\'',@max_date_i,'\'));');
				PREPARE stmt FROM @sql_add_partition;
				EXECUTE stmt;
				DEALLOCATE PREPARE stmt;
				SET i=i-1;
			END WHILE;			
		END IF;		
		
		SET @p_name=date_format(date_add(curdate()-day(curdate())+1, interval 1 month),'%Y%m');
		SET @max_date=date_format(date_add(curdate()-day(curdate())+1, interval 2 month),'%Y%m%d');
		IF CONCAT('p',@p_name)=max_partition_name THEN
			SET countNum = countNum - 1;
			ITERATE out_loop;
		END IF;		
		
		SET @sql_add_partition=CONCAT('ALTER TABLE ',tableName,' ADD PARTITION ( PARTITION p',@p_name,' VALUES LESS THAN (\'',@max_date,'\'));');
		PREPARE stmt FROM @sql_add_partition;
		EXECUTE stmt;
		DEALLOCATE PREPARE stmt;
		
		SELECT count(*) INTO partition_num FROM information_schema.partitions where table_name = tableName;		
		IF partition_num <= 3 THEN
			SET countNum = countNum - 1;
			ITERATE out_loop;
		END IF;
		
		SET @partition_num=0;
		SELECT count(*) INTO @partition_num FROM information_schema.partitions where table_name=tableName;
		SET i = @partition_num-3;
		IF i>0 THEN
			WHILE i>0 DO
				SELECT partition_name INTO drop_partition_name FROM information_schema.partitions where table_name=tableName order by partition_description asc limit 1;
				SET @sql_drop_partition=CONCAT('ALTER TABLE ',tableName,' DROP PARTITION ',drop_partition_name,';');
				PREPARE stmt from @sql_drop_partition;
				EXECUTE stmt;
				DEALLOCATE PREPARE stmt;
				SET i=i-1;
			END WHILE;
		END IF;		
		
		SET done = 0;
		SET countNum = countNum - 1;
		END LOOP out_loop;
	CLOSE cur_tableName;
END
;;
delimiter ;
```

### 5. 创建事件

创建事件是为了让MySQL自动执行建立分区的操作。

```mysql
-- ----------------------------
-- Event structure for event_handle_partition_monthly
-- ----------------------------
DROP EVENT IF EXISTS `event_handle_partition_monthly`;
delimiter ;;
SET collation_database=utf8mb4_0900_ai_ci;
CREATE EVENT `event_handle_partition_monthly`
ON SCHEDULE
EVERY '1' MONTH STARTS '2020-04-28 00:00:00'
COMMENT '每月28号调用存储过程，生成/删除推荐相关表的分区'
DO CALL proc_add_partition_by_month()
;;
delimiter ;
```

### 6. 执行sql

新建分区的操作，需要重新建表，注意备份数据。



