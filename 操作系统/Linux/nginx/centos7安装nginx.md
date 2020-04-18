# centos7安装nginx

## 1. 安装相关组件

```sh
yum install -y openssl*
yum -y install ncurses-devel
yum -y install gcc-c++
yum -y install gcc gcc-c++ zlib zlib-devel openssl openssl-devel pcre pcre-devel
```

## 2. 编译安装nginx

```sh
#获取nginx。如果未安装wget，使用 yum -y install wget
wget https://nginx.org/download/nginx-1.17.1.tar.gz
  
#解压，zxvf显示解压文件
tar zxf nginx-1.17.1.tar.gz
  
#进入nginx源文件目录
cd nginx-1.17.1
  
#编译，默认https没有打开，需要添加 --with-http_ssl_module
./configure --with-http_ssl_module
  
#安装Nginx
make && make install
  
#启动Nginx
/usr/local/nginx/sbin/nginx
```

## 3. 防火墙配置

```sh
#查看防火墙状态
firewall-cmd --state
#关闭防火墙
systemctl stop firewalld.service
#打开防火墙
systemctl start firewalld.service
#永久开放80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
#更新防火墙规则
firewall-cmd --reload
```

## 4. 查看nginx启动效果页面

浏览器地址栏输入：`http://your-server-ip`看到下面的页面即为成功！  

![image-20200414204321432](img/image-20200414204321432.png)

## 5. 设置开机自启

1. 在`etc/init.d/`目录下创建nginx文件；

   ```sh
   vim /etc/init.d/nginx
   ```

   

2. nginx文件中添加以下内容，并保存；

   ```yaml
   # chkconfig: - 85 15
   # description: nginx is a World Wide Web server. It is used to serve
   nginx=”/usr/local/nginx/sbin/nginx” #修改成nginx执行程序的路径。
   NGINX_CONF_FILE=”/usr/local/nginx/conf/nginx.conf” #修改成nginx.conf文件的路径。
   ```

3. 给nginx文件添加权限；

   ```sh
   chmod a+x /etc/init.d/nginx
   ```

   到这一步，就可以使用下面的命令控制nginx服务的启动与停止：

   ```sh
   # 启动nginx
   /etc/init.d/nginx start
   # 停止nginx
   /etc/init.d/nginx stop
   ```

4. 使用`chkconfig`管理nginx服务；

   ```sh
   chkconfig --add /etc/init.d/nginx
   ```

   此时，可以使用`service`对`nginx`进行操作了！

   ```sh
   # 启动nginx
   service nginx start
   # 停止nginx
   service nginx stop
   # 重启nginx
   service nginx restart
   ```

5. 设置开机自启；

   ```sh
   chkconfig nginx on
   ```

   