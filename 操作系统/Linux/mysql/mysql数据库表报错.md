# Table './vehicle/agreement' is marked as crashed and should be repaired 
## 今天项目异常，页面打开就报错： 
```yml
### Error querying database. Cause: java.sql.SQLException: Table './vehicle/agreement' is marked as crashed and should be repaired 
### The error may exist in class path resource [mapper/AgreementMapper.xml] 
### The error may involve com.onestep.vehicle.dao.AgreementMapper.getVehicleNo-Inline 
### The error occurred while setting parameters 
### SQL: SELECT ve.registration_no FROM agreement as ag LEFT JOIN vehicle as ve on ag.vehicle_id = ve.id LEFT JOIN customer as cu on ag.customer_id = cu.id where cu.id = ? and ag.status = '1' 
### Cause: java.sql.SQLException: Table './vehicle/agreement' is marked as crashed and should be repaired ; uncategorized SQLException for SQL []; SQL state [HY000]; error code [145]; Table './vehicle/agreement' is marked as crashed and should be repaired; nested exception is java.sql.SQLException: Table './vehicle/agreement' is marked as crashed and should be repaired
```
服务器上mysql.log日志异常：
```yml
[root@crm]# tail -f /var/log/mysqld.log
2018-11-12T13:37:15.954711Z 107 [Note] Access denied for user 'root'@'****' (using password: YES)
2018-11-12T14:18:07.179049Z 108 [ERROR] /usr/sbin/mysqld: Table './vehicle/agreement' is marked as crashed and should be repaired
```
进到MySQL查询agreement表也报这个错： 
```yml
mysql> select * from agreement limit 3 \G;
ERROR 145 (HY000): Table './vehicle/agreement' is marked as crashed and should be repaired
ERROR: No query specified
```

## 解决办法
```yml
$ mysqlcheck -uroot -p vehicle --auto-repair
Enter password:
vehicle.agreement
warning  : Table is marked as crashed
warning  : 1 client is using or hasn't closed the table properly
error    : Size of datafile is: 4096         Should be: 4104
error    : Corrupt
vehicle.agreement_copy                             OK
vehicle.agreement_history                          OK
vehicle.code                                       OK
vehicle.credit_card_info                           OK
vehicle.customer                                   OK
vehicle.file_info                                  OK
vehicle.insurance_info                             OK
vehicle.ownerShip                                  OK
vehicle.user                                       OK
vehicle.vehicle                                    OK
vehicle.vehicle_image                              OK

Repairing tables
vehicle.agreement
info     : Found block that points outside data file at 3968
status   : OK
```