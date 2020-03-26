---
title: '前端对Long类型的值解析时出现精度损失的问题解决方案'
date: 2019-04-17 11:10:25
tag: springboot
---
项目中前端使用的是vue.js，在处理long类型的参数值时，会出现损失精度的情况，导致前端显示的long值与数据库中存储的值不一样，
实际业务中如果需要根据此long值进行一些更新、删除操作时，就会出现问题。

解决办法：对象在序列化成JSON时，将对象中属性类型为long的参数类型转为Stirng。

在springboot项目中，可以在该JavaBean对象的属性上添加 `@JsonSerialize` 注解，springboot默认使用Jackson类库。
```java
@JsonSerialize(using = ToStringSerializer.class)
private Long msgKeyIx;
```