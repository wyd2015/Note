# `Centos7`管理防火墙

`centos7`防火墙默认处于关闭状态，需要先开启服务才能进行管理。另外`centos7`不同于`centos6`，`centos6`使用`iptables`管理防火墙，而`centos7`使用`firewall-cmd`进行管理。

## 1. 开启防火墙

```sh
# 开启服务
systemctl start firewalld.service
```

## 2. 防火墙常用操作

```sh
# 查看帮助
man firewall-cmd

# 关闭防火墙
systemctl stop firewalld.service

# 重启防火墙
systemctl restart firewalld.service

# 启/禁用开机自启
systemctl enable/disable firewalld.service

# 重载防火墙配置
firewall-cmd --reload

# 查看防火墙状态，正常情况下返回 running/FirewallD is not running 
firewall-cmd --state

# 列出支持的zone
firewall-cmd --get-zones

# 列出支持的服务，在列表中的服务是放行的服务
firewall-cmd --get-services

# 查看是否支持ftp服务，返回 yes/no
firewall-cmd --query-service ftp

# 临时开放ftp服务
firewall-cmd --add-service=ftp

# 永久开放ftp服务
firewall-cmd --add-service=ftp --permanent

# 永久移除ftp服务
firewall-cmd --remove-service=ftp --permanent

# 永久对外开放80端口
firewall-cmd --add-port=80/tcp --permanent

# 查看防火墙规则，同iptables命令
iptables -L -n
```

## 3. 开启指定端口

```sh
# 永久开放80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent

# 永久开放3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

**命令含义**  

- `--zone` 作用域；
- `--add-port=80/tcp`：添加端口，格式为：端口/通讯协议
- `--permanent`：永久生效，没有此参数重启后失效