# pringboot调用MySQL存储过程出错记录

## 错误的大致描述：
1. 使用`root`账户`创建`存储过程；
2. 程序登录MySQL使用的是`trsorg`账户，该用户没有权限执行存储过程，因此在创建存储过程时，使用`root`账户去`定义`存储过程；
```sql
-- 使用root账户定义，有权限执行存储过程
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_generate_cfg_table`(IN `id` int(11))

-- 使用trsorg账户定义，没有权限执行存储过程
CREATE DEFINER=`trsorg`@`localhost` PROCEDURE `proc_generate_cfg_table`(IN `id` int(11))
```
3. 无论是直接在112服务器（`MySQL -utrsorg -p`）上登录，还是远程登录（`MySQL -utrsorg -h *.*.*.112 -p`）112服务器，使用`trsorg`账户登录MySQL服务器（112），直接调用存储过程`call proc_generate_cfg_table(1)`建表都没有问题；
4. 在程序实际调用该存储过程时，却无法成功新建对应的表，log记录的报错信息如下：
```yml
18-12-14 10:44:02.020 [http-nio-8001-exec-16] ERROR com.trs.yq.service.ProcedureService-75 -生成指标ID为[3]的分表时发生异常：
org.springframework.dao.TransientDataAccessResourceException: 
### Error querying database.  Cause: java.sql.SQLException: User does not have access to metadata required to determine stored procedure parameter types. If rights can not be granted, configure connection with "noAccessToProcedureBodies
=true" to have driver generate parameters that represent INOUT strings irregardless of actual parameter types.
### The error may exist in URL [jar:file:/home/trs/yq_org_server/yq_server-2.1.0.jar!/BOOT-INF/lib/yq_common-2.1.0.jar!/com/trs/yq/mapper/sqlmap-procedureDAO.xml]
### The error may involve com.trs.yq.dao.ProcedureDAO.generateTable
### The error occurred while executing a query
### SQL: call proc_generate_cfg_table(?)
### Cause: java.sql.SQLException: User does not have access to metadata required to determine stored procedure parameter types. If rights can not be granted, configure connection with "noAccessToProcedureBodies=true" to have driver gene
rate parameters that represent INOUT strings irregardless of actual parameter types.
; SQL []; User does not have access to metadata required to determine stored procedure parameter types. If rights can not be granted, configure connection with "noAccessToProcedureBodies=true" to have driver generate parameters that rep
resent INOUT strings irregardless of actual parameter types.; nested exception is java.sql.SQLException: User does not have access to metadata required to determine stored procedure parameter types. If rights can not be granted, configu
re connection with "noAccessToProcedureBodies=true" to have driver generate parameters that represent INOUT strings irregardless of actual parameter types.
        at org.springframework.jdbc.support.SQLStateSQLExceptionTranslator.doTranslate(SQLStateSQLExceptionTranslator.java:108)
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:73)
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:81)
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:81)
        at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:73)
        at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:446)
        at com.sun.proxy.$Proxy67.selectOne(Unknown Source)
        at org.mybatis.spring.SqlSessionTemplate.selectOne(SqlSessionTemplate.java:166)
        at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:82)
        at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:59)
        at com.sun.proxy.$Proxy161.generateTable(Unknown Source)
        at com.trs.yq.service.ProcedureService.generateTable(ProcedureService.java:40)
        at com.trs.yq.service.ProcedureService.generateTableByCfgGroupId(ProcedureService.java:73)
        at com.trs.yq.server.service.config.ConfigAlertService.createNewConfigInfo(ConfigAlertService.java:2074)
        at com.trs.yq.server.controller.monitor.MonitorConfigController.createNewConfig(MonitorConfigController.java:303)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:483)
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:133)
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:116)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
```

**错误出现的原因**：
>低权限用户访问了高权限用户创建的存储过程。

**解决办法有三种**：
>1. 在springboot配置文件中，在MySQL连接的URL后添加  
`&noAccessToProcedureBodies=true`;
>2. 授予相关连接账户权限：  
`grant select on mysql.proc to 'trsorg'@'localhost'`;
>3. 使用命令将定义者改为对应的低权限用户：  
`update mysql.proc set DEFINER='trsorg' WHERE NAME='proc_generate_cfg_table' AND db='yq_organ'`;
