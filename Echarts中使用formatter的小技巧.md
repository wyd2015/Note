# Echarts中使用formatter的小技巧

使用formatter格式化tooltip文本时，因为参数的具体情况不知道，格式化时感觉无从下手，有个小技巧可以摆脱这种尴尬的情况：

>使用JSON.stringify(params)打印json化后的params参数，知道params的结构就好办了！

```javascript
let option = {
	tooltip: {
		trigger: 'axis',
		formatter: function(params){
			console.log(JSON.stringify(params));
		}
	}
}
```

$PPI = d_p/d_i = sqrt[2]{w^2_p+h^2_p}/d_i$