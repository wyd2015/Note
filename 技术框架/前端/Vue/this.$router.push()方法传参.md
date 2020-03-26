# Vue中this.$router.push()传参失败
## 1. 问题复原：  
js中为传参
```javascript
login(){
    //传参
    this.$router.push({path: '/echarts/echart', params: {id: 3}});
}
```
在另一个组件中接收参数：
```javascript
console.log(this.$route.params.id);
```
这样接收到的 `params` 为 `undefined` 。


## 2. 问题分析：  
由于动态路由也是传递params的，所以在 `this.$router.push()` 方法中path不能和params一起使用，否则params将无效，需要用name来指定页面。  

路由配置文件中定义参数：
```javascript
export defaults new Router({
    routes: [{
        path: '/', 
        component: Login
    },{
        path: '/echarts/echart', 
        name: Echart, 
        component: Echart
    }]
})
```

组件中传参方式：  
```javascript
this.$router.push({name: Echart, params: {id: 3}});
```
组件接收参数方式：  
```javascript
this.$route.params.id
```
使用params传参在地址栏看不到传的参数，  
使用query传参数会在地址栏中显示！