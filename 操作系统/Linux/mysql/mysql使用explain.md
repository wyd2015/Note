# mysql使用explain

通过explain可以查看sql的执行情况，特别是遇到慢查询时，可以借助explain分析索引是否生效，进一步探索sql的优化方式。

使用explain输出的参数如下：

| 列名          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 执行编号，标识select所属行。如果在语句中没有子查询/关联查询，只有唯一的select，每行都将显示1，否则内层的select语句一般会顺序编号，对应其在原始sql中的位置。 |
| select_type   | 显示本行是简单或复杂select。如果查询有任何复杂的子查询，则最外层标记为`PRIMARY`(`DERIVED`、`UNION`、`UNION RESULT`) |
| table         | 查询引用的表名                                               |
| type          | 数据访问（读取）操作类型，标识查询性能的一个重要指标：       |
| possible_keys | 揭示哪一个索引可能有利于高效率查询                           |
| key           | mysql实际采用的查询索引                                      |
| ref           | 显示了之前的表在key列记录的索引中查找值所用的列或常量        |
| rows          | 为了找到符合条件的行所需要读取的总行数，是一个估计值，不精确。通过把所有rows列值相乘，可粗略估算整个查询需要检查的行数。 |
| extra         | 额外信息，如：`using index`、`filesort`等。                  |

这里的type需要重点关注一下，不同访问方式的实际查询性能不同，从好到坏依次是：system>const>eq_ref>ref>fulltext>ref_or_null>index_merge>unique_subquery>index_subquery>range>index>index>ALL

一般来说，需要保证查询至少达到`range`级别，最好能达到`ref`级别。

不同访问方式的说明如下。

| 类型   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| ALL    | 最坏的情况，全表扫描。                                       |
| index  | 和全表扫描一样。只是扫描表时按索引的次序进行，而不是按行的次序扫描。主要优点是避免了排序，但开销依然非常大。如在`Extra`列看到`Using index`，说明正在使用覆盖索引，只扫描索引的数据，它比按索引次序全表扫描的开销要小很多。 |
| range  | 范围扫描。一个有限制的索引扫描。`key`列显示使用了哪个索引。当使用`=`、`<>`、`>`、`<`、`>=`、`<=`、`IS NULL`、`BETWEEN AND `或`IN`操作符时，可以使用range方式。 |
| ref    | 一种索引访问的方式，它返回所有匹配某个单个值的行。此类索引访问只有当使用`非唯一索引`或`唯一索引非唯一前缀`时才会发生，这个类型与`eq_ref`不同的是：它用于关联操作时只使用索引的最左前缀，或索引不是`UNIQUE`和`PRIMARY KEY`。ref可以用于使用`=`或`<=>`操作符的带索引的列。 |
| eq_ref | 最多返回一条符合条件的记录，使用唯一性索引或主键查找时会发生，高效。 |
| const  | 当确定最多只有一行匹配时，mysql优化器会在查询前读取一次（只读取这一次），因此非常快。当主键放入where子句时，mysql把这个查询转化为一个const访问方式，高效。 |
| system | const类型的一种特例，表仅有一行满足条件。                    |
| Null   | mysql在优化阶段分解查询语句，在执行阶段甚至用不到访问表或索引，高效。 |

