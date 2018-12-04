今天在处理一个get请求时，前台需要向服务端传一个数据，服务端以List集合形式接收，大致形式如下：
### 前端
```js
//userIds是checkbox选中的主键值数组
this.$http.get(url, {params: this.userIds}).then(resp => {
    //业务逻辑
});
```
### 后端
```java
@GetMapping("/getList")
public Result getList(
    @RequestParam(value="userIds"), required = false) List<Integer> userIds){
    //业务逻辑
}
```

这样处理服务端接收到的`userIds`始终为`null`，试过使用POST请求，或者带上@RequestBody注解，都不行。
后来看了一下其他人写的方式，发现只需要在前端传参数时，将`userIds`数组变成字符串形式即可，即
```js
this.$http.get(url, {params: this.userIds.toString()}).then(resp => {
    //业务逻辑
});
```