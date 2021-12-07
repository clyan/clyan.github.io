## 为什么要有规范化标准
1. 软件开发需要多人协助
2. 不同开发者具有不同编码习惯和喜好
3. 不同喜好增加项目维护成本
4. 每个项目或者团队需要明确同一的标准

## 哪些地方需要规范化标准
1. 代码、文档、甚至是提交日志
2. 开发过程中认为编写的成果物
3. 代码标准化规范最为重要

## 实施规范化的方法
1. 编码前人为的标准约定
2. 通过工具实现Lint

## 常用的规范化实现方式
1. eslint工具使用
2. 定制ESlint 校验规则
3. eslint 对 typescript 的支持
4. eslint 结合四动画工具或者Webpack
5. 基于ESlint的衍生工具
6. Stylelint工具的使用

## ESLint
1. 主流的JS lint工具检测JS代码质量
2. ESlint很容易同一开发者的编码风格
3. ESlint可以帮助开发者提升编码能力

### 入门使用
1. 初始化项目
```bash
npm init -y
```
2. 安装eslint模块为开发依赖
```bash
npm install eslint -D
```
3. 通过cli命令验证安装结果
```bash
npx eslint --version
```

初始化项目

在根目录新建`1-prepare.js`
```js
const foo = 123;

function fn() {
    console.log("hellow")
        console.log("eslint")
}
fn1()
```
```bash
npx eslint --init
```
根据命令行提示。选择对应的选项，生成你需要的eslint规则

默认安装最新版本

![](https://s2.loli.net/2021/12/04/ZfQibwNSd785sYW.png)

![](https://s2.loli.net/2021/12/04/WwfGQjVKpuCirJc.png)

执行
```
npx eslint ./1-prepare.js
```
提示：
![](https://s2.loli.net/2021/12/04/QensqROLfl8xZkP.png)

将`.eslintrc.js` 中 ` ecmaVersion: 13` 改为 `12` 再次执行

![](https://s2.loli.net/2021/12/04/cM9nq6URjkvxADY.png)

执行命令时加上 `--fix` 可以修复大部分格式错误，但是无法自动修复语法错误

![](https://s2.loli.net/2021/12/04/arP2ilC7hwWyEVg.png)

### ESlint 配置文件
先新建一个测试文件`2-configuration.js` 
```js
document.getElementById('name')
```

`.eslintrc.js`
```js
module.exports = {
  // 标记环境信息，代码运行在的环境，如browser中存在window, node中不存在
  env: {
    browser: true,
    es2021: true
  },
  // 使用的eslint规范，可以在node_modules中找到
  extends: [
    'standard'
  ],
  parserOptions: {
    ecmaVersion: 12
  },
  rules: {
  }
}

```
#### env 

表示代码运行环境， `eslint`根据环境判断哪些全局`api`是可用的

可以用到的环境及说明, **注意：** 这些环境不是互斥的，可以同时配置多个为`true`
![](https://s2.loli.net/2021/12/04/WArwbVfFne6HNT7.png)


此时将`browser` 改为 `false`, 按常理来说，执行一下命令会包错，`document`属于`browser` , 但是并不会报错
```bash
npx elsint ./2-configuration.js
```
原因如下: 在 `extends`中以来了 `standard` 配置， 其中大量的规则

![](https://s2.loli.net/2021/12/04/D6FnTqZWGYvBR4e.png)

`globals` 中将`document` 、 `navigator` 、`window` 都设置了全局可读的变量,

修改`2-configuration.js`
```js
alert('123')
```
![](https://s2.loli.net/2021/12/04/JAvCeFTjRlLuIkY.png)

#### extends

> 继承共享配置，多个项目之间共享一个eslint配置就可以定义一个公共的配置文件，继承即可，可以继承多个

#### parserOptions

> 影响 esma (js)  版本的语法检测，不代表某个成员是否可用， 某个成员是否可用还是env中的配置决定的

修改`2-configuration.js`
```js
const foo = 1
```
并将`ecmaVersion` 改为 `5`, 执行`npx elsint ./2-configuration.js`，
![](https://s2.loli.net/2021/12/04/Hh3vpWa57YSE8rG.png)

报错，这是因为在`es5`中不能使用模块化

将`node_modules`下的`eslint-config-standard` 包中`sourceType`修改为`script`

![](https://s2.loli.net/2021/12/04/izfEZyHWUdxrbc2.png)

再次执行，既可以看到 `const`在`es5`下是不被支持的

![](https://s2.loli.net/2021/12/04/tPjRBm12u9yc6hF.png)

#### rules
> 可以使用的lint规则

"off" 或 0 - 关闭规则

"warn" 或 1 - 开启规则，使用警告级别的错误：warn (不会导致程序退出)

"error" 或 2 - 开启规则，使用错误级别的错误：error (当被触发的时候，程序会退出)

#### globals
> 在最新的配置中并没有默认体现， 用于额外的配置可以使用的全局成员

如修改`2-configuration.js`
```js
JQuery('123')
```
执行`npx elsint ./2-configuration.js` 报错：

![](https://s2.loli.net/2021/12/04/adH3u68yDTLXoeK.png)

在`.eslintrc.js` 中添加`globals` 配置, 再执行`npx elsint ./2-configuration.js`,就不会有错误了
```js
  globals: {
    'jQuery': 'readonly'
  }
```

### ESlint 配置注释
> 实际开发中难免遇到一两个必须要违反规则的情况，不能因为这一两个点就去推翻校验规则的配置
> 可以使用`配置注释` 在代码内单独的取消应用规则

**注意：** 将配置文件以上的修改还原至初始状态

新建`3-configuration-comment.js` 案例文件
```js
const str1 = '${name} is a coder'
console.log(str1)
```
![](https://s2.loli.net/2021/12/04/MpBAmk7EbTy4iF8.png)

```js
const str1 = '${name} is a coder' // eslint-disable-line no-template-curly-in-string
```

不仅能够禁用某个规则，也能修改某个变量的规则, [Configuring Rules](http://eslint.cn/docs/user-guide/configuring#configuring-rules)

### 结合gulp使用
> 在gulp案例中，添加eslint, 在打包前进行代码检查
安装： `npx eslint --init`, `npm i gulp-eslint -D`
```js
import eslint from 'gulp-eslint'
const script = () => {
  return src('src/assets/scripts/*.js', { base: 'src' })
    .pipe(eslint())
    .pipe(eslint.format()) //格式化格式
    .pipe(eslint.failAfterError()) //格式错误，终止运行
    .pipe(babel({ presets: ['@babel/preset-env'] })) //注意传入preset-env进行转换
    .pipe(dest('dist'))
}
module.exports = {
  script
}
```
执行`gulp script` 会报错, 需要修改版本 `eslint-config-standard`版本， [错误原因及解决办法](https://www.codeleading.com/article/22415009619/)

```
npm i eslint-config-standard@14.1.1 eslint-plugin-standard --dev
```

### 结合webpack使用
除了像上面一样初始化eslint配置外

安装`eslint-loader` 并在webpack配置文件中加上eslint-loading并设置`enfore: true`(表示先执行这个loader进行处理)
![](https://s2.loli.net/2021/12/04/DJY4FwGQktpnVrU.png)

如果是react项目, 安装`eslint-plugin-react` 并在`.eslintrc`中添加如下配置

```
extends: [
  'plugin:react/recommended'
]
```

### 现代化项目集成ESlint
> vue和react 的项目都已经集成eslint

### ESlint检查Ts

以前使用`tslint`进行校验,现在推荐`eslint` + `ts插件`对ts代码校验

创建项目时，选择ts即可
![](https://s2.loli.net/2021/12/04/zitGSLKAF5xQqNv.png)

**parser:** 指定语法解析器
**plugins：** 使用`@typescript-eslint`
```js
module.exports = {
  env: {
    browser: true,
    es2021: true
  },
  extends: [
    'standard'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module'
  },
  plugins: [
    '@typescript-eslint'
  ],
  rules: {
  }
}
```

## StyleLint
> 检测css格式, [stylelint](https://stylelint.io/user-guide/get-started/)
```bash
npm i stylelint -D
npm i stylelint-config-standard -D
```
在根目录下新建`.stylelintrc.js` 

```js
module.exports = {
    extends: "stylelint-config-standard"
}
```
```bash
npx stylelint ./index.css
```



校验`scss`代码

```bash
npm i  stylelint-config-sass-guidelines -D
```

修改`.stylelintrc.js` 
```js
module.exports = {
    extends: [
        "stylelint-config-standard",
        "stylelint-config-standard-scss"
    ]
}
```

```bash
npx stylelint ./index.scss
```

## prettier
> 自动格式化代码
安装：
```bash
npm i prettier -D
```
`.` 目录下所有文件， `--write` 格式化后，替换源文件
```bash
npx prettier .  --write 
```

## husky

```
npm install husky -D

```
执行`npx husky-init`

![](https://s2.loli.net/2021/12/04/R6GqdPlcbEWe2Lk.png)

**pre-commit**
```js
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm test
```

运行git commit时就会执行 `npm test` , 修改对应的命令即可

## lint-staged
```bash
npm i lint-staged -D
```
