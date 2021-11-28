# 模块化
## 为什么需要模块化
 抽离公共代码，隔离作用域，避免变量冲突

## 模块化的分类
### CommonJS
> node中使用的规范，只能由于服务端

**person.js**

```js
module.exports = {
    name: '123'
}
```
**index.js**

```js
    const person = require('./person')
    console.log(person)
```
    
### AMD
> 为了解决浏览器端模块化开发，第一个出现的浏览器模块化规范
> 
> 代表作：requireJS
> 
> [阮一峰： Javascript模块化编程（三）：require.js的用法](https://www.ruanyifeng.com/blog/2012/11/require_js.html)，  [requireJS官方文档](https://requirejs.org/docs/api.html)

步骤：
1. 引入require.js
2. 使用define定义模块
3. 使用requirejs引入模块


**index.html**
```html
<!-- data-main属性的作用是，指定网页程序的主模块 -->
<script defer src="https://cdn.bootcdn.net/ajax/libs/require.js/2.3.6/require.min.js" data-main="js/main"></script>
</html>
```

**js/main.js**
```js
requirejs(["custom-dep-function"], function(depFunction) {
    console.log(depFunction)
});
```
**js/custom-dep-function.js**

```js
// custom-dep-function 依赖custom-normal-variable模块
define(["custom-normal-variable"], function(dep) {
    console.log(dep)
    return {
        ...dep,
        opacity: 0
    }
});
```

**js/custom-normal-variable.js**
```js
    // 定义普通的变量 
    define({
        color: '#fff',
        fontSize: '16px'
    });
```
    
### CMD
> 对AMD的改进
> 代表作： Sea.js， 作者是阿里的玉伯

### IIFE

```js
var b = 2;
;(function foo(global) {
    const a = 123;
    console.log(a) // 123
    console.log(global.b) // 2
}(window))

;(function foo(global) {
    const a = 456;
    console.log(a) //456
    console.log(global.b) // 2
})(window)
    console.log(b) // 2
    console.log(a) //  a is not defined
```

### UMD
> AMD, CommonJS, IIFE的结合体
```js
(function (factory) {
    typeof define === 'function' && define.amd ? define(factory) :
    factory();
})((function () { 'use strict';

    const adventurer = {
        name: 'Alice',
        cat: {
        name: 'Dinah'
        }
    };
    const dogName = adventurer.dog?.name;
    console.log(dogName);

}));

```
    
### ESM
   
**a.js**
```js
const a = 1
export default a
```
**index.js**
```js
import a from './a'
console.log(a) // 1
```