---
title: git合并分支
date: 2019-06-06 09:32:27
tag: git
---
# 需求
现有分支：dev 和 master，需要将dev分支的代码合并到master分支。
# 操作
1. 由dev分支切换到master分支：
```yml
git checkout dev
```
2. 拉取master分支下的最新代码：
```yml
git pull
```
3. 如果拉完代码有冲突，先解决冲突并commit到暂存区，然后再执行合并的操作：
```yml
git merge dev
```
4. 如果存在代码冲突，先解决冲突，然后执行add和commit操作
```yml
git add .

# 如果不是使用git commit -m "备注"，git会自动将合并结果作为备注进行提交
git commit
```
5. 本地代码推到远程仓库
```yml
git push
```