# vue-cli生成的assets与static目录的区别
#### 1. **assets**
在项目编译过程中会被webpack处理解析为模块依赖，只支持相对路径的形式。如
```html
<img src="./logo.png">

.logo{
    background: url(./logo.png);
}
```
以上`./logo.png`均是使用的相对路径，将有webpack解析为模块依赖。
#### 2. **static**
这个目录就是存放第三方文件的地方，里面的文件不会被webpack解析、处理。最终build时会被直接复制到最终的打包目录(dist/static)下。必须使用绝对路径引用这些没有被webpack解析过的文件。这是通过`config.js`文件中的`build.assetsPublic`和`build.assetsSubDirectory`链接来确定的。任何放在static目录下的文件需要以绝对路径的形式引用：/static/filename.png

## 用js动态加载assets或者图片时出现404状态码
代码示例：
```javascript
<li v-for="(image, index) of imgs" :key="index">
    <img :src="image.src">
</li>

data(){
    return {
        imgs: [{./1.png}, {./2.png}]
    }
}
```
项目实际跑起来后发现图片不显示，错误码404.  
#### **[原因](#)**：在webpack中会将图片当作模版来使用，因为是动态加载，所有url-loader无法解析图片地址，然后`npm run dev`或`npm eun build`之后导致路径没有被加工处理【被webpack解析到的路径都会被解析为/static/img/filename.png】

#### **[解决办法](#)**：
1. 将图片作为模块加载进去，比如：  
```javascript
images:[{src: require('./i.png')}, {src: require('./2.png')}];
```
2. 将图片放到static目录下，但必须写成绝对路径，如:
```javascript
images: [{src: "static/1.png"}, {src: "static/2.png"}];
```
#### 如果本地图片过多，可以这样简化操作：
1. 在static文件夹下新建一个json文件夹；
2. 把图片路径全部录入到一个json文件中，并将此json文件放到第1步创建的json文件夹中；
```json
{
    "images": [{
        "src": "/static/home/logo.png"
    },{
        "src": "/static/home/ladar.png"
    }]
}
```
3. 在vue文件中引入json文件。
```javascript
import imgSrc from "../../static/home/img.json";
export default {
    data(){
        return {
            imgs: imgSrc.images;
        }
    }
}
```