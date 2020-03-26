---
title: '使用Cloudflare+WebSocket+TLS+Nginx搭建V2ray拯救被封国外VPS 脚本已可用'
date: 2020-02-24
categories: VPN
---
# 使用Cloudflare+WebSocket+TLS+Nginx搭建V2ray拯救被封国外VPS 脚本已可用
1. 购买一个域名，或自己申请免费的域名；
2. 注册cloudflare，并添加域名，确保域名可用；
3. 进入域名管理页，点击`Add a Site`按钮；
4. 在cloudflare的DNS设置界面，添加DNS解析记录；
> type：A，name：www，value：*.*.*.*(ipv4地址)，TTL：Automatic，status：橘黄色
> type：A，name：*。xyz(自己购买的二级域名)，value：*.*.*.*(ipv4地址，同上)，TTL：Automatic，status：橘黄色
5. 在域名注册商的管理界面修改域名使用的 `nameservers`，修改为CloudFlare提供的两个`nameservers`；
6. 一键安装`v2ray`
```bash
# 安装curl
yum update -y && yum install curl -y

# 一键安装 Vmess+websocket+TLS+Nginx+Website
bash <(curl -L -s https://raw.githubusercontent.com/wulabing/V2Ray_ws-tls_bash_onekey/master/install.sh) | tee v2ray_ins.log

# 启动v2ray
systemctl start v2ray
# 启动nginx
systemctl start nginx

# 检查服务运行状态
systemctl status v2ray
systemctl status nginx
```