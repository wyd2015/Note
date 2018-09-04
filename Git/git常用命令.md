# git常用命令
## 一、Git配置账号信息
```bash
#编辑配置文件
git config -e [--global]

#配置用户名
git config --global user.name wcg
#配置用户邮箱
git config --global user.email wang.chenguang@china.com.cn

#查看配置信息
git config --list

#获取帮助信息
git help config

#配置自动换行(提交到Git仓库时自动将换行符转换为 lf )
git config --global core.autocrlf input

#配置密钥
ssh-keygen -t rsa -C wang.chenguang@china.com.cn #生成密钥
ssh -T git@github.com #测试是否成功

#配置别名，git没有自动完成功能，别名这时候就有用了
# git st = git status
git config --global alias.st status 
# git cm = git commit
git config --global alias.cm commit
# git br = git branch
git config --global alias.br branch 
# git co = git checkout
git config --global alias.co checkout
```

## 二、pull
### 作用：取回远程仓库某个分支的更新，再与本地的指定分支合并。
pull操作只能拉取`origin`里的一个url地址（`fetch-url`），默认为添加到origin的第一个地址。
```yml
git pull origin master
git pull --all  #获取远程所有分支
git pull origin develop:master #获取origin主机的develop分支，并与本地的master分支合并

git pull origin develop #远程分支与当前分支合并
#此命令等同于一下两句：
git fetch origin
git merge origin/develop
```
在某些场合，Git会自动在本地分支和远程分支之间，建立一种追踪关系(tracking)。比如，在git clone时，所有本地分支默认和远程仓库的同名分支建立追踪关系，也即是本地的`master`分支自动追踪`origin/master`分支。  
也可以手动创建追踪关系：  
```yml
#指定master分支追踪origin/develop分支
git branch --set-upstream master origin/develop
```
如果当前分支与远程分支存在追踪关系，`git pull`就可以省略远程分支名称。
```yml
git pull origin
```

如果需要更改pull的来源，只需要更改config文件里url的顺序即可，以第一个url为准。