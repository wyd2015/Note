---
title: '@RequestBody与@ResponseBody的区别'
date: 2019-07-02 11:10:02
categories: 'springboot'
tag: '@RequestBody', '@ResponseBody'
---
# @ResponseBody
一般作用于方法上，表示该方法的返回结果直接写入http响应正文中，一般在异步获取数据时使用。通常是在使用@RequestMapping后，
返回值通常被解析为`跳转路径`，加上`@ResponseBody`后，返回结果就不会被解析为跳转路径，而是以其他格式的
数据（json、xml）直接写入response中。

# @RequestBody
作用于参数上，将HTTP请求正文插入方法中，使用合适的HttpMessageConverter将请求写入某个对象（用DTO接收参数）。

## 使用时机
1. GET、POST方式下，根据request、header、Content-Type的值来判断：  
>- application/x-www-form-urlencoded：可选，这种情况的数据使用@RequestParam或@ModelAttribute也可以处理，也可以使用@RequestBody处理；
>- multipart/form-data：不能使用@RequestBody处理；
>- 其他格式：必须使用@RequestBody处理，格式包括：application/json、application/xml等。

2. PUT方式下，根据request、header、Content-Type的值来判断：  
>- application/x-www-form-urlencoded：必须使用@RequestBody进行处理；
>- multipart/form-data：不能使用@RequestBody处理；
>- 其他格式：必须使用@RequestBody处理，格式包括：application/json、application/xml等。

[request的body部分的数据编码格式由header部分的Content-Type指定](#)
```js
/* 点击上报按钮 */
submitUpload(){
  //表单形式提交 this.dataForm，json处理
  this.formData.append('wp', JSON.stringify(this.dataForm));
  this.$refs.upload.submit();
  let config = { headers: {'Content-Type': 'multipart/form-data'} };//指定表单类型提交
  this.$http.post('/upfile', this.formData, config).then(resp=>{
    //上传成功后的逻辑
    ...
  }).catch(e=>{console.log(e)})
}
```