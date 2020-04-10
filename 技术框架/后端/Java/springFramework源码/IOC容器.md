# IOC容器

## 1. 配置元数据

有三种方式：

- 基于xml配置；

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
      
      <!--id用于标识bean的名称，class为bean实例的全限定包名-->    
      <bean id="..." class="...">  
          <!-- collaborators and configuration for this bean go here -->
      </bean>
  </beans>
  ```

- 基于注解的配置；

  ```java
  @Service
  public class PetStoreServiceImpl{}
  ```

- 基于Java的配置

  ```java
  @Configration
  public class MybatisConfig{
      private int connectTimeout = 1000;
  }
  ```

## 2. 实例化IOC容器

通过`ApplicationContext`的构造方法将指定路径下的配置元数据加载到IOC容器中，如

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("service.xml", "dao.xml")
```

其中，`service.xml`文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->
	<bean id="petStore" class="org.example.service.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
    </bean>
</beans>
```

`dao.xml`文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao" class="org.example.dao.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <bean id="itemDao" class="org.example.dao.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
</beans>
```

一个bean定义可以跨越多个xml文件起作用，每个单独的xml配置文件都代表体系结构中的逻辑层或模块，开发者可使用`ApplicationContext`（应用程序上下文）的构造函数从所有这些xml片段中加载xml中指定的`bean`，xml文件中使用`<import/>`即可实现：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

> 此处使用``<import/>``时，xml文件的路径不建议使用 `../ `这种相对路径的方式来引用`父级目录`中的文件，这样做会创建对当前应用程序之外的文件的依赖。特别不建议将该引用用于`classpath:URLs`的形式（如`classpath:../service.xml`）
>
> 这里也可以使用绝对路径，但最好使用`${...}`占位符来间接引用绝对路径。

## 使用IOC容器

`ApplicationContext`是一个高级工厂的接口，该工厂能够维护不同bean及其依赖关系的注册表。通过使用方法`T getBean(String name, Class<T> requiredType)`可以检索指定的bean。

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("service.xml", "dao.xml");
PetStoreService service = ctx.getBean("petStore", PetStoreService.class);
service.getUserList();
```

也可以使用`grovvy`配置来加载元数据配置，如下所示：

```xml
ApplicationContext ctx = new GenericGroovyApplicationContext("service.groovy", "dao.groovy");
```

最灵活的变体：`GenericApplicationContext`与t特定的文件`Reader`结合使用，如与xml文件的`XmlBeanDefinitionReader`结合使用如下所示：

```java
GenericApplicationContext gctx = new GenericApplicationContext();
new XmlBeanDefinitionReader(gctx).loadBeanDefinitions("service.xml", "dao.xml");
//new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
gctx.refresh();
```

