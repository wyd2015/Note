---
title: 'springboot接收前端传过来的list'
Data: 2019-04-12 15:12:49
Tag: Springboot
---
项目中的收藏信息功能，想要将用户选中的信息以对象数组形式，用post请求发送到服务端，
然后服务端接收到的是list对象，然后遍历list，解析对象，将list中的信息进行解析、入库。

前端使用的是 `vue` + `vue-resource`，异步请求如下：  
这里有几个需要注意的地方：  
1. 组装好的数组里的每个对象的属性名不需要使用引号括起来，但使用引号也可以正常被接收；
2. 发送post请求时，要将 `emulateJSON` 设为 `false`才可以，这里是为了是数据以json串的形式进行提交，`{emulateJSON: true}` 是以表单方式提交数据；
3. 服务端接收list必须以实体对象形式，因此需要将List<***Object>作为一个属性封装到一个DTO类里，然后使用此DTO对象接收数据。
4. post请求中的参数名称，必须和DTO类中定义的list的属性名对应上。
```js
/* 批量收藏 */
collectBatch(){
  let collectList = [];
  // this.infoList是所有的信息集合
  this.infoList.forEach(item => {
    // this.boxGroup是用户选择的信息主键id
    if(this.boxGroup.includes(item.localClassifyId)){
      collectList.push({
        msgKey: item.msgKey,
        msgUrl: item.msgUrl,
        msgUname: item.msgUname,
        msgTitle: item.msgTitle,
        msgUkey: item.msgUGChKey,
        situation: item.situation,
        msgKeyIx: item.msgKeyIndex,
        msgAbstract: item.msgAbstract,
        msgPublishTime: item.msgPublishTime
      });
    }
  })
  let url = gl.serverURL + "/collectBatch";
  // 这里collectList是DTO对象的一个属性，所以需要用{}括起来，否则传过去的是一个数组，而不是对象
  this.$http.post(url,{collectList},{emulateJSON:false}).then(response => {
      // 收藏结果处理
    }, response => {}
  )
},
```

服务端代码：  
```java
/**
* 批量收藏
* @param batchObj 这里必须以实体对象形式接收，不能直接使用List<Object>接收
* @return
* @return:Result
*/
@PostMapping(value="/collectBatch")
public Result collectBatch(@RequestBody CollectBatchObj batchObj){
  Result result = new Result();
  List<Object> collectList = batchObj.getCollectList();
  result.setMessage("收藏成功！");
  return result;
}

// 实体对象
@Data
public class CollectBatchObj implements Serializable {
  private static final long serialVersionUID = -2472934879429867712L;
  private List<Object> collectList;
}
```


如果是需要接收整型数的list集合，可以直接使用List<Integer>的形式进行接收，代码如下：  
- 前端代码：  
```js
setCollectState(){
  let idList = [this.activeInfo.localClassifyId];
  let patchParams = {
    isCollect: this.isCollect,
    localCalssifyIds: idList.toString() // 这里数组后加上.toString()方法即可
  }
  let patchURL = gl.serverURL + '/warning/classify/collect';
  this.$http.patch(patchURL, patchParams, {emulateJSON: true}).then(resp => {
    let ret = resp.body;
    let status = ret.status;
    if(status == 0){
      this.refreshInfoCollectStatus(isCollect);
    }
  })
},
```
- 服务端代码：  
```java
@PatchMapping("/classify/collect")
public Result localClassifyCollect(HttpServletRequest request,
  @RequestParam(value = "isCollect", required = true) int isCollect,
  @RequestParam(value = "localCalssifyIds", required = true) List<Integer> localCalssifyIds){
  Result result = new Result();
  return result;
}
```