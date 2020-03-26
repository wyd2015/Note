# css中position属性的用法
记住一句话：
>**relative定位父级元素；absolute定位内部元素。**

如果用position来布局，**父级**元素的position属性必须为**relative**；
而定位于父级内部某个位置的元素，最好用absolute，因为它不受父级元
素的padding属性影响.
>也可以用relative就是计算的时候要把父级元素的**padding**算进去。

## absolute
默认参照浏览器左上角定位，配合top, right, bottom, left（以下简称trbl）进行定位，属性如下：
- 无 trbl 情况下，参照父级左上角； 
- 如果没有父级，则参照浏览器左上角； 无父级元素，但存在文本，则以最后一个文字的右上角为原点，但不断开文字，覆盖于上方；
- 如果设定有 trbl，且父级没有position属性，默认以浏览器左上角为原点开始定位，位置有 trbl 的属性值决定；
- 如果设定为 trbl，且父级有position属性（无论是absolute还是relative），默认以父级左上角为原点开始定位，位置由 trbl 决定；

总结一下就是：
>- 父级设定position属性；
>- 设定top, right, bottom, left属性值。

## relative
默认参照父级原始点为起点定位。如果没有父级元素，以文本的**上一个元素的底部**为起始点，配合 trbl 进行定位；当父级元素内有padding属性值时，参照父级内容区的原始点进行定位。

- 不存在trbl，参照父级左上角；
- 没有父级，参照浏览器左上角；
- 没有父级，但存在文本，则以文本的底部为原始点进行定位，并将文字断开；
- 设定trbl，父级没设定position属性，以父级左上角为原点进行定位；
- 设定trbl，父级设定position属性，以父级左上角为原点定位，但若父级有padding属性，则以内容区域的左上角为原点进行定位。

总结一下：
>relative 均以**父级左上角**为起点定位，会受padding影响。