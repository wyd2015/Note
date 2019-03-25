---
title: 'Linux下查找nginx配置文件位置'
Date: 2019-03-26 00:48:25
Tag: nginx
---
服务器上配置的nginx由于很长时间没再修改过，今天需要再次进行配置时，一时想不起来nginx.conf的具体位置了。  
网上找到了一种方法:
```sh
# 查找nginx可执行文件的位置
[root@crm sbin]# whereis nginx
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz /usr/share/man/man3/nginx.3pm.gz

# 验证配置文件可用性，正常情况下会给出配置文件的位置
[root@crm sbin]# /usr/sbin/nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

# 修改配置文件后，重启nginx服务
[root@crm sbin]# cd /usr/sbin/
[root@crm sbin]# ./nginx -s reload
```