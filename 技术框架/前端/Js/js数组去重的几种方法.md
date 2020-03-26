# JS数组去重的几种方法
### 1. 创建空数组，利用indexOf()方法，检测旧数组的项是否在新数组中，如果在则接着遍历下一个元素，不在则将当前元素放到新数组中，最后的新数组就是去重后的结果集。
```js
filterElements(list){
  let newArray = [];
  for(let i=0; i<list.length; i++){
    if(newArray.indexOf(list[i]) === -1){
      newArray.push(list[i]);
    }
  }
  return newArray; // 返回去重后的结果集
}
```
### 2. 创建空数组和空对象，判断数组是否在对象中。
```js
filterElements(list){
  let temp = {};
  let newArray = [];
  for(let i=0; i<list.length; i++){
    if(!temp[list[i]]){ // 如果temp中没有list[i]
      temp[list[i]] = true; // list[i]存到temp
      newArray.push(list[i]); // list[i]添加到新数组
    }
  }
  return newArray; // 返回去重后的结果集
}
```

### 3. 下标判断法
```js
filterElements(list){
  let newArray = [list[0]];
  for(let i=1; i<list.length; i++){ // 从list的第二项开始遍历
    if(list.indexOf(list[i]) === i){ // list的第i项是i时，存入新数组
      newArray.push(list[i]);
    }
  }
  return newArray; // 返回去重后的结果集
```

### 4. 排序后去重
```js
filterElements(list){
  let newArray = [];
  list.sort(); // 数组排序
  for(let i=0; i<list.length; i++){
    if(list[i] != newArray[newArray.length-1]){
      newArray.push(list[i]);
    }
  }
  return newArray; // 返回去重后的结果集
}
```

### 5. 使用ES6的 new Set(list)去重，这种去重不适合元素为复杂对象的数组。
```js
let result = new Set(list); // 直接返回去重后的结果集
```