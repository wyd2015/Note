# 多数据源自动切换
## 一、单数据源整合步骤：  
1. 配置数据源DataSource的Bean;
2. 使用DataSource配置事务管理器；
3. 使用DataSource配置SqlSessionFactory的Bean；
4. 配置MapperScanConfigurer的Bean。  

其中，配置文件application.properties:  
```yml
#mybatis mapping文件位置配置
mybatis.mapper-locations=classpath:com/ethan/datasource/mapper/*.xml

######datasource######
###spring boot自动配置单数据源###
spring.datasource.url=jdbc:mysql://localhost:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&allowMultiQueries=true
spring.datasource.username=root
spring.datasource.password=111111
```

## 二、多数据源自动切换整合
多数据源自动切换实现原理：`切面编程`。  
`application.properties：`  
```yml
#mybatis mapping文件位置配置
mybatis.mapper-locations=classpath:com/ethan/datasource/mapper/*.xml

###手动配置多数据源###
#master
multiple.datasource.master.driver-class-name=com.mysql.jdbc.Driver
multiple.datasource.master.url=jdbc:mysql://localhost:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&allowMultiQueries=true
multiple.datasource.master.username=root
multiple.datasource.master.password=111111
#slave1
multiple.datasource.slave1.driver-class-name=com.mysql.jdbc.Driver
multiple.datasource.slave1.url=jdbc:mysql://localhost:3306/vehicle?useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&allowMultiQueries=true
multiple.datasource.slave1.username=root
multiple.datasource.slave1.password=111111
#slave2
multiple.datasource.slave2.driver-class-name=com.mysql.jdbc.Driver
multiple.datasource.slave2.url=jdbc:mysql://localhost:3306/shiro?useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&allowMultiQueries=true
multiple.datasource.slave2.username=root
multiple.datasource.slave2.password=111111
#other
multiple.datasource.other.driver-class-name=com.mysql.jdbc.Driver
multiple.datasource.other.url=jdbc:mysql://localhost:3306/mysql?useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&allowMultiQueries=true
multiple.datasource.other.username=root
multiple.datasource.other.password=111111
```
`切面类：`
```java
package com.ethan.datasource.common;

import com.ethan.datasource.annotation.TargetDataSource;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

@Slf4j
@Aspect
@Order(-1)
@Component
public class DynamicDataSourceAspect {

    @Pointcut("execution(* com.ethan.datasource.service.*.get*(..))")
    public void pointCut(){}

    /**
     * 方法执行之前更换数据源
     * @param joinPoint
     * @param targetDataSource
     */
    @Before("@annotation(targetDataSource)")
    public void doBefore(JoinPoint joinPoint, TargetDataSource targetDataSource){
        DataSourceKey dataSourceKey = targetDataSource.dataSourceKey();
        DynamicDataSourceContextHolder.set(dataSourceKey);

        if(dataSourceKey == DataSourceKey.DB_OTHER){
            log.info("设置当前数据源为： " + DataSourceKey.DB_OTHER);
        }else{
            log.info("使用默认数据源： " + DataSourceKey.DB_MASTER);
        }
    }

    /**
     * 执行方法后清除数据源
     * @param joinPoint
     * @param targetDataSource
     */
    @After("@annotation(targetDataSource)")
    public void doAfter(JoinPoint joinPoint, TargetDataSource targetDataSource){
        log.info("清除前的数据源： " + targetDataSource.dataSourceKey());
        DynamicDataSourceContextHolder.clear();
        log.info("清除后的数据源： " + DynamicDataSourceContextHolder.get());
    }

    /**
     * 使用从库执行方法前
     * @param joinPoint
     */
    @Before(value = "pointCut()")
    public void doBeforeWithSlave(JoinPoint joinPoint){
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        //获取当前切点方法对象
        Method method = methodSignature.getMethod();
        if(method.getDeclaringClass().isInterface()){
            try {
                method = joinPoint.getTarget().getClass()
                        .getDeclaredMethod(joinPoint.getSignature().getName(), method.getParameterTypes());
            } catch (NoSuchMethodException e) {
                log.error("方法不存在！", e);
            }
        }

        if(method.getAnnotation(TargetDataSource.class) == null){
            DynamicDataSourceContextHolder.setSlave();
        }
    }
}

```
`多数据源配置类：`  
```java
package com.ethan.datasource.config;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import com.ethan.datasource.common.DataSourceKey;
import com.ethan.datasource.common.DynamicRoutingDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class DynamicDataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "multiple.datasource.master")
    public DataSource dbMaster(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "multiple.datasource.slave1")
    public DataSource dbSlave1(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "multiple.datasource.slave2")
    public DataSource dbSlave2(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "multiple.datasource.other")
    public DataSource dbOther(){
        return DruidDataSourceBuilder.create().build();
    }

    /**
     * 动态数据源核心
     * @return 数据源实例
     */
    @Bean
    public DataSource dynamicDataSource(){
        DynamicRoutingDataSource ds = new DynamicRoutingDataSource();
        ds.setDefaultTargetDataSource(dbMaster());

        Map<Object, Object> dsMap = new HashMap<>(4);
        dsMap.put(DataSourceKey.DB_MASTER, dbMaster());
        dsMap.put(DataSourceKey.DB_SLAVE1, dbSlave1());
        dsMap.put(DataSourceKey.DB_SLAVE2, dbSlave2());
        dsMap.put(DataSourceKey.DB_OTHER, dbOther());
        ds.setTargetDataSources(dsMap);

        return ds;
    }

    /**
     * sqlSession会话工厂
     * @return
     * @throws Exception
     */
    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dynamicDataSource());
        //此处设置为了解决找不到mapper文件的问题
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:/com/ethan/datasource/mapper/*.xml"));
        return sqlSessionFactoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory());
    }

    /**
     * 事务管理
     * @return 事务管理实例
     */
    @Bean
    public PlatformTransactionManager platformTransactionManager(){
        return new DataSourceTransactionManager(dynamicDataSource());
    }
}
```

demo地址：[springboot2.0配置多数据源自动切换]()