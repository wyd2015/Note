# mysql中的binlog模式

mysql中binlog日志用于记录数据被修改的相关信息。查询方式为：

```sql
mysql> show variables like '%log_bin%';
+---------------------------------+-----------------------------+
| Variable_name                   | Value                       |
+---------------------------------+-----------------------------+
| log_bin                         | ON                          |
| log_bin_basename                | /var/lib/mysql/binlog       |
| log_bin_index                   | /var/lib/mysql/binlog.index |
| log_bin_trust_function_creators | OFF                         |
| log_bin_use_v1_row_events       | OFF                         |
| sql_log_bin                     | ON                          |
+---------------------------------+-----------------------------+
6 rows in set (0.00 sec)
```

binlog有三种模式：`Statement Level`、`Row Level`、`Mixed`。

## Statement Level

只记录master上所执行语句的细节，以及执行语句时的上下文信息。每一条修改数据的sql都会记录到master的binlog中，slave在复制时sql线程会解析成和原来master端执行过得相同的语句来执行。

### 优点

不需要记录每一行的数据变化，减少了binlog的日志量，节约IO，提高性能。

### 缺点

很多新功能的加入在复制时容易出问题。

## Row Level

记录所有改动过的数据行。

### 优点

在该模式下，binlog中可以不记录执行的sql语句的上下文相关信息，仅仅记录哪一条数据被修改了，因此，该模式下产生的日志内容可以非常清楚的记录下每一行数据的修改细节，方便理解，而且也不会出现某些情况下的存储过程，或function、trigger的调用和触发无法被正确复制的问题。

### 缺点

由于记录的细节点多，可能会产生大量的日志。

### Mixed

前两种混合的模式。根据执行的每一条sql自动选择记录日志的方式。