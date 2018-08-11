# lombok+logback+slf4j 简化log日志输出
## 一直以来对logback+slf4j+lomback 简化日志输出的具体配置搞不清楚，试了几次老是出问题（\<scope>的问题），今天总算摸到了门路，总结如下：  

1. **引入相关依赖**：lombok、slf4j-api、logback-classic三个依赖；
    ```xml
    <!-- lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.2</version>
    </dependency>

    <!-- slf4j-api -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
    </dependency>

    <!-- logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
    ```
    >**注意**：这里的\<dependency>如果是从[Maven Repository](http://mvnrepository.com/artifact/org.projectlombok) 直接拷贝下来的，需要手动删除\<scope>xxx\</scope>标签！  
    否则，使用@Slf4j注解后，项目无法正常启动。
2. 准备**logback.xml**，此文件是logback的默认配置文件，主要是对日志的输出位置、日志文件的命名规则、保存时间以及日志输出的级别设定等，该文件放置在resources目录下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<!-- encoder 默认配置为PatternLayoutEncoder -->
		<encoder>
			<pattern>%d{yy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36}-%L -%msg%n</pattern>
		</encoder>
	</appender>

    <appender name="err" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/error.log</file>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--每日备份日志存储格式和位置 -->
            <fileNamePattern>log/backup/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--最大保存天数 -->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP"> 
            <MaxFileSize>20MB</MaxFileSize> </TimeBasedFileNamingAndTriggeringPolicy> -->
        <encoder>
            <pattern>%d{yy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36}-%L -%msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="warn" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/warn.log</file>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/backup/warn.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36}-%L -%msg%n</pattern>
        </encoder>
    </appender>

	<appender name="suc" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>log/success.log</file>
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>INFO</level>
			<onMatch>ACCEPT</onMatch>
			<onMismatch>DENY</onMismatch>
		</filter>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>log/backup/success.%d{yyyy-MM-dd}.log</fileNamePattern>
			<maxHistory>15</maxHistory>
		</rollingPolicy>
		<!-- <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP"> 
			<MaxFileSize>20MB</MaxFileSize> </TimeBasedFileNamingAndTriggeringPolicy> -->
		<encoder>
			<pattern>%d{yy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36}-%L -%msg%n</pattern>
		</encoder>
	</appender>
    
	<appender name="bug" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>log/debug.log</file>
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>DEBUG</level>
		</filter>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>log/backup/debug.%d{yyyy-MM-dd}.log</fileNamePattern>
			<maxHistory>15</maxHistory>
		</rollingPolicy>
		<!-- <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP"> 
			<MaxFileSize>20MB</MaxFileSize> </TimeBasedFileNamingAndTriggeringPolicy> -->
		<encoder>
			<pattern>%d{yy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36}-%L -%msg%n</pattern>
		</encoder>
	</appender>

	<root level="info">
		<!-- <level value="error" /> <level value="info" /> -->
		<appender-ref ref="suc" />
		<appender-ref ref="err" />
        <appender-ref ref="warn" />
		<appender-ref ref="bug" />
		<appender-ref ref="STDOUT" />
	</root>
	
	<!-- mybatis log configure -->
	<logger name="com.ethan.shiro.dao" level="DEBUG"/>
</configuration>
```

以上是配置，使用方法就是在需要输出日志的类上添加注解**@Slf4j**，  
然后在需要打印日志的位置使用log.info() 即可。
```java
@Slf4j
@SpringBootApplication
public class ShiroApplication {
	public static void main(String[] args) {
		SpringApplication.run(ShiroApplication.class, args);
		log.info("Shiro Application start sucessfully ...");
	}
}
```
如果是打印异常信息则是在catch块里使用：
```java
try{
    int i = 1/0;
}catch(Exception e){
    log.error("程序发生异常了：", e);
}
```