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

## 集成editorconfig配置

EditorConfig 有助于为不同 IDE 编辑器上处理同一项目的多个开发人员维护一致的编码风格。

```yaml
# http://editorconfig.org

root = true

[*] # 表示所有文件适用
charset = utf-8 # 设置文件字符集为 utf-8
indent_style = space # 缩进风格（tab | space）
indent_size = 2 # 缩进大小
end_of_line = lf # 控制换行类型(lf | cr | crlf)
trim_trailing_whitespace = true # 去除行首的任意空白字符
insert_final_newline = true # 始终在文件末尾插入一个新行

[*.md] # 表示仅 md 文件适用以下规则
max_line_length = off
trim_trailing_whitespace = false
```


VSCode需要安装一个插件：EditorConfig for VS Code
![image-20210722215138665](https://tva1.sinaimg.cn/large/008i3skNgy1gsq2gh989yj30pj05ggmb.jpg)

## ESLint
1. 主流的JS lint工具检测JS代码质量
2. ESlint很容易同一开发者的编码风格
3. ESlint可以帮助开发者提升编码能力

### VSCode安装ESLint插件
![image-20210722215933360](https://tva1.sinaimg.cn/large/008i3skNgy1gsq2oq26odj30pw05faaq.jpg)

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
> Prettier 是一款强大的代码格式化工具，支持 JavaScript、TypeScript、CSS、SCSS、Less、JSX、Angular、Vue、GraphQL、JSON、Markdown 等语言，基本上前端能用到的文件格式它都可以搞定，是当下最流行的代码格式化工具。
安装：
1.安装prettier

```shell
npm install prettier -D
```

2.配置.prettierrc文件：

* useTabs：使用tab缩进还是空格缩进，选择false；
* tabWidth：tab是空格的情况下，是几个空格，选择2个；
* printWidth：当行字符的长度，推荐80，也有人喜欢100或者120；
* singleQuote：使用单引号还是双引号，选择true，使用单引号；
* trailingComma：在多行输入的尾逗号是否添加，设置为 `none`；
* semi：语句末尾是否要加分号，默认值true，选择false表示不加；

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": true,
  "trailingComma": "none",
  "semi": false
}
```



3.创建.prettierignore忽略文件

```
/dist/*
.local
.output.js
/node_modules/**

**/*.svg
**/*.sh

/public/*
```

4.VSCode需要安装prettier的插件

![image-20210722214543454](https://tva1.sinaimg.cn/large/008i3skNgy1gsq2acx21rj30ow057mxp.jpg)

5.测试prettier是否生效

* 测试一：在代码中保存代码；
* 测试二：配置一次性修改的命令；

在package.json中配置一个scripts：

```json
    "prettier": "prettier --write ."
```

**解决eslint和prettier冲突的问题：**

安装插件：（vue在创建项目时，如果选择prettier，那么这两个插件会自动安装）

```shell
npm i eslint-plugin-prettier eslint-config-prettier -D
```

添加prettier插件：

```json
  extends: [
    "plugin:vue/vue3-essential",
    "eslint:recommended",
    "@vue/typescript/recommended",
    "@vue/prettier",
    "@vue/prettier/@typescript-eslint",
    'plugin:prettier/recommended'
  ],
```

## husky

虽然我们已经要求项目使用eslint了，但是不能保证组员提交代码之前都将eslint中的问题解决掉了：

* 也就是我们希望保证代码仓库中的代码都是符合eslint规范的；

* 那么我们需要在组员执行 `git commit ` 命令的时候对其进行校验，如果不符合eslint规范，那么自动通过规范进行修复；

那么如何做到这一点呢？可以通过Husky工具：

* husky是一个git hook工具，可以帮助我们触发git提交的各个阶段：pre-commit、commit-msg、pre-push

如何使用husky呢？

这里我们可以使用自动配置命令：

```shell
npx husky-init && npm install
```

这里会做三件事：

1.安装husky相关的依赖：

![image-20210723112648927](https://tva1.sinaimg.cn/large/008i3skNgy1gsqq0o5jxmj30bb04qwen.jpg)

2.在项目目录下创建 `.husky` 文件夹：

```
npx huksy install
```

![image-20210723112719634](https://tva1.sinaimg.cn/large/008i3skNgy1gsqq16zo75j307703mt8m.jpg)

3.在package.json中添加一个脚本：

![image-20210723112817691](https://tva1.sinaimg.cn/large/008i3skNgy1gsqq26phpxj30dj06fgm3.jpg)

接下来，我们需要去完成一个操作：在进行commit时，执行lint脚本：

![image-20210723112932943](https://tva1.sinaimg.cn/large/008i3skNgy1gsqq3hn229j30nf04z74q.jpg)


这个时候我们执行git commit的时候会自动对代码进行lint校验。

## lint-staged
```bash
npm i lint-staged -D
```
## git commit规范
### 代码提交风格

通常我们的git commit会按照

通常我们的git commit会按照统一的风格来提交，这样可以快速定位每次提交的内容，方便之后对版本进行控制。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsqw17gaqjj30to0cj3zp.jpg)

但是如果每次手动来编写这些是比较麻烦的事情，我们可以使用一个工具：Commitizen

* Commitizen 是一个帮助我们编写规范 commit message 的工具；

1.安装Commitizen

```shell
npm install commitizen -D
```

2.安装cz-conventional-changelog，并且初始化cz-conventional-changelog：

```shell
npx commitizen init cz-conventional-changelog --save-dev --save-exact
```

这个命令会帮助我们安装cz-conventional-changelog：

![image-20210723145249096](https://tva1.sinaimg.cn/large/008i3skNgy1gsqvz2odi4j30ek00zmx2.jpg)

并且在package.json中进行配置：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsqvzftay5j30iu04k74d.jpg)

这个时候我们提交代码需要使用 `npx cz`：

* 第一步是选择type，本次更新的类型

| Type     | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| feat     | 新增特性 (feature)                                           |
| fix      | 修复 Bug(bug fix)                                            |
| docs     | 修改文档 (documentation)                                     |
| style    | 代码格式修改(white-space, formatting, missing semi colons, etc) |
| refactor | 代码重构(refactor)                                           |
| perf     | 改善性能(A code change that improves performance)            |
| test     | 测试(when adding missing tests)                              |
| build    | 变更项目构建或外部依赖（例如 scopes: webpack、gulp、npm 等） |
| ci       | 更改持续集成软件的配置文件和 package 中的 scripts 命令，例如 scopes: Travis, Circle 等 |
| chore    | 变更构建流程或辅助工具(比如更改测试环境)                     |
| revert   | 代码回退                                                     |

* 第二步选择本次修改的范围（作用域）

![image-20210723150147510](https://tva1.sinaimg.cn/large/008i3skNgy1gsqw8ca15oj30r600wmx4.jpg)

* 第三步选择提交的信息

![image-20210723150204780](https://tva1.sinaimg.cn/large/008i3skNgy1gsqw8mq3zlj60ni01hmx402.jpg)

* 第四步提交详细的描述信息

![image-20210723150223287](https://tva1.sinaimg.cn/large/008i3skNgy1gsqw8y05bjj30kt01fjrb.jpg)

* 第五步是否是一次重大的更改

![image-20210723150322122](https://tva1.sinaimg.cn/large/008i3skNgy1gsqw9z5vbij30bm00q744.jpg)

* 第六步是否影响某个open issue

![image-20210723150407822](https://tva1.sinaimg.cn/large/008i3skNgy1gsqwar8xp1j30fq00ya9x.jpg)

我们也可以在scripts中构建一个命令来执行 cz：

![image-20210723150526211](https://tva1.sinaimg.cn/large/008i3skNgy1gsqwc4gtkxj30e207174t.jpg)



### 代码提交验证

如果我们按照cz来规范了提交风格，但是依然有同事通过 `git commit` 按照不规范的格式提交应该怎么办呢？

* 我们可以通过commitlint来限制提交；

1.安装 @commitlint/config-conventional 和 @commitlint/cli

```shell
npm i @commitlint/config-conventional @commitlint/cli -D
```

2.在根目录创建commitlint.config.js文件，配置commitlint

```js
module.exports = {
  extends: ['@commitlint/config-conventional']
}
```

3.使用husky生成commit-msg文件，验证提交信息：

```shell
npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
```
