---
title: 'setTimeout定时器中this的指向问题'
Date: 2019-04-02 17:41:00
Tag: vue
---
在项目中使用setTimeout定时器去控制按钮的加载状态显示时，遇到定时器失效的问题。  
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
>使用function定义的函数，this的指向随着调用环境的变化而变化；  
箭头函数中的this指向固定不变：定义函数的环境。

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