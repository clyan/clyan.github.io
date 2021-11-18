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

### box-shadow 与 outline的区别
`box-shadow` 不能实现虚线, 但可实现多层边框

`outline` 只能设置一个值，只能实现边框，可以实现虚线边框


[完整案例地址](https://codesandbox.io/s/css3zong-jie-w0ymf?file=/%E5%A4%9A%E9%87%8D%E8%BE%B9%E6%A1%86.html:389-425)