---
title: 'el-upload一次请求上传多张图片'
data: 2019-07-01 19:32:57
categories: 'vue'
tag: 'vue', 'el-update'
---
# 问题还原
使用element-ui 2.x 版本的el-upload组件上传图片时，如果是一次上传多张，
默认会发送相应次数的请求，这样一来就会导致最终只有一张图片路径被记录到数据库。

# 解决方法
使用el-upload中的 `http-request` 来自定义上传方法的实现，以 `FormData` 的形式进行上传，图片和参数分开处理。
## 前端处理：
```html
<!-- template部分 -->
<el-form-item label="上传附件" class="is-required">
  <el-upload class="upload-demo" ref="upload"
    multiple :limit="3" 
    list-type="picture"
    :action="serverurl+'/recCode/upfile'"
    :http-request="uploadFile"
    :on-change="uploadOnChange"
    :file-list="fileList"
    :data="dataForm"
    :auto-upload="false">
    <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过2M</div>
  </el-upload>
</el-form-item>
<el-form-item>
  <el-button size="small" @click="cancel">取消</el-button>
  <el-button size="small" type="primary" @click="submitUpload">上报</el-button>
</el-form-item>
```
```js
/* script部分 */
export default {
  data() {
    return {
      formData: new FormData(),//初始化表单数据对象
      dataForm: { reportContent: '', reportAddress: '', id: '', orderId: ''},
    }
  },
  methods: {
    /* 手动上传，覆盖组件默认的上传方法 */
    uploadFile(file){
      this.formData.append('file', file.file);//此处添加上传的图片到formData
    },
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
  }
}
```
## 服务端处理
1. 如果前端传的是对象：  
服务端直接从request中取出FormData中的对象参数，取出来的是json串，使用JSONObject解析。

2. 如果前端传的是单个的参数：  
服务端可以直接使用对应类型的参数+@RequestParam进行接收。

```java
import net.sf.json.JSONObject;

@ResponseBody
@PostMapping("/upfile")
public Result loadAtt(HttpServletRequest request,
  @RequestParam(value = "file", required = false) MultipartFile[] file) {//用file接收上传的图片
  String order = request.getParameter("wp");//从request请求中取参数
  JSONObject jsonObject = JSONObject.fromObject(order);
  WpOrderResult orderResult = (WpOrderResult) JSONObject.toBean(jsonObject, WpOrderResult.class);
  Integer reportUserId = getUser(request).getId();
  return receiveCodeService.upfile(orderResult, file, reportUserId);
}
```