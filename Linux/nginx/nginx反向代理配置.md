---
title: 'nginx反向代理配置'
date: 2019-06-14 19:14:50
tag: nginx
categories: linux
---
# nginx作为反向代理时有个很容易掉进去的坑

先说下在项目部署时碰到的问题吧。

需求： wx系统里的一个模块需要调用wp系统的接口，在wp的服务端拦截器里对此接口进行了免登录过滤，以方便wx可以直接调用。
另外，生产环境对外只开放一个8080端口，由于多个系统同时使用，因此需要使用标识，如：/wp、/wx进行区分。

最初的时候，我对wp、wx的服务端、前端的配置如下：
```yml
server {
  listen: 8080;
  server_name: localhost;

  # wp前端
  location /wp {
    root /home/admin/web; # 如果下一行还有配置，当前行后的分号不可少
    index index.html index.htm;
    try_files $uri $uri/ /wp/index.html
  }

  # wx前端
  location /wx {
    root /home/admin/web;
    index index.html index.htm;
    try_files $uri $uri/ /wx/index.html
  }

  # wp 服务端
  location /wp/api {
    proxy_pass http://localhost:8086;
    proxy_connect_timeout 15s;
    proxy_send_timeout 15s;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # wx 服务端
  location /wx/api {
    proxy_pass http://localhost:8087;
    proxy_connect_timeout 15s;
    proxy_send_timeout 15s;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # wx 反向代理wp的接口
  location /wx/proxy {
    proxy_pass http://localhost:8086;
    proxy_connect_timeout 15s;
    proxy_send_timeout 15s;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  error_page 500 502 503 504 /50x.html
  location = /50x.html {
    root html;
  }
}
```
使用命令 `/opt/nginx/sbin/nginx -s reload` 重启nginx服务。


最初是这样配后，页面正常显示，但服务端接口无法正常达到服务端，前端报错：404。

后来在[NGINX服务器的反向代理PROXY_PASS配置方法讲解](https://www.cnblogs.com/lianxuan1768/p/8383804.html)一文中指出：  
```yml
location /wx/api {
  proxy_pass http://localhost:8087; # 问题在此处加不加 / 
  proxy_connect_timeout 15s;
  proxy_send_timeout 15s;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```
此处的proxy_pass，当在url后面使用了 `/`：`http://localhost:80807`，相当于是绝对路径，nginx不会吧location中匹配的路径部分代理走，
也即是最终的请求URL为：`http://localhost:80807/+参数` 不会再有 `/wx/api`；

如果proxy_pass 后不加 `/`，也即是：`http://localhost:80807`，则会把`/wx/api`也带到最终的请求url里：
``http://localhost:80807/wx/api+参数`。

最终的配置应该是：
```yml
# Nginx配置
server {
  listen: 8080;
  server_name: localhost;

  # wp前端
  location /wp {
    root /home/admin/web; # 如果下一行还有配置，当前行后的分号不可少
    index index.html index.htm;
    try_files $uri $uri/ /wp/index.html
  }

  # wx前端
  location /wx {
    root /home/admin/web;
    index index.html index.htm;
    try_files $uri $uri/ /wx/index.html
  }

  # wp 服务端
  location /wp/api {
    proxy_pass http://localhost:8086/;
    proxy_connect_timeout 15s;
    proxy_send_timeout 15s;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # wx 服务端
  location /wx/api {
    proxy_pass http://localhost:8087/;
    proxy_connect_timeout 15s;
    proxy_send_timeout 15s;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # wx 反向代理wp的接口
  location /wx/proxy {
    proxy_pass http://localhost:8086/;
    proxy_connect_timeout 15s;
    proxy_send_timeout 15s;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  error_page 500 502 503 504 /50x.html
  location = /50x.html {
    root html;
  }
}
```

此处对应apache的反向代理配置就没这么饶了，不用考虑/的问题
```yml
# Apache配置
Listen: 8080  # 监听8080端口

<VirtualHost *:8080> # 每个服务对应一个VirtualHost节点
  DocumentRoot /home/admin/web/wx/  # 此处配置前端静态资源位置
  ServerName localhost:8080 # 服务页面访问地址

  # 配置网站根目录，以及对根目录的访问控制
  <Directory "/home/admin/web/wx/">
    Options Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all

    <IfModule rewrite module>
      RewriteEngine On
      RewriteBase /
      RewriteRule ^index\.html$ - [L]
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteCond %{REQUEST_FILENAME} !-d
      RewriteRule . /index.html [L]
    </IfModule>
  </Directory>

  # wx 服务端配置
  proxyPass /wx/api http://localhost:8086
  proxyPassReverse /wx/api http://localhost:8086

  # wx 服务端反向代理wp接口配置
  proxyPass /wx/proxy http://localhost:8087
  proxyPassReverse /wx/proxy http://localhost:8087
</VirtualHost>
```