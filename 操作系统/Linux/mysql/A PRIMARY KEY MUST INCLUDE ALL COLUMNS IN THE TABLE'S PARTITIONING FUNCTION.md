#  A PRIMARY KEY MUST INCLUDE ALL COLUMNS IN THE TABLE'S PARTITIONING FUNCTION

MySQL主键限制，每一个分区表使用的列，必须包含在`primaryKey/uniqueKey`中。



MySQL官方文档：

>18.5.1. Partitioning Keys, Primary Keys, and Unique Keys.
>
>This section discusses the relationship of partitioning keys with primary keys and unique keys. The rule governing this relationship can be expressed as follows: All columns used in the partitioning expression for a partitioned table must be part of every unique key that the table may have.
>
>In other words,every unique key on the table must use every columnin the table's partitioning expression. (This also includes the table's primary key, since it is by definition a unique key. This particular case is discussed later in this section.) For example, each of the following table creation statements is invalid: 

简单来说就是：

>分区字段必须包含在主键字段内，至于为什么MYSQL会这样考虑，为了确保主键的效率；否则同一主键区的东西一个在Ａ分区，一个在Ｂ分区，显然会比较麻烦。

## 解决办法

建表时将分区字段放到主键中，组成复合主键。

```sql
CREATE TABLE T1 (
     id int(8) NOT NULL AUTO_INCREMENT,
     createtime datetime NOT NULL,
     PRIMARY KEY (id,create_time)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
PARTITION BY RANGE COLUMNS(create_time)(
	PARTITION p201903 VALUES LESS THAN ( '20190401' ),
    PARTITION p201904 VALUES LESS THAN ( '20190501' ),
    PARTITION p201905 VALUES LESS THAN ( '20190601' ),
    PARTITION p201906 VALUES LESS THAN ( '20190701' ),
    PARTITION p201907 VALUES LESS THAN ( '20190801' ),
    PARTITION p201908 VALUES LESS THAN ( '20190901' ),
    PARTITION p201909 VALUES LESS THAN ( '20191001' ),
    PARTITION p201910 VALUES LESS THAN ( '20191101' ),
    PARTITION p201911 VALUES LESS THAN ( '20191201' ),
    PARTITION p201912 VALUES LESS THAN ( '20200101' ),
    PARTITION p202001 VALUES LESS THAN ( '20200201' ),
    PARTITION p202002 VALUES LESS THAN ( '20200301' ),
    PARTITION p202003 VALUES LESS THAN ( '20200401' )
);
```

