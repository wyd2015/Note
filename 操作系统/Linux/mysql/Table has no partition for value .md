# Table has no partition for value from *

## 背景

根据时间字段`create_time`对表按月进行分区，当前时间`2020-04-07 21:34:14`分区语句为：

```sql
alter table rmd_msg_1 partition by range columns(create_time) (
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

插入数据时控制的时间范围为`2019-04-06`~`2020-04-06`，`create_time`的值在此范围内随机生成。

## 问题来源

最新的一个分区`p202003`存储的是2020年3月份的数据，随机生成会有`2020-04-01 00:00:00`之后的时间，这样的时间由于找不到可以匹配的分区存储，因此无法保存`create_time`为`2020-04-01 00:00:00`之后的数据。

## 解决办法

- 添加新分区来存储新数据。说明一个问题，如果确定一张表要分区，必须要先建好分区，才能插入新数据；
- 调整终止时间点到`2020-04-01 00:00:00`之前。