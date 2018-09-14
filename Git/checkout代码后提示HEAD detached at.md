# checkout代码后进行pull操作，提示HEAD detached at

今天切换项目分支，把代码切换到一个tj分支，checkout后没注意，等把手里的问题解决,提交代码到本地仓库，  
进行pull操作时，代码拉取失败，才发现项目名称后的分支提示信息是：  
 `HEAD detached at 5a2d8fa -> origin`。  

网上查了一下，HEAD detached at ******(提交的记录号)，表示当前分支处于游离状态，不属于已有的任何分支，所以要提交的代码不知道往哪里提，就会给出这样一个提示。  

**[解决思路](#)**:  
1. 基于本次提交创建一个临时分支`temp`；
2. 切换到实际分支；
3. 将`temp`分支merge到实际分支；
4. 删除临时分支`temp`。
  
具体操作命令如下：  
```bash
$ git branch temp fef4501
$ git checkout test
$ git merge temp
$ git branch -d temp
```