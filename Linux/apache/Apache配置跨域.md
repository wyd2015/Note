---
title: 'Apache配置跨域'
date: 2019-04-25 22:21:58
tag: 'Linux'
---
一、相关概念：
1. 同源：URL由协议、域名、端口和路径组成，如果两个URL的协议、域名、端口相同，则表示它们同源；

2. 同源策略：浏览器的同源策略，限制了来自不同源的"document"或脚本，对当前"document"读取或设置某些属性。 从一个域上加载的脚本不允许访问另外一个域的文档属性。

二、针对Apache代理设置跨域访问：  
1. 找到Apache配置文件：`httpd.conf`，一般在 `/usr/local/httpd/`目录下；
2. 编辑httpd.conf文件：`vi httpd.conf`；
3. 找到被注释的 `#LoadModule headers_module modules/mod_headers.so`，取消注释；
```yml
# 开启apache请求头信息的自定义模块
LoadModule headers_module modules/mod_headers.so
```
4. 在独立`Virtual Host`节点下添加一行配置：
```yml
Header set Access-Control-Allow-Origin *
```
5. 保存更改；
6. 重启Apache服务。