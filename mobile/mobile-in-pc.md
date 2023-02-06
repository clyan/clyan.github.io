**前置条件：** 移动端适配使用的是如下适配方案, 也就是使用的rem作为适配，该示例中用到的是13.333333vw, 使得1rem等于100px。

```typescript
html{
    font-size: 13.333333vw;
}
body {
	font-size: 0.14rem; // 等于 14px
}
```

**问题现状:**  移动端页面在pc端页面显示异常
先写一段简单的vue代码，生成4个盒子和一个底部fixed定位的盒子。
```vue
<div class="list" v-for="i in 4" style="height: 1rem;border: 1px solid;">
     这是一段文本 {{ i }}, font-size: 0.18rem
</div>
 <div class="bottom" style="background: gray;position: fixed; bottom: 0;height: 2rem;width: 100%;color:#fff">
     fixed定位
 </div>
```
以上代码在移动端显示如下：


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ba66b893573420b9e18591d91de617c~tplv-k3u1fbpfcp-watermark.image?)

在pc端展示如下：


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef806eb944c24db285dcecdc146a8398~tplv-k3u1fbpfcp-watermark.image?)

以上对比很明显发现pc端访问大了很多，明显底部fixed定位遮住了部分盒子内容，在pc端查看时就很不友好。

**问题根源**：适配是使用的rem根据html的font-size做适配，在pc端的时候font-size会变得很大，同时没有固定盒子的宽度，整个页面会充满屏幕，所以就导致访问出问题，知道了问题所在那么也就有了对应的解决方案了。

**解决方案：** 
1. 解决根元素无限变大
2. 限制pc端显示页面最大宽度
3. 解决fixed定位相对于窗口，修正为相对于body
4. 将滚动条宽度设置为0，避免滚动条宽度占用内容宽度
```vue
//判断是否是手机端
 function isMobile() {
  return navigator.userAgent.match(
    /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
  );
}
// 如果不是移动端则认为是pc端
if (!isMobile()) {
  let html = document.querySelector("html");
  let body = document.querySelector("body");
  // 设置为 50px, 为什么不是100px, 因为实际开发时都是二倍图，所以写50px;
  html.style.fontSize = "50px";
  // 页面始终为一屏高
  html.style.height = "100vh";
  // 使body居中
  html.style.display = "flex";
  html.style.justifyContent = "center";
  html.style.alignItems = "center";
  html.style.overflow = "hidden";
// 固定宽度
  body.style.width = "375px";
 // 固定高度
  body.style.height = "812px";
  // 加个阴影点缀一下。
  body.style.boxShadow = "0 0 20px rgba(0, 0, 0, .5)";
  // 最后重点：pc端fixed定位,修正为body, 而不是视口
  body.style.transform = "translate(0)";
}
```
重点注意：body.style.transform = "translate(0)"; 很重要，不然应用了fiexd定位的相对的还是窗口，宽度就全屏了，而不是 375px了。

如下图：来自MDN对fixed的描述

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb5db3fc49434a7db38596a4af1f02a2~tplv-k3u1fbpfcp-zoom-1.image)


最后将pc端滚动条宽度改成 0
```
/* 设置滚动条的样式 */
::-webkit-scrollbar {
    width:0;
}
```
最后就完美的解决的pc端访问移动端项目啦, 如下图：


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e28ee9d735b428f9c045acfdd992d83~tplv-k3u1fbpfcp-watermark.image?)

以上的基础上稍加修改应该可以适用于其它移动端适配方案，欢迎指出错误！