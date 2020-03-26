---
title: '''Git新建工程'''
date: 2018-10-13 17:39:51
categories: Git
tags: git
---
# 使用Git新建工程
## 一、直接在本地初始化后推送到远程仓库
```yml
git init    #初始化
git status  #获取git状态
git add . # ./*代表全部添加
git commit -m "commit message"  #添加提交信息
git remote add origin git@github.com:wcg/GitTest.git    #添加远程仓库地址
git push -u origin develop #push到远程develop分支，同时设置默认跟踪分支
```

## 二、从远程Git仓库克隆到本地
```yml
#克隆到当前目录
git clone git://github.com/wcg/GitTest.git  

#克隆时指定存放位置
git clone git://github.com/wcg/GitTest.git  customDirectory
```