## rollup是什么

`Rollup`是`JavaScript`的一个模块绑定器，它将一小段代码编译整合成一个整体，比如库或应用程序

默认只识别`ES`模块化和支持导入`js`文件，如需要导入其他模块如`commonJS`或者其他文件如`json` 文件需要额外的插件支持，告诉`rollup`如何处理。 

## rollup与webpack的使用场景

 **项目案例:**

  **使用rollup：**  `Vue、React、D3、Three.js、Moment` 等等

  **使用webpack:**  `elementUI、 Ant Design、Vant`

  **大多数情况下， 对应用程序使用 `webpack`，对纯js库使用 `Rollup`**
  
  **如果要书写大量的css,请不要使用webpack**

[Webpack 和 Rollup：相同但不同](https://medium.com/webpack/webpack-and-rollup-the-same-but-different-a41ad427058c)

> 可以将常用的工具函数打包到js中发布npm包，如判断设备类型，生成随机数等等。
## 安装

**全局安装：**

```
npm install -g rollup
```

**项目安装：**

```
npm install -D rollup
```

如果是团队项目，尽可能的在项目内安装， 因为这样可以保持版本一致，也不用团队中每个人都单独的进行一次全局安装

## 使用

使用`rollup`有两种使用形式，一种是常规的使用配置文件，一种是进阶使用`jsAPI` （推荐，更具灵活性）

### 配置文件

默认在根目录下创建`rollup.config.js`

```js
export default {
  input: 'src/main.js',	// 文件解析入口
  output: {
    file: 'bundle.js', // 生成的路径与文件名
    format: 'cjs'		// 文件的模块形式
  }
};
```

最基本的需要提供`input`和`output`代表文件解析的入口和最终生成文件的位置与文件格式

**注意：** 配置文件使用的是`ESModule` 这样有助于在你的代码中方便引入配置文件信息, `rollup`内部会将其转化成`CommonJS`

也可以将其命名为`rollup.config.cjs` 可以使rollup省略文件的编译

也可以使用ts的文件作为配置文件，需要安装 `@rollup/plugin-typescript`,并指定`--configPlugin` 为 `typescript`

```
rollup -c rollup.config.ts --configPlugin typescript
```

如果不使用ts时，推荐使用`defineConfig` 定义配置文件对象，因为这样会在你书写配置时，编辑器会有自动补全的提示

```js
import { defineConfig } from 'rollup';
export default defineConfig({
  input: 'src/main.js',	// 文件解析入口
  output: {
    file: 'bundle.js', // 生成的路径与文件名
    format: 'cjs'		// 文件的模块形式
  }
});
```

### JSAPI

新建一个build.js, 使用 `rollup.rollup` 

```js
const rollup = require('rollup');
// [input的配置](https://rollupjs.org/guide/en/#inputoptions-object)
const inputOptions = {};
// [output的配置](https://rollupjs.org/guide/en/#outputoptions-object)
const outputOptions = {};
async function build() {
   // 构建阶段
  const bundle = await rollup.rollup(inputOptions);
  
   // 生成阶段
  const generate = await bundle.generate(outputOptions);

  // 输出阶段
  await bundle.write(outputOptions);
  // 完成
  await bundle.close();
}
build();
```

并执行该文件

```
node build.js
```

可以达到与使用配置文件同样的效果，但JSAPI能够更灵活的修改`input`,`output` 等

## 教程
简介中有提到，`rollup`仅仅是一个模块打包器，且只识别js文件，拓展更多的功能需要更多的插件，接下来一步步使用
1. **初始化项目**
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

**执行结果：** 在`dist`目录下生成`5`个不同格式的文件，以`umd`格式为例，可以看到只打包了`a` , 因为`b`没有使用到，也就是`rollup`原生支持的`tree-shaking`特性，

**注意：** `tree-shaking`是由`ESModule`的静态编译的特性所支持的

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

控制台输出一堆警告，这是由于`rollup`默认识别相对路径`'./'`的形式，对于导入第三方包，默认认为是外部依赖

![image-20211115162035840.png](https://i.loli.net/2021/11/21/2HWdGXE8TNBoJvA.png)

修改`rollup.config.js`配置文件，添加`external`  (告诉`rollup`不要将此`jquery`打包，而是作为外部依赖)，并在 `umd` 与 `iife` 配置下添加 ` globals： { 'jquery': '$'}` (告诉`rollup` 全局变量`$`即是`jquery`)

![image-20211115162720092.png](https://i.loli.net/2021/11/21/lrURo8YVTMgEe5L.png)

这个时候`jquery`，就并不会打包到最终的`bundle`中了

![image-20211115165426886.png](https://i.loli.net/2021/11/21/MbuRznP2EgUpZO1.png)



### @rollup/plugin-node-resolve

**如果想`jquery`打包进去呢，并且不报错就需要安装`@rollup/plugin-node-resolve`插件，告诉 `Rollup` 如何找到它**

[node-resolve参数配置](https://github.com/rollup/plugins/tree/master/packages/node-resolve)

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

**报错：** 原因是`rollup`只支持 `esm` 引入模块的方式，我们打开`node_modules`下`jquery`包，看一下为什么

![image-20211115164539917.png](https://i.loli.net/2021/11/21/mATB6ps1zFt2gcG.png)

我们可以看到`jquery`只支持`commonJS`与`IIFE`格式，所以此时我们需要另外的插件将`commonJS`转换为`ESM`

### @rollup/plugin-commonjs

```bash
npm install @rollup/plugin-commonjs --save-dev
```

修改配置`rollup.config.js`

```js
import commonjs from '@rollup/plugin-commonjs';
{
	plugins: [nodeResolve(), commonjs()]    
}
```

再次进行打包：`jquery`就打包进来了

![image-20211115165329101.png](https://i.loli.net/2021/11/21/hJ62zOe1YUV5vjm.png)

## ES6降级处理

此时我使用ES6语法，我们都知道，ES6甚至更高版本，低版本浏览器下是不支持的，所以我们必须使用babel进行降级处理。

###  @rollup/plugin-babel

> Babel默认只转换JS语法，而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法(比如`Object.assign`)都不会转码。

需要安装`rollup`的`babel插件`， `babel核心包`， `babel预设`（本质是babel插件集合）

**注意：** `babel`将源代码解析为`AST`节点，然后修改`AST`节点，最后生成目标代码，每个`babel`插件即是一个函数且只转义一种语法。

```js
npm install @rollup/plugin-babel @babel/core @babel/preset-env --save-dev
```

添加`rollup.config.js`配置

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

添加根目录下 `.babelrc` 文件

```js
{
    "presets": [
        [
          "@babel/preset-env"
        ]
      ]
  }
```

修改`main.js`的内容， 使用了 `const` 语法，箭头函数，和可选链语法（方便取对象中的值），这些都是属于es6以上版本的语法

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

使用了`babel`以后，接着执行打包，可以看到转换为了普通函数类型， `const`转换成了`var`, `可选链`也做了相应的处理

![image-20211115181436302.png](https://i.loli.net/2021/11/21/6zdh9oieWHK2X8v.png)

这一切都归功于`babel`及它强大的插件集，前面有说到一个插件一般只转换一种语法，@babel/preset-env 是一个插件集，所以有可能有一些语法没有，需要额外的配置对应的插件。



**babel转换时会根据指定的浏览器进行编译**

在根目录下编写`.browserslistrc`

```js
Chrome >= 80
iOS >= 11
```

`babel`会读取该配置，告诉`babel`编译后的语法只需要支持Chrome >= 80， iOS >= 11即可

再次执行打包， 发现`const` 和`箭头函数`都没有被转换，`可选链 ?. `依旧被转换了, 因为 `Chrome >= 80， iOS >= 11` 版本已经支持`const` 和 `箭头函数`,  `可选链 ?. `依旧不支持

![image-20211115183655173.png](https://i.loli.net/2021/11/21/CNbKHpj6c1GOPdi.png)



还有前面提到的，`babel`对于一些新的API是不会编译的，测试一下,修改`main.js`

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

再次打包， 可以发现`Promise`, `Set`等`API`是不会转义的，这时候就需要 `polyfill`（垫片/补丁）

![image-20211115185048890.png](https://i.loli.net/2021/11/21/BMGw2QNJO3kSvdR.png)

修改`.babelrc`，使用`corejs3`（`core-js` 它是`JavaScript`标准库的 `polyfill（垫片/补丁）`, 新功能的`es'api'`转换为大部分现代浏览器都可以支持）

`useBuiltIns: 'usage'`, 代表按需引入，代码中使用到的语法，才会给你引入对应的polyfill

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



## 代码压缩
去除空格、替换命名，将代码压缩成一行，减少体积
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

![image.png](https://i.loli.net/2021/11/27/FuR7mKfT4kMybtV.png)

## 样式处理(css, less, sass, stylus)
> 开发js库一般是不写css，如果要写大量的css，那可能你开发的是项目，优先选择webpack，webpack里也有开发library的配置。
> 
> 如果你的js类库里还是必不可少要写些css的话，也可采用如下插件对css进行处理


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

执行打包，生成了一串js将css使用style注入到页面中。

![image.png](https://i.loli.net/2021/11/27/VbZOXTtwIMKAcJ3.png)

**效果：**

![image.png](https://i.loli.net/2021/11/27/5dAZwCvq6rLIO47.png)

**同时也可以独立出一个的css文件**

**修改rollup.config.js**

将css独立到单独的文件中

```js
import path from 'path'; // path是node的内置模块用于处理路径
  plugins: [
      postcss({
          extract: true,
          // Or with custom file name
          extract: path.resolve('dist/main.css')
      })
  ]
```

![image.png](https://i.loli.net/2021/11/27/VUQacRwCNHMutP1.png)

**index.html** 引入

```html
<link rel="stylesheet" href="./dist/main.css" />
```



**同时postcss默认支持less, scss，stylus 预编译语法，注意postcss只是使rollup能够识别 .less, .scss, .styl 后缀的文件，而不能直接编译这些文件，编译这些文件还是需要安装对应的包。**

**less：**

```js
npm install less -D
```

**scss:**  安装好下面两个依赖即可，**注意：**安装`node-sass`可能会报错，需要`python`环境支持，如果遇到，则安装`python`环境并配置`电脑环境变量`

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

#### 添加css前缀，压缩

**注意：** `autoprefixer` 也会读取 `.browserslistrc` 的配置， 根据目标浏览器的版本，决定是否加上前缀

```bash
# css前缀
npm i autoprefixer -D
```

将根目录下的 `.browserslistrc` 文件 `IOS`版本调至 `7`

```css
Chrome >= 80
iOS >= 7
```

**rollup.config.js**

```js
import autoprefixer from 'autoprefixer'
{
    plugins: [
        postcss({
            minimize: true, // 压缩代码
            plugins:[
                autoprefixer(),
            ]
        }),
    ]
}
```
结果：

![image.png](https://i.loli.net/2021/11/27/BHkLuVYP2aDAK1t.png)

## 复制静态资源
### rollup-plugin-copy

如果是构建应用程序，有一些资源是不需要打包的，比如图片，本地的第三方js, css, 但同时我们需要放置到最终的打包目录下

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


## 实时打包
使用rollup的 watch模式即可, 加上`-w` 参数，保存文件时会自动重新打包

```bash
  "scripts": {
      "dev": "rollup -cw"
  }
```
## 本地开发服务器与热更新

###  rollup-plugin-serve
> 在实际开发过程中，使用本地服务器查看页面，调试代码，(也就是使用http://xxxx:8888的形式去访问项目，而不是直接打开本地文件，如D://index.html这样的形式访问)。
###  rollup-plugin-livereload
>  每次修改代码，不需要重新启动项目即可立即生效
```bash
  npm install rollup-plugin-serve -D
  npm install rollup-plugin-livereload -D
```
**rollup.config.js**
```js
import serve from 'rollup-plugin-serve';
import livereload from 'rollup-plugin-livereload';
{
  plugins: [
      livereload(),
      serve({
          // open: true,  
          port: 8888,
          contentBase: '', //以根目录开启本地服务，默认访问根目录的index.html
      }),
  ],
}
```

## rollup的核心原理
### 如何实现一个插件
> 在实现插件前，我们必须弄懂rollup插件的运行机制

我们经常使用的Vue或者react框架都是有生命周期的概念的，而对于rollup而言同样有，插件不同的钩子运行在不同的阶段，多个插件实现相同的钩子也有顺序，

就好比，Vue中父子组件，兄弟组件，他们的不同的钩子运行时机都是有差异的

### Rollup插件钩子
rollup分为2个阶段，build, output, 每个阶段都有很多钩子
####  钩子的类型
1. parallel(并行的)： 如果有几个插件实现了这个钩子，那么所有的插件都将按照指定的插件顺序运行。如果一个钩子是异步的，这类钩子的后续钩子将并行运行，而不等待当前钩子。
2. sequential(顺序的)： 如果有几个插件实现了这个钩子，那么所有的插件都将按照指定的插件顺序运行。如果钩子是异步的，后续此类钩子将等待直到当前钩子解析完毕。
3. first： 如果有多个插件实现了这个钩子，那么这些钩子将依次运行，直到钩子返回一个非null或undefined的值为止。
4. async(异步的)： 钩子也可以返回一个解析为相同类型值的promise; 否则，钩子被标记为同步
5. sync(同步的)
   
**构建阶段钩子**

以下图片引自官方文档：

![image.png](https://i.loli.net/2021/11/28/usiA5wHbUtBWLrG.png)

怎么理解上述钩子？ 什么并行的，顺序的？ 通过一个简单的案例描述一下

从上图我们可以看到`resolveId` 是一个 `async, first` 的钩子

**初始化项目目录如下：**

![image.png](https://i.loli.net/2021/11/28/5KdulyqTcg4ftvo.png)

`ex1插件`返回 `a.js` 告诉`rollup`, 所有的引入都视为引入`a.js`，同时`ex1插件`的`resolveId` 钩子返回了非`null| underfined` 所以`ex2 插件`的`resolveId` 钩子不会再执行

`ex2插件`的load依旧不受影响，输出 `ex2 load`

![image.png](https://i.loli.net/2021/11/28/A2LqRg8x1rUSKjz.png)


### 插件规范
> 引自官方文档

1. 插件应该有一个带有rollup-plugin-前缀的明确名称。
2. package.json中包含rollup-plugin关键字。
3. 应该测试插件。我们推荐mocha或ava，它们支持开箱即用的 promise。
4. 尽可能使用异步方法。
5. 用英语记录您的插件。
6. 如果合适，请确保您的插件输出正确的源映射。
7. 如果您的插件使用“虚拟模块”（例如用于辅助函数），请在模块 ID 前加上\0. 这可以防止其他插件尝试处理它。




### 实现一个支持引入`.png` 的插件
```js
const fs = require('fs');
const path = require('path');
function pngResolverPlugin() {
    return {
      name: 'png-resolver',
      resolveId(source, importer) {
        if (source.endsWith('.png')) {
          console.log("resolveId", path.resolve(path.dirname(importer), source))
          // 返回图片的真实路径 ： D:\Desktop\rollup\rollup-06\src\grass.png
          return path.resolve(path.dirname(importer), source);
        }
      },
      load(id) {
        if (id.endsWith('.png')) {
          // id 就是resolveId钩子赶回的路径 如： D:\Desktop\rollup\rollup-06\src\grass.png
          // 使用rollup的 emitFile, 并指定 source 使用 fs.readFileSync 读取文件
          const referenceId = this.emitFile({
            type: 'asset',
            name: path.basename(id),
            source: fs.readFileSync(id)
          });
          console.log("id", id)
          // 导出 import.meta.ROLLUP_FILE_URL_referenceId
          return `export default import.meta.ROLLUP_FILE_URL_${referenceId};`;
        }
      }
    };
  }
module.exports =  pngResolverPlugin
```

![image.png](https://i.loli.net/2021/11/28/DyunFfqRX5pJx2e.png)
### 为什么全局安装rollup或者其他的脚手架，就能够执行对应的命令
```
package.json
"bin": {
    "rollup": "dist/bin/rollup"
}
同时 "dist/bin/rollup" 对应的文件顶部需加上 #!/usr/bin/env node ， 表示这个文件由node执行
```


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