# SpringBoot2.0整合jsp
## 注意点：  
1. 只能打包成`war`类型的格式；
2. 必须将打好的`war`包放到外部tomcat容器中运行。

步骤：  
1. 引入外部tomcat依赖：
```xml
<!-- SpringBoot web 核心组件 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
</dependency>

<!-- SpringBoot 外部tomcat支持 -->	
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```
2. 在application.properties配置文件中配置jsp文件路径和后缀：
```yaml
spring.mvc.view.prefix=/WEB-INF/jsp/     #视图（jsp）文件路径
spring.mvc.view.suffix=.jsp              #视图（jsp）文件后缀
```

3. 在项目中新建视图文件存储路径：  
这里一定不能把JSP页面放到 `resources/jsp` 路径下，否则项目运行后访问不到此路径。  
正确的存放位置应该是 `src/main/webapp/WEB-INF/jsp/*.jsp`

4. 注意事项：  
使用IDEA生成SpringBoot2.0 的整合jsp项目时，会在`xxxApplication.java`统计目录下
自动生成一个`ServletInitializer.java`文件，此文件不可删，因为springboot整合jsp，
需要使用外部的http服务器（Tomcat/Jetty）来部署打包好的`war`包，
此`ServletInitializer.java`文件就是为了初始化该外部http服务器用的。


# 全局异常捕获
**[原理]()**：AOP面向切面编程

```java
/**
 * @ControllerAdvice Controller的辅助类，在此作为全局异常处理的切面类。异常出现时发送邮件也是在此处实现。
 * @ControllerAdvice 指定扫描范围(basePackage="com.ethan.jsp.controller")，一般扫描controller位置。
 * @ControllerAdvice 约定的几种返回方式：
 *      1. 返回String类型，表示跳到某个view；
 *      2. 返回ModelAndView
 *      3. 返回Model+@ResponseBody
 */
//@ControllerAdvice
//@ResponseBody
@RestControllerAdvice(basePackages = {"com.ethan.jsp.controller"}) //必需,相当于 @ControllerAdvice + @ResponseBody，用于以json形式返回捕获到的异常
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)//必需,用于设定捕获的异常类型
    public Map<String, Object> exceptionHandler(){

        Map<String, Object> map = new HashMap<>();
        map.put("status", 104);
        map.put("message", "系统异常！");

        return map;
    }

}
```

