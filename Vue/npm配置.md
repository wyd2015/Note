# npm配置
## 1. npm获取配置的6种方式，优先级由高到低
- 命令行后添加配置参数：`--proxy http://server:port`；
- 环境变量：以`npm_config_`为前缀的环境变量将会被认为是npm的配置属性。如果设置代理proxy，可添加这样的环境变量：`npm_config_proxy=http://server:port`；
- 用户配置文件：可通过`npm config get userconfig`查看配置文件路径。Mac系统默认路径是`$HOME/.npm`;
- 全局配置文件：可通过`npm config get globalconfig`查看文件路径，Mac默认路径是`/usr/local/etc/npmrc`；
- 内置配置文件：安装npm目录下的`npmrc`文件；
- 默认配置：npm本身有默认配置参数。如果以上5条均未设置，则npm会使用默认配置参数。

>在设置配置属性时属性值默认是被存储于用户配置文件中，如果加上`--global`，则被存储在全局配置文件中;  
如果要查看npm的所有配置属性（包括默认配置），可以使用`npm config ls -l`;   
如果要查看npm的各种配置的含义，可以使用`npm help config`。

## 2. 针对npm配置的命令行操作
```yml
npm config set <key> <value> [--global]
npm config get <key>
npm config delete <key>
npm config list
npm config edit
npm get <key>
npm set <key> <value> [--global]
```

## 3. 为npm设置代理
windows环境下，基于shadowsocket或shadowsocketR的网络代理，默认使用`127.0.0.1:1080`作为代理的ip和端口。
```yml
npm config set proxy http://127.0.0.1:1080
npm config set https-proxy http://127.0.0.1:1080
```
如果代理需要认证：
```yml
npm config set proxy http://username:password@127.0.0.1:1080
npm config set https-proxy http://username:pawword@127.0.0.1:1080
```

## 4. 取消代理设置
```yml
npm config delete proxy
npm config delete https-proxy
```