# MySQL5.7重置root账号密码
### 1. 修改MySQL配置文件，设置免密码登录
```sh
[root@localhost ~]# vim /etc/my.cnf

-----------------------------------------
[mysqld] # 在此标签下添加
skip-grant-tables # 添加此句，设置免密码登录
-----------------------------------------
```
### 2. 重启MySQL服务
```sh
[root@localhost ~]# service mysqld restart
```
### 3. 登录MySQL
```sh
# 提示输入密码时直接回车即可
oot@localhost ~]# mysql -uroot -p
Enter password: 
```
### 4.设置新密码
```sql
-- 重置密码
update user set authentication_string=password('123456') where user='root';

-- 刷新权限
flush privileges;
```

### 5. 修改配置文件，注释掉免密登录的语句
```sh
[root@localhost ~]# vim /etc/my.cnf

----------------------------------
[mysqld]
#skip-grant-tables 
---------------------------------
```
### 6. 重启MySQL服务
```sh
[root@localhost ~]# service mysqld restart
```
### 7. 使用新密码登录即可
```sh
[root@localhost ~]# mysql -uroot -p123456
```