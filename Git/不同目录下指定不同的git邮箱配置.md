# 在不同目录下指定不同的配置文件：   

[理想代码位置](#):  
- 学习用项目目录：`/improve/`
- 工作用项目目录：`/work/`

1. 修改'gitconfig'文件，在最下面添加如下代码：
```yml
[user]
    name = wcg
    email = wang.chenguang@china.com.cn

[includeIf "gitdir: ~/work/"]
    path = .gitconfig-work
[includeIf "gitdir:~/improve/"]
  path = .gitconfig-improve
```
2. 添加`gitconfig-work`文件和`gitconfig-improve`文件，分别添加对应的配置：
```yml
# work工作目录
[user]
    name = wcg
    email = wang.chenguang@china.com.cn

# improve学习目录
[user]
    name = wcg
    email = wyd@gmail.com
```
这样一来，在work目录就会使用工作的邮箱！