# @Value使用要点
- @Value属性不能直接赋值给 `static` 的属性/方法上，否则取不到值；
- 使用@Value注解的类需要加上@Component注解，以便启动时将该类实例化到spring容器中。

### 如果需要为静态属性通过@Value属性赋值，应该经@Value注解加到该属性的set方法上。即：
```java
@Component
public class FtpFileUtil {
  
  private static String ftpHost;
  
  @Value("${ftp.server.host}")
  public void setFtpHost(String ftpHost) {
    FtpFileUtil.ftpHost = ftpHost;
  }
}
```