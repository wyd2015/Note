---
title: ' Illegal mix of collations (utf8_general_ci,IMPLICIT) and (utf8mb4_general_ci,COERCIBLE) for operation "="'
date: 2019-04-18 15:56:30
tag: mysql
---
今天项目中出现一个问题，服务端log报错
`java.sql.SQLException: Illegal mix of collations (utf8_general_ci,IMPLICIT) and (utf8mb4_general_ci,COERCIBLE) for operation '='`

日志如下：
```yml
19-04-18 15:38:40.608 [http-nio-8082-exec-12] DEBUG c.t.y.d.M.getAppoint-181 -==>  Preparing: select * from monitor_choose_lib where source_title=? and deal_gid is not null 
19-04-18 15:38:40.608 [http-nio-8082-exec-12] DEBUG c.t.y.d.M.getAppoint-181 -==> Parameters: ​​​中国人的智慧！//@占豪:这个真牛！//@绝版出版社:好牛逼 山西(String)
19-04-18 15:38:40.609 [http-nio-8082-exec-12] ERROR druid.sql.Statement-149 -{conn-610002, pstmt-629604} execute error. select * from monitor_choose_lib where  source_title=? and deal_gid is not null
java.sql.SQLException: Illegal mix of collations (utf8_general_ci,IMPLICIT) and (utf8mb4_general_ci,COERCIBLE) for operation '='
  at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:964)
  at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3973)
  at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3909)
  at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2527)
  at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2680)
  at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2487)
  at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:1858)
  at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:1197)
```
这种问题出现的原因是待比较的字符串中可能存在一些类似表情的编码(utf8mb4)字符，而数据库表中存储该值的字段编码(utf8)不支持此种格式。

MySQL查看数据库表的字符编码
```sql
mysql> show create table monitor_choose_lib\G;
*************************** 1. row ***************************
       Table: monitor_choose_lib
Create Table: CREATE TABLE `monitor_choose_lib` (
  `lib_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '选题库ID',
  `source_id` int(11) DEFAULT NULL COMMENT '消息id',
  `source_title` varchar(255) DEFAULT NULL COMMENT 'title',
  `choose_gid` int(11) DEFAULT NULL COMMENT '推送该条消息的组id',
  `choose_gname` varchar(250) DEFAULT NULL COMMENT '来源于选题库',
  `deal_gid` int(11) DEFAULT NULL COMMENT '处理这条消息的组id',
  `deal_gname` varchar(250) DEFAULT NULL COMMENT '指派给选题库',
  `choose_type` tinyint(4) DEFAULT NULL COMMENT '选题类型： 1-热点选题  2-正选选题 3-备选选题',
  `is_delete` tinyint(4) DEFAULT '0' COMMENT '是否删除：  1：是  0：否',
  PRIMARY KEY (`lib_id`)
) ENGINE=InnoDB AUTO_INCREMENT=488 DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```
可以看到 `monitor_choose_lib` 表的字符编码是 `utf8`，由于表字段没有指定字符编码，因此采用表的编码格式，即utf8，并不是 `utf8mb4`。  

因此，解决方案：  
更改数据表指定字段的字符编码为 `utf8mb4`：  
```sql
alter table monitor_choose_lib modify column source_title varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```