# IDEA编译spring源码
## 1. 安装、配置gradle
## 2. 下载spring源码
- github官方版，目前最新为5.2版
>https://github.com/spring-projects/spring-framework/release
- 4.2注释版
>https://github.com/wanwanpp/spring-framework-4.2.0
-
## 3. 配置SSR科学代理
为了加快源码编译时下载相关文件的速度，有SSR/SS的，可以添加代理。
这里可使用`Manual proxy configuration`，也可以使用`Auto-detect proxy settings`，配置参数按下图设置就行。

配置好可以点击下面的`Check connection`按钮，测试代理是与否可用。在弹出的对话框里的输入框输入'http://google.com'，提示成功就说明可以了。
![](img/proxy.png)
## 4. spring源码编译
```yml
# 准备工作
gradle cleaneclipse idea

# 编译spring-oxm
gradlew :spring-oxm:compileTestJava

# spring-core
gradle objenesisRepackJar
gradle cglibRepackJar
```