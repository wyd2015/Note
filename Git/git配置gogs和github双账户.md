---
title: 'git配置gogs和github双账户'
categories: Git
Date: 2019-03-22 10:22:15
---
以下操作均在win10 x64环境下进行。  
## 1. 如果配置了git全局账号，先取消此全局设置
```git
git config --global --unset user_name
git config --global --unset user_email1@gmail.com
```
## 2. SSH配置`id_rsa`私钥和`id_rsa.pub`公钥
在`C:\Users\Administrator（这里是win10系统用户名）\.ssh`目录下执行以下命令：  
```yml
# github账号
ssh-keygen -t rsa -C github@gmail.com
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Administrator/.ssh/id_rsa): id_rsa_github  #自定义
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_github.
Your public key has been saved in id_rsa_github.pub.
The key fingerprint is:
SHA256:2KqlFSO4+/iX3Nf3FwtRZ7WxbVKHmPexo9b7p7No github@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
|               o+|
|              o *|
|  . o + S    =.*.|
|  +oo.   .  .+E..|
+----[SHA256]-----+

# gogs账号
λ ssh-keygen -t rsa -C gogs@163.com
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Administrator/.ssh/id_rsa):   # 自定义，在此我直接回车，使用默认值
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\Administrator/.ssh/id_rsa.
Your public key has been saved in C:\Users\Administrator/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:3zB0DErcvW+7cNgBN5JXZtQjpZtHxGxZF+Hgz/8 gogs@163.com
The key's randomart image is:
+---[RSA 2048]----+
|        .o=..o===|
|      . o.=+Bo=++|
|             .o .|
|              ..E|
+----[SHA256]-----+
```
## 3. 添加公钥到对应gogs和github账户
gogs与github登录帐号后，左上角点击头像，选择用户设置项，进到设置页后选择
gogs——`SSH密钥`—>`增加密钥`  
github——`SSH and GPG keys`—>`New SSH key`  
密钥名称自定义；  
密钥内容将第二步生成的对应`id_rsa.pub`（对应gogs）和`id_rsa_github.pub`（对应github）里的内容粘贴进去，保存即可。
## 4. 配置账号信息
在`C:\Users\Administrator\.ssh`目录下新建`config`文件，没有后缀，然后用文本编辑器打开。  
```yml
# 配置 Gogs
Host 192.168.1.180 # 可自定义，一般跟域名一致
  HostName 192.168.1.180 # 域名，不可自定义
  IdentityFile C:\\Users\\Administrator\\.ssh\\id_rsa  # 私钥位置
  PreferredAuthentications publickey # 授权方式
  User wcg # gogs用户名
	
# 配置 Github
Host github.com
  HostName github.com
  IdentityFile C:\\Users\\Administrator\\.ssh\\id_rsa_github
  PreferredAuthentications publickey
  User wyd2015
```
【注意】这里的Host虽然可以自定义设置，但会影响正常操作比如在进行clone或链接测试的操作时就需要使用你设置的Host进行操作，如
```yml
# @符号后面的主机名必须是你定义的
git clone gogs@192.168.1.180:wcg/Note.git
git clone git@github.com:wyd2015/Note.git
```
## 5. 测试连接性
```yml
# 测试gogs
λ ssh -T gogs@192.168.1.180
The authenticity of host '192.168.1.180 (192.168.1.180)' can't be established.
RSA key fingerprint is SHA256:YbrCjSL87+bzlAf8v3apWPaBFUOA5thc4hKLN2+tVdY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.180' (RSA) to the list of known hosts.
Enter passphrase for key 'C:\\Users\\Administrator\\.ssh\\id_rsa':
Hi there, You've successfully authenticated, but Gogs does not provide shell access.
If this is unexpected, please log in with password and setup Gogs under another user.

# 测试github
ssh -T git@github.com
λ ssh -T git@github.com
The authenticity of host 'github.com (13.250.177.223)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of known hosts.
Enter passphrase for key 'C:\\Users\\Administrator\\.ssh\\id_rsa_github':
Hi wyd2015! You've successfully authenticated, but GitHub does not provide shell access.
```
