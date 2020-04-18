# 准备工作
- VPS服务器
- 网络可用

## 一、VPS服务器的购买
目前，Vultr的服务器最便宜的 $3.5/月足够使用，这也是目前性价比最高的了。

服务器部署出现ip后，可以在线检测ip地址的可用状态：[IP可用性检测工具](https://www.toolsdaquan.com/ipcheck/) 

![](img/ipCheck.png)

一定要确认可用后再往下进行！

## 二、SSR搭建
### 1. 使用SSH工具（Xshell、putty等）连接服务器；
### 2. 安装ssr
```yml
# 如果已经安装过wget，忽略此命令
yum install wget -y
# 安装ssr脚本
wget --no-check-certificate https://freed.ga/github/shadowsocksR.sh; bash shadowsocksR.sh
```
按提示输入连接密码（非服务器密码）和端口号即可。

### 3. 安装锐速
我用的是 `centOS7 x64`, 版本号：`3.10.0-229.1.2.el7.x86_64`，需要更换内核后再安装锐速。
```yml
# 更换内核，操作完成后系统会自动重启，ssh工具会自动重连
wget --no-check-certificate -O rskernel.sh https://raw.githubusercontent.com/hombo125/doubi/master/rskernel.sh && bash rskernel.sh

# 锐速安装，出现提示一直回车即可
yum install net-tools -y && wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && bash appex.sh install
```

如果是 `centOS6 x64` 直接执行下面的命令即可完成安装，无需更换内核
```yml
wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && bash appex.sh install '2.6.32-642.el6.x86_64'
```

SSR和锐速的服务默认开机自启。

重启ssr服务的命令：
```yml
/etc/init.d/shadowsocks restart
```