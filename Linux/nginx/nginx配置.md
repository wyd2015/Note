---
title: 'Nginx作为代理服务器和文件服务器的配置'
Date: 2019-03-26 00:57:42
Tag: nginx
---
今天一个项目中碰到从nginx文件服务器下载文件时报404的错误，仔细分析了一下原来是nginx作为文件服务器的配置文件出了问题。最根本的还是没有理解nginx作为文件服务器配置时对文件路径的解析原理。

```yml
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

  access_log  /var/log/nginx/access.log  main;

  sendfile            on; # 作为文件服务器时需要开启
  tcp_nopush          on;
  tcp_nodelay         on;
  keepalive_timeout   65;
  types_hash_max_size 2048;

  include             /etc/nginx/mime.types;
  default_type        application/octet-stream;

  # Load modular configuration files from the /etc/nginx/conf.d directory.
  # See http://nginx.org/en/docs/ngx_core_module.html#include
  # for more information.
  include /etc/nginx/conf.d/*.conf; # 自定义的nginx配置文件

  # 这里是nginx同时为两个server提供服务，所以需要配两个server节点

  # 80端口的server
  server {
    listen  80; # 开放的端口号
    server_name  192.168.1.113; # 主机名，一般为ip地址

    # Load configuration files for the default server block.
    include /etc/nginx/conf.d/*.conf; # 不适用自定义配置可以不要

    # 在此配置前端页面的静态资源路径配置
    location / { 
      root /data/test/dist/;
    }

    # 图片服务器路径配置
    location /image/ {
      root  /data/test/files/cat/;
      autoindex on;
    }

    # nginx 404错误页面路径配置
    error_page 404 /404.html;
      location = /40x.html {
    }

    # nginx 服务器异常错误页面路径配置
    error_page 500 502 503 504 /50x.html;
      location = /50x.html {
    }
  }
  
  # 731端口的server
  server {
    listen  731;
    server_name  192.168.1.113; # 在同一个服务器上，所以这里与上面的server是一样的

    location / {
      root /home/hospital/web/dist/;
    }

    location /files/capsule/ {
        root   /home/hospital/;
        autoindex on;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
  }
}
```
这里要说明的是 nginx 作为文件服务器使用时的配置路径原理：  

```yml
# 文件存放的实际路径是 /home/hospital/files/capsule/
location /files/capsule/ {
  root /home/hospital/;
  autoindex on;
}
```
 1. 当上传文件时，会将 '/files/capsule/'拼接在文件名前：`/files/capsule/file.md` 保存入库；  

 2. 当下载文件时，直接将下载请求拼接成：`http://192.168.1.13:731/files/capsule/file.md`

 3. 当下载请求到达nginx时，nginx会对`/files/capsule/`进行解析，而解析的过程简单来说就是，  
 将`/files/capsule/`中`files`前的那个`/`替换成`root`所指向的路径`/home/hospital/`。
 ![](img/config.png)  
 这样，解析后的最终路径就变成了`/home/hospital/files/capsule/`，也即是实际的文件服务器位置。