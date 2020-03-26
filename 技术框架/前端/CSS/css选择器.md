---
title: css选择器
catagory: CSS
---
## 1. 基本选择器
|序号|选择器|含义|
|--|--|--|
|01|**\***|通用元素选择器|
|02|**tagName**|标签选择器|
|03|**.**|类选择器|
|04|**#**|id选择器|
示范：
```html
<div class="content">
    <span id="link"></span>
</div>
```
```css
<!-- 通用元素选择器 -->
* {
    margin: 0;
    padding: 0;
}

<!-- 标签选择器 -->
div {
    display: inline-flex;
    align-items: center;
}

<!-- 类选择器 -->
.content {
    display: inline-flex;
    align-items: center;
}

<!-- id选择器 -->
#link {
    color: blue;
}
```

## 2. 多元素组合选择器
|序号|选择器|含义|
|--|--|--|
|05|**E,F**|多元素选择器，同时匹配所有E元素或F元素，EF之间用逗号分隔|
|06|**E F**|后代元素选择器，匹配所有E元素的后代F元素，EF间用空格分隔|
|07|**E>F**|子元素选择器，匹配所有E元素的子元素F|
|08|**E+F**|毗邻元素选择器，匹配所有紧随E元素之后的同级元素F|

## 3. css2.1属性选择器
|序号|选择器|含义|
|--|--|--|
|09|**E[attr]**|匹配所有具有attr属性的E元素，不考虑它的值。`E在此处可省略`|
|10|**E[attr=val]**|匹配所有attr属性值为val的E元素|
|11|**E[attr~=val]**|匹配所有attr属性具有多个空格分隔的值、其中一个值等于val的E元素|
|12|**E[attr\|=val]**|匹配所有attr属性具有多个连字号分隔的值，其中一个值为val开头的E元素，主要用于lang属性|
```css
p[title]{
    color: #fff;
}

div[class=error]{
    color: red;
}

td[headers~=coll]{
    color: #000
}

p[lang|=en]{
    color: #120;
}

blockquote[class=quote][cite]{
    color: blue;
}
```
## 4. css2.1中的伪类
|序号|选择器|含义|
|--|--|--|
|13|**E: first-child**|匹配父元素的第一个元素|
|14|**E: link**|匹配所有未被点击的链接|
|15|**E: visited**|匹配所有已被点击的链接|
|16|**E: active**|匹配鼠标已经被按下，还有释放的E元素|
|17|**E: hover**|匹配鼠标悬停其上的E元素|
|18|**E: focus**|匹配当前获得焦点的E元素|
|19|**E: lang(c)**|匹配lang属性等于c的E元素|

## 5. css2.1中的伪元素
|序号|选择器|含义|
|--|--|--|
|20|**E: first-line**|匹配E元素的第一行|
|21|**E: first-letter**|匹配E元素的第一个字母|
|22|**E: before**|在E元素之前插入生成的内容|
|23|**E: after**|在E元素之后插入生成的内容|
```css

```


## 6. css3同级元素通用选择器
|序号|选择器|含义|
|--|--|--|
|24|**E ~ F**|匹配任何在E元素之后的`同级`F元素|
```css
p ~ url {
    background: #fff;
}
```

## 7. css3同级元素通用选择器
|序号|选择器|含义|
|--|--|--|
|25|**E[attr ^= "`val`"]**|匹配attr属性值以`val`开头的元素|
|26|**E[attr $= "`val`"]**|匹配attr属性值以`val`结尾的元素|
|27|**E[attr \*= "`val`"]**|匹配attr属性值包含`val`的元素|
```css
div[id^="nav"] {
    background: #ccc;
}
```


## 8. css3同级元素通用选择器
|序号|选择器|含义|
|--|--|--|
|28|**E:enabled**|匹配表单中激活的元素|
|29|**E:disabled**|匹配表单中已被禁用的元素|
|30|**E:checked**|匹配表单中被选中的`radio`或`checkbox`元素|
|31|**E:selection**|匹配用户当前已选中的元素|
```css
input[type="text"]:disabled {
    background: rgba(255, 255, 255, 0.2);
}
```


## 9. css3中的结构伪类
|序号|选择器|含义|
|--|--|--|
|32|**E:root**|匹配文档的根元素，对于HTML文档就是html元素|
|33|**E:nth-child(n)**|匹配其父元素的第n个子元素，第一个编号为`1`|
|34|**E:nth-last-child(n)**|匹配其父元素的倒数第n个子元素，第一个编号为`1`|
|35|**E:nth-of-type(n)**|与`:nth-child()`作用类似，但仅匹配使用同种标签的元素|
|36|**E:nth-last-of-type(n)**|与`:nth-last-child()`作用类似，但仅匹配使用同种标签的元素|
|37|**E:last-child**|匹配父元素的最后一个子元素，等同于`:nth-last-child(1)`|
|38|**E:first-of-type**|匹配父元素下使用同种标签的第一个子元素，等同于`:nth-of-type(1)`|
|39|**E:last-of-type**|匹配父元素下使用同种标签的最后一个子元素，等同于`:nth-last-of-type(1)`|
|40|**E:only-child**|匹配父元素下仅有的一个子元素，等同于`:first-child`, `:last-child`或`:nth-child(1)`, `:nth-last-child(1)`|
|41|**E:only-of-type**|匹配父元素下使用同种标签的唯一一个子元素，等同于`:first-of-type`, `:last-of-type`或`:nth-of-type(1)`, `:nth-last-of-type(1)`|
|42|**E:empty**|匹配一个不包含任何子元素的元素，`文本节点也被看作子元素`|
```css
p:nth-child(3) { color:#f00; }
p:nth-child(odd) { color:#f00; }
p:nth-child(even) { color:#f00; }
p:nth-child(3n+0) { color:#f00; }
p:nth-child(3n) { color:#f00; }
tr:nth-child(2n+11) { background:#ff0; }
tr:nth-last-child(2) { background:#ff0; }
p:last-child { background:#ff0; }
p:only-child { background:#ff0; }
p:empty { background:#ff0; }
```

## 10. css3的反选伪类
|序号|选择器|含义|
|--|--|--|
|43|**E: not(s)**|匹配不符合当前选择器的任何元素|
```css
:not(p) {
    border: 1px solid red;
}
```


## 11. css3中的 :target伪类
|序号|选择器|含义|
|--|--|--|
|44|**E: target**|匹配文档中特定id点击后的效果|
