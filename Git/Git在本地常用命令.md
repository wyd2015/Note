---
title: '''Git本地常用命令'''
date: 2018-10-12 17:39:51
categories: Git
tags: git
---
# Git在本地常用命令
## add
```yml
git add *   #跟踪新文件
git add -u [path]   #添加[指定路径]下已跟踪的文件
```

## rm
```yml
git rm -f * #强制移除文件
git rm -f --cached *    #停止追踪指定文件，但该文件会保留在工作区
git mv file_from file_to    #重命名跟踪文件

git log #查看提交记录
```

## commit 提交
```yml
git commit  #提交更新
git commit -m "message" #带有更新说明的提交
git commit -a  # 跳过使用暂存区，把所有已经跟踪的文件暂存起来一并提交
git commit --amend  #修改最后一次提交
git commit log  #查看所有提交，包括没有push的commit
git commit -m "#133"    #关联issue[133]
git commit -m "fix #133" #commit并关闭isuue[133]
git commit -m "概要描述'$'\n\n''1.详细描述'$'\n''2.详细描述"    #提交概要描述和详细描述
```

## reset 重置
```yml
git reset HEAD *  #取消已暂存的文件
git reset --mixed HEAD *  #取消已暂存的文件
git reset --soft HEAD *   #重置到指定状态，不会修改索引区和工作树
git reset --hard HEAD *   #重置到指定状态，会修改索引区和工作树
git reset --files *     #重置index区文件
```

## revert 撤销
```yml
git revert HEAD     #撤销前一次操作
git revert HEAD~    #撤销前前一次操作
git revert commit   #撤销指定操作
```

## checkout
```yml
git checkout --file     #取消对文件的修改
git checkout branch|tab|commit -- file_name     #从仓库取出file覆盖到当前分支
git checkout HEAD~1 [文件]    #将会更新working directory取匹配某次commit
git checkout -- .   #从暂存区取出文件覆盖工作区
git checkout -b branch_new 0c3d45     #表示从当前分支commit哈希值为0c3d45的节点，分一个新的分支branch_new出来，并切换到branch_new分支。
```

## diff 对比
```yml
git diff            #比较worktree和index间的差异
git diff files      #查看指定文件的差异
git diff --stat     #查看简单的diff结果
git diff --cached   #比较index和HEAD之间的差异
git diff HEAD       #比较worktree和HEAD之间的差异
git diff branch     #比较worktree和branch之间的差异
git diff branch1 branch2    #比较两个分支间的差异
git diff commit 57b52f commit 2d5aba  #比较两次提交间的差异
git diff master...test   #master与test分支的共有父分支与test分支之间的差异比较
```

## stash 暂存区
```yml
git stash   #将工作区现场（已跟踪文件）储存起来，等以后恢复了再继续工作
git stash list  #查看保存的工作现场
git stash apply     #恢复工作现场
git stash drop  #删除stash内容
git stash pop   #恢复的同时直接删除stash内容
git stash apply stash@{0}   #当你保存了不止一份工作现场时，用于恢复指定的工作现场
```

## merge 合并
```yml
git merge --squash test #合并压缩，将test上的commit压缩为一条
```

## cherry-pick 挑选合并
```yml
git cheery-pick commit  #挑选合并，将commit合并到当前分支
git cheery-pick -n commit  #挑选多个提交，合并完后可以继续挑选下一个提交
```

## rebase 重置基准
```yml
git rebase master   #将master分支上超前的提交变基到当前分支
git rebase --onto master 169a3 #限制回滚范围，rebase当前分支169a3以后的提交
git rebase --interactive    #交互模式，修改commit
git rebase --continue   #处理完冲突继续合并
git rebase --skip   #跳过
git rebase --abort  #取消合并
```

## branch 分支
```yml
git branch   #列出本地分支
git branch -r   #列出远程分支
git branch -a   #列出所有分支

git branch -v   #查看各分支最后一个提交对象的信息

git branch --merge  #查看已经合并到当前分支的分支
git branch --no-merge   #查看未合并到当前分支的分支

git branch test  #新建test分支
git branch branch[branch|commit|tag]    #从指定位置新建分支
git branch --track branch remote-branch     #新建一个分支，与指定的远程分支建立追踪关系

git branch -m oldName newName   #重命名oldName分支为newName

git push origin:branchName  #删除远程分支branchName
git push origin --delete newBranch  #删除远程分支newBranch

git branch -d test  #删除本地test分支
git branch -D test  #强制删除本地test分支

git branch --set-upstream dev origin/dev    #将本地dev分支与远程dev分支间建立连接
```
