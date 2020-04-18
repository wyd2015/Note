# 问题
黑哥跟我说谷歌学术用不了，但谷歌搜索没问题。打开谷歌学术页，提示“we`re sorry ...”。谷歌搜了下发现问题和解决办法。

# 原因
服务器的ip端由于一些敏感操作导致ip被谷歌封禁。一般封的是IPV4，而IPV6的地址很少的被封（仅目前为止）。

# 解决办法
使用IPv6来访问[谷歌学术](https://scholar.google.com/)

## 1. 首先开启服务器IPV6的服务
这里我用的是vultr的vps，服务器为 $3.5/月，centOS7 x64的系统。

使用的是IPv4，而vultr默认是不给开启IPV6的，需要手动设置IPV6的开启状态。位置：  
`Server Information` -> `Settings` -> `IPv6`  
开启开关即可，不会额外收费！

## 2. 添加服务器配置
在`/etc/hosts`文件中添加谷歌学术的IPv6地址的DNS解析。
```yml
# 编辑 hosts文件
vim /etc/hosts
```
需要添加的内容：
```yml
## Scholar 谷歌学术
2404:6800:4008:c06::be scholar.google.com
2404:6800:4008:c06::be scholar.google.com.hk
2404:6800:4008:c06::be scholar.google.com.tw
2401:3800:4001:10::101f scholar.google.cn
2404:6800:4008:c06::be scholar.google.com.sg
2404:6800:4008:c06::be scholar.l.google.com
2404:6800:4008:803::2001 scholar.googleusercontent.com
```

## 重启SSR服务
如果用的逗比脚本或基于逗比脚本的修改版脚本，可以尝试使用一下命令重启：
```yml
/etc/init.d/shadowsocks restart
```
或者粗暴些，直接重启服务器
```yml
reboot
```
