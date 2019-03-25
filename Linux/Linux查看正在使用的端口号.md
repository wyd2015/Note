---
title: 'Linux查看正在使用的端口号'
Date: 2019-03-26 01:18:25
Tag: nginx
---
netstat常用命令：  
```yml
netstat
-t: 显示TCP端口
-u: 显示UDP端口
-l: 仅显示监听套接字
-p: 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序
-n: 不进行DNS轮询，显示IP

# 查看当前所有tcp端口
netstat -ntlp

# 查看所有80端口使用情况
netstat -ntulp | grep 80

# 查看所有3306端口使用情况
netstat -an | grep 3306

# 查看一台服务器上有哪些服务和端口
netstat -lanp

# 查看一个服务有几个端口，以MySQL为例
ps -ef | grep mysql

# 查看某一端口的连接数
netstat -pnt | grep :3306|wc

# 查看某一端口的连接客户端IP
netstat -anp | grep 3306
```


```sh
[root@crm sbin]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:731             0.0.0.0:*               LISTEN      1062/nginx: master
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1062/nginx: master
tcp6       0      0 :::3306                 :::*                    LISTEN      933/mysqld
tcp6       0      0 :::8080                 :::*                    LISTEN      25066/java
tcp6       0      0 :::8081                 :::*                    LISTEN      26082/java
tcp6       0      0 :::22                   :::*                    LISTEN      748/sshd
```