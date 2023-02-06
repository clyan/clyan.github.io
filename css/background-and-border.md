## 半透明边框
### 半透明的实现方案
1. rgba(255, 255, 255, 0.5);
2. hsla(0, 0%, 100%, 0.5);
3. opacity: 0.5;

现在我们要实现一个半透明的边框，首先使用hsla或rgba设置边框半透明

```css
body {
    background: #000;
}
.box {
    width: 100px;
    height: 100px;
    background: red;
    border: 10px solid hsla(0, 0%, 100%, 0.5);
    /* border: 10px solid rgba(255, 255, 255, 0.5); */
}
```
除以上属性之外，加上`background-clip`
```css
.box {
  background-clip: padding-box;
}
```

**效果分别为左右两图:** 

![](https://i.loli.net/2021/11/18/tnfuRJWeMKkiAvI.png)

对比如上两个效果，可以看到在不加`background-clip`前，边框颜色为半透明红色，背景颜色会填充至border，这是background背景的默认行为。

### background-clip

我们可以通过`background-clip`进行修改背景填充的范围。

`background-clip` 的属性值：
1. border-box (默认)： 背景填充至 `边框 + 内边距 + 内容区域`
2. pading-box: 背景填充 `内边距 + 内容区域`
3. content-box： 背景填充 `内容区域`

[完整案例地址](https://codesandbox.io/s/css3zong-jie-w0ymf?file=/%E5%8D%8A%E9%80%8F%E6%98%8E%E8%BE%B9%E6%A1%86.html)


## 多重边框
### 边框的实现方案
1. border: 实现一层边框（最常规的方案）
2. box-shadow：可实现一层或多层边框
3. outline: 可实现一层或配合border实现两层

### box-shadow
**语法格式：**

**box-shadow:** h-shadow v-shadow [blur, [spread, [color, [inset ]]];

**box-shadow:** x轴偏移量 y轴偏移量 [模糊值, [扩散半径, [颜色, [阴影处于内侧 ]]];


**使用box-shadow来实现多重边框**

使用 `,` 设置多个阴影， 需要注意的是 `box-shadow`  最先设置的值位于最上层，如果半径大于下一层则会遮住下面的层级

```css
.box {
    width: 100px;
    height: 100px;
    box-shadow: 0 0 0 20px #f4aab9, 
                0 0 0 40px #66ccff;
}
```

**效果如下：**
![](https://i.loli.net/2021/11/19/ZK5w2nP9La3AElN.png)

**使用内侧阴影 inset：** 

```css
box-shadow: 0 0 0 20px #f4aab9 inset, 0 0 0 40px #66ccff inset;
```
注意观察颜色的顺序其实是与上面的相反的

**效果如下：** 
![inset应用于元素内侧](https://i.loli.net/2021/11/19/zlivMtIuhLQEsjo.png)

对比不使用 `inset` 的情况下，从表现上来说整体盒子要小很多，但是其实盒子的大小是保持不变的

**注意点:**
1. box-shadow 不会影响布局，不会受到box-sizing的影响 （inset会影响）
2. box-shadow 默认出现在元素的外侧，所以不会响应事件, 使用inset 并配合padding可实现点击
    ```css
    .box3 {
        box-shadow: 0 0 0 20px #f4aab9 inset, 0 0 0 40px #66ccff inset;
        padding: 40px;
    }
    .box4 {
        box-shadow: 0 0 0 20px #f4aab9 inset, 0 0 0 40px #66ccff inset;
        padding: 40px;
        box-sizing: border-box;
    }
    ```
    ```html
    <div class="box box3">boxshow inset padding</div>
    <div class="box box4">boxshow inset padding box-sizing</div>
    ```
    **效果如下：** 
    ![](https://i.loli.net/2021/11/19/mtK3Zvdf18BVPIa.png)

    先使用`box-shadow`设置边框，再使用padding撑开盒子大小，使得`box-shadow`设置边框不占用内容的区域。
    
    此时`box3` 的宽度为 width(100px) + padding(40px) = 140px; `box4` 的宽度为 100px;

### outline

**语法：** outline: outline-color outline-width outline-style (不分顺序)

使用outline设置多重边框，并且可以设置虚线边框，同时配合 `outline-offset` 负值可实现如下第二张图的缝边效果
```css
.box5 {
    background: #66ccff;
    border: 10px solid #655;
    outline: 5px solid #f4aab9;
}
.box6 {
    background: #66ccff;
    outline: 2px dashed #f4aab9;
    outline-offset: -10px;
    padding: 10px;
    box-sizing: border-box;
}
```
```html
  <div class="box box5">outline</div>
  <div class="box box6">outline-offset</div>
```
**效果如下：** 
![](https://i.loli.net/2021/11/19/KkGdfnVJQYF5cLp.png)

**注意：** `outline` 一般情况下只支持矩形， 不像`border-radius`可以设置圆角，只有在`Firefox`下 使用 `-moz-outline-radius` 设置，但是不推荐使用该功能，兼容性太差

**效果如下：**  ![](https://i.loli.net/2021/11/21/FSyJfiI7D8zsgVe.png)

### box-shadow 与 outline的区别
`box-shadow` 不能实现虚线, 但可实现多层边框

`outline` 只能设置一个值，只能实现边框，可以实现虚线边框

[完整案例地址](https://codesandbox.io/s/css3zong-jie-w0ymf?file=/%E5%A4%9A%E9%87%8D%E8%BE%B9%E6%A1%86.html:389-425)


## 背景定位
日常开发中经常要使用背景图片之类的，但对background及其系列属性不够了解，
导致每次都写的很迷糊,重新学习一下
### background
**该属性是以下属性的简写**

**基本顺序如下:** bg-color | bg-image | bg-position [ / bg-size]? | bg-repeat | bg-attachment | bg-origin | bg-clip

1. `background-color:`    背景颜色。
2. `background-image:`    背景图像, 可以是真实的图片路径, 也可以是创建的渐变背景;
3. `background-position:` 背景图像的位置;
4. `background-size:` 背景图像的大小;
5. `background-repeat:`   背景图像的铺排方式;
6. `background-attachment:`   背景图像是滚动还是固定;
7. `background-origin:`   背景图像显示的原点[background-position相对定位的原点];
8. `background-clip:` 设置背景图像向外剪裁的区域;

**以上属性，顺序并不固定，但是需要有一些注意点：**

1. `background-position` 和 `background-size` 属性, 之间需使用`/`分隔, 且`position`值在前, `size`值在后。
2. 如果同时使用 `background-origin` 和 `background-clip` 属性, `origin`属性值需在`clip`属性值之前, 如果`origin`与`clip`属性值相同, 则可只设置一个值。

同时`background` 和 `box-shadow` 一样 是个复合属性，可以设置多个值。

每组属性间使用逗号分隔, 其中`背景颜色`不能设置多个且只能放在最后一组。

如设置的多组属性背景图像之间存在重叠, 则`前面的`背景图像会`覆盖`在`后面的`背景图像上。

```css
.box2 {
    background: url(https://i.loli.net/2021/11/21/ElxJBf8X7pu1aZm.png) 0% 0%/60px 60px no-repeat padding-box,
        url(https://i.loli.net/2021/11/21/ElxJBf8X7pu1aZm.png) 40px 10px/110px 110px no-repeat content-box,
        url(https://i.loli.net/2021/11/21/ElxJBf8X7pu1aZm.png) 140px 40px/200px 100px no-repeat content-box #58a;
}
```
`蓝色背景颜色`对应的由最后一行值设置的 `#58a` 同时前面的属性不能设置背景颜色 

**效果如下：**
![image.png](https://i.loli.net/2021/11/21/a4SN23lFZOsbwDj.png)

### background-position

用于设置背景图像的位置, 默认值: 0% 0%, 效果等同于 left top。

取值说明:
                                        
1. 如果设置一个值, 则该值作用在横坐标上, 纵坐标默认为50%(即center) ;
2. 如果设置两个值, 则第一个值作用于横坐标, 第二个值作用于纵坐标, 取值可以是方位关键字`left / top / center / right / bottom`,
   
   如 `background-position: left center;` 也可以是百分比或长度, 百分比和长度可为设置为负值, 如: `background-position: 10% 30px;`

3. 另外css3支持3个值或者4个值, 注意如果设置3个或4个值, 偏移量前必须有关键字, 如: `background-position: right 20px bottom 30px;`

效果如下： 
![SupportingImages.png](https://i.loli.net/2021/11/21/frCtYjLTlws7BvI.png)

[案例地址](https://codesandbox.io/s/css3zong-jie-w0ymf?file=/%E8%83%8C%E6%99%AF%E5%AE%9A%E4%BD%8D.html&resolutionWidth=320&resolutionHeight=675), [MDN 案例效果调试地址](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position)

### background-size
用来指定背景图像的大小。默认值: auto

取值说明: 
1. 可使用 长度值 或 百分比 指定背景图像的大小, 值不能为负值。
2. 如果设置一个值, 则该值用于定义图像的宽度, 图像的高度为默认值 auto, 根据宽度进行等比例缩放; 如果设置两个值, 则分别作用于图像的宽和高。 
3. `auto` 默认值, 即图像真实大小。 
4. `cover` 将背景图像等比缩放到完全覆盖容器, 背景图像有可能超出容器。(即当较短的边等于容器的边时, 停止缩放) 
5. `contain` 将背景图像等比缩放到宽度或高度与容器的宽度或高度相等, 背景图像始终被包含在容器内。(即当较长的边等于容器的边时, 停止缩放)

效果如下： 
![](https://i.loli.net/2021/11/21/eOcySCirvTkPYbE.png)


### background-origin
1. 用于设置 background-position 定位时参考的原点, 默认值 padding-box , 另外还有两个值: border-box 和 content-box。

![](https://i.loli.net/2021/11/21/1hvY6mTCbJdjftw.png)

### background-attachment
**属性值：**
1. `fixed` 背景相对于视口是固定的。 即使元素具有滚动机制，背景也不会随着元素移动。 (这与background-clip: text不兼容。)  
2. `local` 背景相对于元素的内容是固定的。 如果元素具有滚动机制，则背景与元素的内容一起滚动，背景绘画区域和背景定位区域相对于元素的可滚动区域而不是边框。  
3. `scroll` 背景相对于元素本身是固定的，不会随着元素内容滚动。 (它被有效地附加到元素的边界上。)
     
使用 [MDN 在线调试background-attachment](https://developer.mozilla.org/en-US/docs/Web/CSS/background-attachment) 查看效果更直观些

### background-clip
在半透明边框中已经说明过了

### background-repeat
background-repeat 属性用来设置背景图像铺排填充方式, 默认值: repeat 。

**取值说明:**
1. `repeat-x` 表示横向上平铺, 相当于设置两个值 `repeat` `no-reapeat`; 
2. `repeat-y` 表示纵向上平铺, 相当于设置两个值 `no-repeat` `repeat`; 
3. `repeat` 表示横向纵向上均平铺; 
4. `no-repeat` 表示不平铺;
5. `round` 表示背景图像自动缩放直到适应且填充满整个容器。 注意: 当设置为 `round` 时, `background-size`的 `cover` 和 `contain` 属性失效。
6. `space` 表示背景图像以相同的间距平铺且填充满整个容器或某个方向. 注意: 当 `background-size` 设置为 `cover` 和 `contain` 时, `background-rapeat` 的 `space` 属性失效。

**注意:** `background-repeat` 的 `round/space` 属性, 部分Firefox版本不支持。

使用 [MDN 在线调试background-repeat](https://developer.mozilla.org/en-US/docs/Web/CSS/background-repeat) 查看效果更直观些

## 圆角
### border-radius

**简述：**用于设置盒子的边角形状， 当垂直半径与水平半径相等时，表现的是一个圆角，不相等时，则是一个椭圆角

**语法：** 

borde-radius:  水平 / 垂直 

是以下属性的简写

**border-radius：** 

​	(左上)border-top-left-radius    (右上)border-top-right-radius    

​	(右下)border-bottom-right-radius    (左下)border-bottom-left-radius

​	`水平 / 垂直` 的取值都是四个角的水平 / 垂直的缩写， `/`两边都各自按 TRBL 顺序取值。

**TRBL指的是：** 上右下左，对于border-radius而言则是：`左上`、`右上`、`左下`、`右下 `

**当只设置 `/` 一边的值时，代表的是水平与垂直的半径一致，也就是圆角，规则如下**

**1个属性值时：**

​	**例：**border-radius: 20px; 代表的是: border-radius: 20px 20px 20px 20px

**2个属性时:**

​	**例：**border-radius: 20px  30px; 代表的是: border-radius: 20px 30px 20px 30px

​	**20px设置的是：** `左上` 与 `右下`

​	**30px设置的是：** `右上` 与 `左下`

**3个属性时:**

​	**例：** border-radius: 20px  30px 50px; 代表的是： border-radius: 20px 30px 50px 30px

​	**20px设置的是：** `左上` 

​	**30px设置的是：** `右上` 与 `左下`

​	**50px设置的是：** `右下`

**4个属性时:**  `左上` `右上`   `右下`   `左下` 一 一对应 border-radius: 20px 30px 40px 50px

以上的规则都适用`水平/ 垂直`的简写: 以4个属性为例子：

水平 / 垂直保持一致
```
  border-top-left-radius: 20px 20px;
  border-top-right-radius: 30px 30px;
  border-bottom-left-radius: 40px 40px;
  border-bottom-right-radius: 50px 50px;
```
所以表现的是圆角

**border-left-top-radius：** 

![](https://s2.loli.net/2021/12/08/I7xG8lH6fDiZkAc.png)

**语法：** 

border-top-left-radius: 水平  垂直

如： border-top-left-radius: 10px 20px;

### 当设置50%与100%的区别

**当使用border-radius时设置四个角：**  使用50% 与100%无区别

```html
.box {
    background: #ccc;
    height: 100px;
    width: 100px;
    margin-right: 10px;
}
.box11 {
	border-radius: 50%;	
}
.box12 {
	border-radius: 100%;
}
<div class="box box12"></div>
<div class="box box13"></div>
```

![](https://s2.loli.net/2021/12/08/cMj5R3d4oJlr9n1.png)

当只设置一个角时与两个角时：设置100%

如果两个相邻角的半径之和超过了相应盒子边的长度，那么浏览器要重新计算，以保证两者不会重合。

假设有一个 100px 的盒子，若 **border-top-left-radius:100%;** 则盒子会变成一个半径为 100px 的 1/4圆。（如下图左）

这个时候，如果我们再给一个 **border-top-right-radius:100%;** 此时相邻的两个角的半径之和已经超过了盒子的长度，浏览器需要重新计算。计算的规则就是同时缩放两个圆角的半径，直至两个相邻角的半径和为盒子的长度。也就是说，当两个圆角的半径为 **50%** 的时候，圆角正好符合 W3C 标准。（下图右）

```html
.box13 {
    border-top-left-radius: 100%;
    border-top-right-radius: 100%;
}
.box14 {
	border-top-left-radius: 100%;
}      
<div class="box box13"></div>
<div class="box box14"></div>
```

![](https://s2.loli.net/2021/12/08/4A32emOcYjSEKXU.png)

### 其他注意点

当border-radius的值设置大于border的宽度时，内半径会出现圆角

```css
border: 15px solid green;
border-radius: 25px;
```
![](https://s2.loli.net/2021/12/08/tGoFAhxXlkjZgEe.png)

border-radius小于等于border时
```css
border: 15px solid green;
border-radius: 15px;
```

![](https://s2.loli.net/2021/12/08/lxK1oEaXTqdsy4t.png)


**详解链接：** https://www.cnblogs.com/lhb25/archive/2013/01/30/css3-border-radius.html

**上面的demo地址:**  [codeSandbox](https://codesandbox.io/s/css3zong-jie-w0ymf?file=/borderRadius.html)

**参考地址：** [MDN border-radius](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-radius) [MDN border-top-left-radius ](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-left-radius)  [菜鸟教程](https://www.runoob.com/css3/css3-border-radius.html)