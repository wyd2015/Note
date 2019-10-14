# IDEA中为spring源码创建一个gradle测试模块
## 1. 按图中步骤创建一个子模块；
![](img/createSubModule.png)

在spring源码根目录的`setting.gradle`文件末尾已自动添加刚才创建的子模块名称  

![](img/autoAdd.png)

## 2. 修改配置
刚才创建的`spring-trs-test`模块中，gradle配置文件名称默认是`build.gradle`，这里可以改成spring中定义好的格式`spring-trs-test.gradle`。

参考资料：[Spring源码阅读之在spring源码中创建一个gradle测试模块](https://blog.csdn.net/u010999809/article/details/94293328)