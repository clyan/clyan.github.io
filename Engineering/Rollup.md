## rollup是什么

`Rollup`是一个模块打包工具，支持使用`ES Module`，将小块代码构建成一个大的 库 或 应用程序。

## 为什么使用

将项目拆分成小的单独文件中，这样开发软件通常会很简单，因为这通常会消除无法预知的相互影响，

以及显著降低了所要解决的问题的复杂度(complexity of the problem)，并且可以在项目最初时，就简洁地编写小的项目。

## rollup与webpack的使用场景

**大多数情况下， 对应用程序使用 `webpack`，对库使用 `Rollup`**

**Webpack：** 需要代码拆分，或者有大量静态资源，或者正在构建具有大量 CommonJS 依赖项的东西。

**Rollup:** 代码库是 ES2015 模块，构建供其他人使用的东西。

[Webpack 和 Rollup：相同但不同](https://medium.com/webpack/webpack-and-rollup-the-same-but-different-a41ad427058c)

## 项目案例

**使用rollup：**  Vue、React、D3、Three.js、Moment 等等

**使用webpack:**  elementUI、 Ant Design、Vant

## 入门使用

1. **安装rollup**

    ```bash
    npm i -g rollup
    ```

2. **初始化项目**
    **生成package.json**

    ```js
    npm init - y
    ```

    **编写需要打包的js**

    **a.js**

    ```js
    export const a = 123
    ```

    **b.js**

    ```js
    export const b = () => {}
    ```

    **main.js**

    ```js
    import { a } from './a';
    import { b } from './b'
    console.log(a)
    ```

    **rollup.config.js**
    ```js
    module.exports = {
        input: ['main.js'],
        output: [
            {
                file: 'dist/tm.cjs.js',
                format: 'cjs',  
                name: 'TM', 
            },
            {
                file: 'dist/tm.amd.js',
                format: 'amd',  
                name: 'TM', 
            },
            {
                file: 'dist/tm.esm.js',
                format: 'esm',  
                name: 'TM', 
            },
            {
                file: 'dist/tm.umd.js',
                format: 'umd',  
                name: 'TM', 
            },
            {
                file: 'dist/tm.iife.js', // 文件输出的地址与名称
                format: 'iife',  //  五种输出格式：amd /  es6 / iife / umd / cjs
                name: 'TM',     // 当format为iife和umd时必须提供，将作为全局变量挂在window(浏览器环境)下：window.TM =
                // sourcemap:true  //生成tm.map.js文件，方便调试
            },
        ],
    }
    ```

    **rollup支持5种模块格式：** 其中umd， 与iife 可用于浏览器端，esm 也可在浏览器中通过script标签上添加 type="module"的形式 引入： `<script type="module" src="xxxx"></script>`

3. **添加执行命令**

   **package.json**

   ```js
   "scripts": {
     "build": "rollup -c"
   }
   ```

4. **运行命令**

```bash
npm run build
```

**项目的基本结构如下：**

![image-20211115160532363.png](https://i.loli.net/2021/11/21/FUYuW7XbScrtwyP.png)

**执行结果：** 在dist目录下生成5个不同格式的文件，以umd格式为例，可以看到只打包了a , 因为b没有使用到，也就是rollup原生支持的tree-shaking特性，

**注意：**tree-shaking是由ESModule的静态编译的特性所支持的

![image-20211115160657349.png](https://i.loli.net/2021/11/21/BCV29QJqmd7aThv.png)

## 引入第三方包

比如我现在要使用一个jquery

```bash
npm install -S jquery
```

main.js中引入

```js
import $ from 'jquery'
console.log($)
```

再次打包

```bash
npm run build
```

控制台输出一堆警告，这是由于rollup默认识别相对路径`'./'`的形式，对于导入第三方包，默认认为是外部依赖

![image-20211115162035840.png](https://i.loli.net/2021/11/21/2HWdGXE8TNBoJvA.png)

修改`rollup.config.js`配置文件，添加`external`  (告诉rollup不要将此jquery打包，而作为外部依赖)，并在 `umd` 与 `iife` 配置下添加 ` globals： { 'jquery': '$'}` (告诉rollup 全局变量$即是jquery)

![image-20211115162720092.png](https://i.loli.net/2021/11/21/lrURo8YVTMgEe5L.png)

这个时候`jquery`，就并不会打包到最终的`bundle`中了

![image-20211115165426886.png](https://i.loli.net/2021/11/21/MbuRznP2EgUpZO1.png)

**但是想将jquery打包进去呢？**



需要告诉Rollup如何找到它，这里就需要安装另外一个插件

### @rollup/plugin-node-resolve

[node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve)

```js
npm install @rollup/plugin-node-resolve --save-dev
```

修改好`rollup.config.js`, 先将上面配置好的`globals`和`external`注释掉，再加入如下配置

```js
import { nodeResolve } from '@rollup/plugin-node-resolve';
export default {
  plugins: [nodeResolve()]
};
```

再次执行`npm run build`

![image-20211115163929559.png](https://i.loli.net/2021/11/21/pyCnsXixQEhlzj9.png)

**报错：** 原因是rollup只支持esm 引入模块的方式，我们打开node_modules下jquery包，看一下为什么

![image-20211115164539917.png](https://i.loli.net/2021/11/21/mATB6ps1zFt2gcG.png)

我们可以看到jquery只支持commonJS与IIFE格式，所以此时我们需要另外的插件将commonJS转换为ESM

### @rollup/plugin-commonjs

```bash
npm install @rollup/plugin-commonjs --save-dev
```

修改配置rollup.config.js

```js
import commonjs from '@rollup/plugin-commonjs';
{
	plugins: [nodeResolve(), commonjs()]    
}
```

再次进行打包：`jquery`就打包进来了

![image-20211115165329101.png](https://i.loli.net/2021/11/21/hJ62zOe1YUV5vjm.png)


看起来还不错的样子，那么加点难度，此时我使用ES6语法，我们都知道，ES6甚至更高版本，低版本浏览器下是不支持的，所以我们必须使用babel进行降级处理。

## ES6降级处理
###  @rollup/plugin-babel

> Babel默认只转换JS语法，而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法(比如`Object.assign`)都不会转码。

需要安装rollup的babel插件， babel核心包， babel预设（本质是babel插件集合）

**注意：**babel他将源代码解析为AST节点，然后修改AST节点，最后生成目标代码，每个babel插件只转义一种语法。

```js
npm install @rollup/plugin-babel @babel/core @babel/preset-env --save-dev
```

添加rollup.config.js配置

```js
import babel from '@rollup/plugin-babel';
{
    plugins: [
        nodeResolve(), 
        commonjs(),
        babel({
            babelHelpers: 'bundled',
            exclude: 'node_modules/**'
    	})
    ]
}
```

添加根目录下 .babelrc 文件

```js
{
    "presets": [
        [
          "@babel/preset-env"
        ]
      ]
  }
```

修改main.js的内容， 使用了 const语法，箭头函数，和可选链语法（方便取对象中的值），这些都是属于es6的语法

```js
// 箭头函数
const a = () => {
    const adventurer = {
        name: 'Alice',
        cat: {
          name: 'Dinah'
        }
      };
    // 可选链，获取对象的属性，没有则返回 undefined
    const dogName = adventurer.dog?.name;
    // adventurer.dog && adventurer.dog.name; 
    console.log(dogName);
}
a()
```

使用了babel以后，接着执行打包，可以看到转换为了普通函数类型， const转换成了var, 可选链也做了相应的处理

![image-20211115181436302.png](https://i.loli.net/2021/11/21/6zdh9oieWHK2X8v.png)

这一切都归功于babel及他强大的插件集，前面有说到一个插件一般只转换一种语法，@babel/preset-env 是一个插件集，所以有可能有一些语法没有，需要额外的配置对应的插件。



**babel转换时会根据指定的浏览器进行编译**

在根目录下编写`.browserslistrc`

```js
Chrome >= 80
iOS >= 11
```

babel会读取该配置，编译后的语法支持Chrome >= 80， iOS >= 11即可

再次执行打包， 发现const 和箭头函数都没有被转换，可选链 `?. `依旧被转换了

![image-20211115183655173.png](https://i.loli.net/2021/11/21/CNbKHpj6c1GOPdi.png)



还有前面提到的，babel对于一些新的API是不会编译的，测试一下,修改main.js

```js
new Promise((resolve, reject) => {
  resolve('123')
})

Object.assign({}, {'name': 'ywy'})

console.log(new Set(['1', '2', '3']))

function *ye() {
  yield 1;
}
console.log(ye().next())
```

再次打包， 可以发现Promise, Set等API是不会转义的，这时候就需要 polyfill（垫片/补丁）

![image-20211115185048890.png](https://i.loli.net/2021/11/21/BMGw2QNJO3kSvdR.png)

修改.babelrc，使用corejs3（core-js 它是JavaScript标准库的 polyfill（垫片/补丁）, 新功能的es'api'转换为大部分现代浏览器都可以支持）

useBuiltIns: 'usage', 代表按需引入，代码中使用到的语法，才会给你引入对应的polyfill

```js
{
    "presets": [
        [
          "@babel/preset-env",
          // 新增以下三行
          {
            "modules": false,
            "useBuiltIns": "usage",
            "corejs": 3
          }
        ]
    ]
}
```

![image-20211115190006827.png](https://i.loli.net/2021/11/21/Nnmw9geZzkvKdSQ.png)

最后再将所有的js压缩一下，就差不多了

## 代码压缩
### rollup-plugin-terser

```bash
npm i rollup-plugin-terser --save-dev
```

**rollup.js**

```js
import { terser } from "rollup-plugin-terser";
{
  plugins: [terser()],
};
```

执行打包：

![image-20211115230950406](D:\work\rollup\README\imgs\image-20211115230950406.png)

## 打包CSS
### rollup-plugin-postcss

[ rollup-plugin-postcss文档](https://github.com/egoist/rollup-plugin-postcss)

```bash
npm i postcss rollup-plugin-postcss -D
```

**修改rollup.config.js**

```js
import postcss from 'rollup-plugin-postcss'
module.exports = {
    input: ['main.js'],
    output: [
        {
            file: 'dist/tm.umd.js',
            format: 'umd',  
            name: 'TM',
        }
    ],
    plugins: [
        postcss()
    ]
}
```

**在根目录编写一个 `index.css`** ， 将id=app的节点字体颜色设置为红色

```css
#app {
    color: red
}
```

**修改main.js**

```js
import "./index.css";
console.log(123)
```

**编写一个html** 引入打包后的js

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>rollup测试</title>
  </head>
  <body>
    <div id="app">
      red text
    </div>
    <script src="./dist/tm.umd.js"></script>
  </body>
</html>

```

执行打包，生成了一串，js将css使用style注入到页面中。

![image-20211115223318341](D:\work\rollup\README\imgs\image-20211115223318341.png)

![image-20211115223416582](D:\work\rollup\README\imgs\image-20211115223416582.png)

**同时也可以独立出一个的css文件**

**修改rollup.config.js**

将css独立到单独的文件中

```js
    plugins: [
        postcss({
            extract: true,
            // Or with custom file name
            extract: path.resolve('dist/main.css')
        })
    ]
```

**index.html** 引入

```html
<link rel="stylesheet" href="./dist/main.css" />
```



**同时postcss默认支持less, scss，stylus 预编译语法，只是使rollup能够识别 .less, .scss, .styl 后缀的文件，而不能直接编译这些文件，编译这些文件还是需要安装对应的包。**

**less：**

```js
npm install less -D
```

**scss:**  安装好下面两个依赖即可， 但node-sass 安装起来有点费劲，还需要python环境，我一般用less

```js
npm install node-sass sass -D
```

**stylus:**

```js
npm install stylus -D
```

**le.less**

```less
@width: 20px;
@height: 20px;
#app {
    width: @width;
    height: @height;
    background-color: green;
    transform: scale(1.1);
}
```

**st.styl**

```stylus
font = 12px/16px '微软雅黑'  
$width = 16px    
a
    font font                
    width $width            
    margin-left (@width/2)  
```

**main.js**

```js
import "./index.css";
import "./le.less";
import "./st.styl";
// import "./sa.scss";

console.log(123)
```

**最终生成：**

```css
#app {
    color: red
}

#app {
  width: 20px;
  height: 20px;
  background-color: green;
}

a {
  font: 0.75px '微软雅黑';
  width: 16px;
  margin-left: 8px;
}

```

**添加css的样式前缀，再压缩代码**

**注意：** autoprefixer 也会读取 .browserslistrc 的配置， 根据目标浏览器的版本，决定是否加上前缀

```bash
# css前缀
npm i autoprefixer -D
# 压缩代码
npm i cssnano -D
```

将根目录下的 .browserslistrc 文件 IOS版本调至 7

```css
Chrome >= 80
iOS >= 7
```

**rollup.config.js**

```js
{
    plugins: [
        postcss({
            minimize: true, // 压缩代码
            plugins:[
                autoprefixer(),
                cssnano()
            ]
        }),
    ]
}
```
结果：
```css
#app{color:red}
#app{background-color:green;height:20px;-webkit-transform:scale(1.1);transform:scale(1.1);width:20px}
a{font:.75px 微软雅黑;margin-left:8px;width:16px}
```

## 
### rollup-plugin-copy

有一些资源是不需要打包的，比如图片，本地的第三方js, css, 但同时我们需要放置到 最终的打包目录下

```
npm install rollup-plugin-copy -D
```

比如有如下目录，assets/css/external 的文件复制到dist/css目录下

![image-20211118135140974.png](https://i.loli.net/2021/11/24/zuOSLp1dfrgviPD.png)

```js
import copy from 'rollup-plugin-copy'
copy({
    targets: [
        { src: ['public/*','!*.html'], dest: 'dist/'},
        { src: 'src/assets/imgs/*', dest: 'dist/imgs' },
        { src: 'src/assets/css/external/*', dest: 'dist/css' },
        { src: 'src/assets/js/external/*', dest: 'dist/js' },
    ]
})
```

到此，css与js的打包就差不多了，但是你会发现，打完包完的css与js还需要自己手动引入至html中，这很不方便，rollup更适合打包。

### rollup-plugin-bundle-html


## 开启本地服务器
### 
> 如果是构建应用程序或者说要开发浏览器中使用的包，需要实时的进行调试，那么开发服务器是很重要的。

```bash
  npm install rollup-plugin-serve --save-dev
```

## rollup的核心原理
### 为什么全局安装rollup或者其他的脚手架，就能够执行对应的命令
```
package.json
"bin": {
    "rollup": "dist/bin/rollup"
}
同时 "dist/bin/rollup" 对应的文件顶部需加上 #!/usr/bin/env node ， 表示这个文件由node执行
```

## 插件
https://www.cnblogs.com/yangzhuxian/p/13371637.html
// 插件通信
https://www.jianshu.com/p/1baefa43a4b8
### 规范
1. 插件应该有一个带有rollup-plugin-前缀的明确名称。
2. package.json中包含rollup-plugin关键字。
3. 应该测试插件。我们推荐mocha或ava，它们支持开箱即用的 promise。
4. 尽可能使用异步方法。
5. 用英语记录您的插件。
6. 如果合适，请确保您的插件输出正确的源映射。
7. 如果您的插件使用“虚拟模块”（例如用于辅助函数），请在模块 ID 前加上\0. 这可以防止其他插件尝试处理它。

# rollup的一些资料
[官方文档](https://rollupjs.org/guide/en/#overview)
[rollup/awesome 插件集](https://github.com/rollup/awesome)

[Rollup Plugins](https://github.com/rollup/plugins)

https://juejin.cn/post/6844904058394771470#heading-16
https://juejin.cn/post/6934698510436859912#heading-5
https://rollupjs.org/guide/en/#overview
https://github.com/rollup/awesome
https://juejin.cn/post/6844903731343933453#heading-21
https://zhuanlan.zhihu.com/p/95119407
https://juejin.cn/post/6962088402174869517#heading-4
https://segmentfault.com/a/1190000010628352