# Maven引入mybatis依赖时启动项目报错无效的@MapperScan注解
## 报错情况下的配置如下
**1. mybatis配置依赖项**
```xml
<!-- Mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
```

**2. 启动项注解**：
```java
import lombok.extern.slf4j.Slf4j;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import uk.org.lidalia.sysoutslf4j.context.SysOutOverSLF4J;

@Slf4j
@SpringBootApplication
@MapperScan("com.ethan.shiro.dao")
public class ShiroApplication {
	public static void main(String[] args) {
        
                //system.out/error.print()这一类的日志输出转化为log日志进行输出
		SysOutOverSLF4J.sendSystemOutAndErrToSLF4J();

		SpringApplication.run(ShiroApplication.class, args);
		log.info("Shiro Application start sucessfully ...");
	}
}
```

**报错**：
```java
18-08-09 15:07:26.884 [restartedMain] INFO  o.s.core.annotation.AnnotationUtils-1919 -Failed to introspect annotations on class com.ethan.shiro.ShiroApplication: java.lang.annotation.AnnotationFormatError: Invalid default: public abstract java.lang.Class org.mybatis.spring.annotation.MapperScan.factoryBean()
18-08-09 15:07:27.404 [restartedMain] ERROR o.s.boot.SpringApplication-845 -Application run failed
java.lang.annotation.AnnotationFormatError: Invalid default: public abstract java.lang.Class org.mybatis.spring.annotation.MapperScan.factoryBean()
	at java.lang.reflect.Method.getDefaultValue(Method.java:612)
	at sun.reflect.annotation.AnnotationType.<init>(AnnotationType.java:132)
	at sun.reflect.annotation.AnnotationType.getInstance(AnnotationType.java:85)
	at sun.reflect.annotation.AnnotationParser.parseAnnotation2(AnnotationParser.java:266)
	at sun.reflect.annotation.AnnotationParser.parseAnnotations2(AnnotationParser.java:120)
	at sun.reflect.annotation.AnnotationParser.parseAnnotations(AnnotationParser.java:72)
	at java.lang.Class.createAnnotationData(Class.java:3521)
	at java.lang.Class.annotationData(Class.java:3510)
	at java.lang.Class.getAnnotations(Class.java:3446)
	at org.springframework.core.type.StandardAnnotationMetadata.<init>(StandardAnnotationMetadata.java:70)
	at org.springframework.beans.factory.annotation.AnnotatedGenericBeanDefinition.<init>(AnnotatedGenericBeanDefinition.java:58)
	at org.springframework.context.annotation.AnnotatedBeanDefinitionReader.doRegisterBean(AnnotatedBeanDefinitionReader.java:216)
	at org.springframework.context.annotation.AnnotatedBeanDefinitionReader.registerBean(AnnotatedBeanDefinitionReader.java:145)
	at org.springframework.context.annotation.AnnotatedBeanDefinitionReader.register(AnnotatedBeanDefinitionReader.java:135)
	at org.springframework.boot.BeanDefinitionLoader.load(BeanDefinitionLoader.java:158)
	at org.springframework.boot.BeanDefinitionLoader.load(BeanDefinitionLoader.java:135)
	at org.springframework.boot.BeanDefinitionLoader.load(BeanDefinitionLoader.java:127)
	at org.springframework.boot.SpringApplication.load(SpringApplication.java:704)
	at org.springframework.boot.SpringApplication.prepareContext(SpringApplication.java:393)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:328)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1258)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1246)
	at com.ethan.shiro.ShiroApplication.main(ShiroApplication.java:17)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.boot.devtools.restart.RestartLauncher.run(RestartLauncher.java:49)

Process finished with exit code 0
```

[**问题出现的原因**](#)  
在引入mybatis依赖时引错了依赖项，应该直接引入[mybatis-spring-boot-starter](http://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/1.3.2) 依赖，不宜单独引入mybatis依赖和mybatis-spring依赖。

这里有个疑问`为什么直接用mybatis-spring-boot-starter`依赖后可以正常启动，但使用`mybatis + mybatis-spring`后无法正常启动？  
如果只需要简单的使用mybatis进行CRUD操作，直接使用`mybatis-spring-boot-starte`没问题，不需要增加其他配置。  
如果需要引入数据库连接池，就需要  



[**解决办法**](#)  
只需要引入一个[mybatis-spring-boot-starter](http://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/1.3.2)依赖项即可：
```xml
<!-- Mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```

