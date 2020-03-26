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

## 在使用@Value注解时，可以通过`:`指定默认值
```java
@Service
@Slf4j
public class UserService {
    @Autowired
    private SysUserDAO sysUserDAO;
    
    @Value("${login.password:}")//使用这种注解，如果未在配置文件中赋值，则默认将该值置为空字符串""
    private String password;

    @Value("${login.password}")//使用这种注解，如果未在配置文件中赋值，则默认讲该值置为Null
    private String password;
}
```
这里引用application.properties配置文件里配的用户默认密码
```yml
#自定义登录系统初始密码
login.password=123456
```