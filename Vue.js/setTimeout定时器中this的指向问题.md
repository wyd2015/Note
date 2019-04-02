---
title: 'setTimeout定时器中this的指向问题'
Date: 2019-04-02 17:41:00
Tag: vue
---

在项目中使用setTimeout定时器去控制按钮的加载状态显示时，遇到定时器时效的问题。  
`在点击导出按钮后，按钮一直处于加载状态不再改变...`

【代码还原】
```html
<template>
  <div>
    <Button type="primary" :loading="excelLoading" @click="export">导出</Button>
  </div>
</template>

<script>
  export default {
    data(){
      return {
        excelLoading: false
      }
    },
    methods: {
      export() {
        this.excelLoading = true;
        let url = gl.serverURL + '/common/tool/excel';
        window.location.href=url;
        setTimeout(function(){
          this.excelLoading = false;
        }, 1000);
      }
    }
  }
</script>
```

【问题原因】  
>当在vue中使用定时器，并在定时器内部的function里直接使用this时，由于setTimeout函数调用的代码运行在与所在函数完全分离的执行环境上，这就使得this所指向的不再是`当前vue`组件，而是浏览器的`window`对象。this的指向发生变化，自然也就不然改变`excelLoading`的值。

【解决办法】  
1. 在setTimeout定时器内使用`箭头函数`代替`function`关键字：  
```js
setTimeout(()=> {
  this.excelLoading = false;
}, 1000);
```
2. 定义变量暂存this对象：  
```js
export() {
  this.excelLoading = true;
  let url = gl.serverURL + '/common/tool/excel';
  window.location.href=url;

  let vm = this;
  setTimeout(function(){
    vm.excelLoading = false;
  }, 1000);
}
```