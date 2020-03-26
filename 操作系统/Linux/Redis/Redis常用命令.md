# Redis常用命令
```sh
# 批量删除规则键, linux环境执行（不用登录redis）
## -c 表示集群模式
## xargs部分加 -t，表示打印语句
## 去除 > /dev/null 输出执行结果 
redis-cli -c -h 192.168.1.176 --scan --pattern test_db_* | xargs -t redis-cli -c -h 192.168.1.176 del > /dev/null

# 列出所有的键
keys *

# 列出所有以abs_key_开头的键
keys "abs_key_*"

# 删除所有key
flushdb

# 删除所有key
flushall


```