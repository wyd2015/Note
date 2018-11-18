# 区分escape、encodeURI、encodeURIComponent
在前端需要进行页面跳转传值时，如果传递的参数里含有特殊符号，如&、%等符号，如果不对参数作处理，目标页接收到的参数不是预期结果。这时候可以使用 `escape`、`encodeURI`、`encodeURIComponent` 这三个js方法进行处理后再传。

三者区别在于：  
1. `escape`：对`字符串`进行编码，使字符串在所有电脑上可读，编码之后的效果是 `%XX` 或 `%uXXX` 这种形式。
其中，`ASCII、数字、@&/+`，这几个字符串不会被编码，其余的都会。  
2. encodeURI、encodeURIComponent是针对URL编码，它们俩的区别在于编码的字符范围不同：
>encodeURI方法不会对下列字符编码：ASCII字母、数字、~!@#$&*()=:/,;?+'  
>encodeURIComponent方法不会对下列字符编码：ASCII字母、数字、~!*()'