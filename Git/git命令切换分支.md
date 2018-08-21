# Git切换分支
```bash

#查看远程仓库所有分支
git branch -a

#查看本地分支
git branch

#切换到本地develop分支
git checkout develop

#切换到线上develop分支
git checkout -b develop origin/develop
```

`git head`实际是修改HEAD文件内容，让其指向不同的branch分支。  
`HEAD`文件指向的分支就是当前分支。 

一般来讲，HEAD的内容是指向`staging(暂存区)`的`master`分支的。  
```yml
ref: refs/heads/master
```
当然，HEAD也可以指向其他索引文件，不管怎样，这个索引文件的内容又由  
`git reset`控制。