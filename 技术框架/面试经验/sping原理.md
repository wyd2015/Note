# spring原理

可分三点进行说明：

- 结构
- 核心技术
- 优缺点

## 结构

spring是一个轻量级的`IOC`和`AOP`容器框架，是为Java应用程序提供基础服务的一套框架。主要用于简化应用程序的开发，使开发者只需要专注于业务。

spring常见的配置方式有三种：

- 基于`xml`的配置；
- 基于注解的配置：`spring2.5`引入，需要借助`xml`或`javaconfig`结束开启支持；
- 基于Java的配置：`spring3.0`开始支持。

spring主要由以下几个模块组成：

- **Spring Core**：核心类库，提供`IOC`服务；
- **Spring Context**：通过框架式的Bean访问方式，以及企业级功能（`JNDI`、Timer等）；
- **Spring AOP**：`AOP`服务；
- **Spring DAO**：对`JDBC`的抽象，简化了数据访问及其异常的处理；
- **Spring ORM**：对现有的`ORM`框架（`mybatis`、`hibernate`）的支持；
- **Spring Web**：提供了基本的面向Web的综合特性性，如多文件上传等；
- **Spring MVC**：提供面向Web应用的Model-View-Controller实现。

## 核心技术

### 1. IOC

**IOC**：控制反转，指创建对象的控制权的转移，以前有开发者自己根据需要手动创建对象，而IOC容器根据配置文件（对象类型）创建和管理各实例之间的依赖关系，对象与对象之间松散耦合，利于功能复用。

**DI**：依赖注入，与`IOC`是同一个概念的不同表述方式，即应用程序在运行时依赖`IOC`容器来动态注入对象所需要的外部资源。

#### 为什么要用IOC？

- 对象间的关系比较复杂，往往需要程序员手动维护它们之间的依赖关系；
- 解耦，由容器去维护具体的对象；
- 托管类的生产过程，如我们需要在类的生产过程中做一些处理，譬如代理，如果有容器来处理这部分过程，那应用程序就无需关心类是如何完成代理的。

#### IOC原理

使用Java反射机制，根据配置文件在运行时动态的创建、管理对象，并调用对象自己的方法。

#### IOC与DI关系

在Spring中，使用DI去实现IOC。也即是DI是IOC的一种实现方式，但不是唯一实现方式。

#### IOC的两种主要变体

这里是借用spring官方文档的说法：[DI exists in two major variants](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-metadata)

- 基于构造函数的依赖注入：通过容器调用具有多个参数（依赖项）的构造方法来完成；
- 基于Setter的依赖注入。

> `IOC`让相互协作的组件保持松耦合；  
>
> `AOP`允许把遍布与应用各层的功能分离出来形成可重用的功能组件。

