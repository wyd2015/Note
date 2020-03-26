# IDEA为类和方法添加注释模版
## 1. 为类添加注释模版：
### File -> Setting -> Editor -> File and Code Templates -> Includes -> ActionScript File Header 
```java
/**
 * @Description: ${description}
 * @Created in: ${DATE} ${TIME}
 * @Author: Ethan
 * @Modified By: 
 */
```
![添加类模版](img/addComment_0.png)

## 2. 为方法添加注释模版：
1. 菜单: File -> Setting -> Editor -> Live Template  
![添加模板组](img/addComment_1.png)

2. 选中新添加的Method Comment组，点击 [+](#)，选择[1.Live Template](#)：  
![添加模版参数](img/addComment_2.png)  

3. 添加快捷键和模版：
![添加方法模版](img/addComment_3.png)  
```java
/**
 * @Description: $description$
 * @Create time: $date$ $time$
 * @Author: Ethan
 * @params: $params$
 * @return: $return$
 */
 ```

4. 设置参数信息：  
![添加模版](img/addComment_4.png)

5. 保存！