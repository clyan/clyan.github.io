## 前言
> vue3 一开始采用了yarn 与 lerna结合的形式进行包管理，现在看到vue3源码已经没有lerna了，通过yarn和自写脚本进行管理。
> monorepo 组织代码 ，在最外层提供通用配置，如提供ts 及 rollup配置等，我们需要为每个packages下的包进行打包和提供入口，生成dist目录与index.js入口。
> 最新的使用pnpm进行包管理了~~~

## vue3目录结构

先贴代码：github： [vue-next](https://link.zhihu.com/?target=https%3A//github1s.com/vuejs/vue-next) [github1s在线查看地址](https://link.zhihu.com/?target=https%3A//github1s.com/vuejs/vue-next)

> tips: 在github地址中添加1s,打开在线编辑器，方便浏览代码，
> 如：https://github.com/vuejs/vue-next/加上1s : https://github1s.com/vuejs/vue-next/

首先查看vue3的总体文件目录，其中对于本文重点的目录或文件 scripts目录, rollup.config.js, api-extractor.json

```txt
 vue3
 ├── packages        # 所有包（此目录只保持一部分包）
 │   ├── compiler-core           # 编译核心包 
 │   │   ├── api-extractor.json  # 用于合并.d.ts, api-extractor
 │   │   ├── src                 # 包主要开发目录
 │   │   ├── index.js            # 包入口，导出的都是dist目录的文件
 │   │   ├── LICENSE             # 开源协议文件
 │   │   ├── package.json        
 │   │   ├── README.md           # 包描述文件
 │   │   └── __tests__           # 包测试文件
 ├── scripts                      # 一些工程化的脚本，本文重点
 │   ├── bootstrap.js            # 用于生成最小化的子包
 │   ├── build.js                # 用于打包所有packages下的包
 │   ├── checkYarn.js            # 检查是否是yarn进行安装
 │   ├── dev.js                  # 监听模式开发
 │   ├── release.js              # 用于发布版本
 │   ├── setupJestEnv.ts         # 设置Jest的环境
 │   ├── utils.js                # 公用的函数包
 │   └── verifyCommit.js         # git提交验证message
 ├── test-dts                     # 验证类型声明
 │   ├── component.test-d.ts
 |   ├── .....-d.ts
 ├── api-extractor.json          # 用于合并.d.ts
 ├── CHANGELOG.md                # 版本变更日志
 ├── jest.config.js              # jest测试配置
 ├── LICENSE
 ├── package.json
 ├── README.md
 ├── rollup.config.js            # rollup配置
 ├── tsconfig.json               # ts配置
 └── yarn.lock                   # yarn锁定版本文件
```

## yarn 与 workspace

**workspace的作用：**

能帮助你更好地管理多个子project的repo，这样你可以在每个子project里使用独立的package.json管理你的依赖，

又不用分别进到每一个子project里去yarn install/upfrade安装/升级依赖，而是使用一条yarn命令去处理所有依赖就像只有一个package.json一样 

yarn会根据就依赖关系帮助你分析所有子project的共用依赖，保证所有的project公用的依赖只会被下载和安装一次。

workspace的使用 yarn workspace并不需要安装什么其他的包，只需要简单的更改package.json便可以工作。 

首先我们需要确定workspace root，一般来说workspace root都会是repo的根目

根目录下package.json添加如下配置

```json
{
   "private": true,
   "workspaces": [
     "packages/*"  // 指定子目录，可指定多个
   ],
}
```

给根项目安装依赖（一般是公用的开发工具）

```bash
 yarn add -W -D rollup typescript jest prettier
```

给package安装外部npm包

```bash
 yarn workspace @vue/template-explorer add monaco-editor 
```

给package安装内部npm包

```bash
 yarn workspace @vue/runtime-core add @vue/reactivity@3.0.0-alpha.1
```

## api-extractor

API Extractor是一个TypeScript分析工具，可以产生三种不同的输出类型

1. API Report : API Extractor可以从您的项目的主入口点跟踪所有的导出，并生成一个报告，作为API审查工作流的基础。
2. .d.ts Rollups : 就像Webpack可以把你所有的JavaScript文件“卷”到一个单独的bundle中来分发一样，API Extractor也可以把你的TypeScript声明卷到一个单独的.d.ts文件。
3. API Documentation ：API Extractor可以为您的每个项目生成一个“文档模型”JSON文件。这个JSON文件包含提取的类型签名和文档注释。API -document配套工具可以使用这些文件来生成API参考网站，也可以将它们用作自定义文档管道的输入。

![](https://pic1.zhimg.com/v2-20a82036ffa22f662e022c3c745539d4_b.png)

```json
  "mainEntryPointFilePath": "<projectFolder>/lib/index.d.ts"
```

src/index.ts

```ts
 export { default as Log } from './log/Log';
 export { default as ILogHandler } from './log/ILogHandler';
```

src/log/Log.ts

```ts
 import DefaultLogHandler from './DefaultLogHandler';
 import ILogHandler from './ILogHandler';
 ​
 export default class Log {
   private static _logHandler: ILogHandler = new DefaultLogHandler();
   public static initialize(logHandler: ILogHandler): void {
     Log._logHandler = logHandler;
   }
   public static verbose(source: string, message: string): void {
     this._logHandler.verbose(source, message);
   }
 }
src/log/ILogHandler.ts
 export interface ILogHandler {
   verbose(source: string, message: string): void;
 }
 ​
 export default ILogHandler;
src/log/DefaultLogHandler.ts
 export default class DefaultLogHandler {
   verbose(source: string, message: string): void {
     console.log(message)
   } 
 }
```

使用tsc 编译在生成如下文件

```txt
 lib
 ├── index.d.ts
 ├── index.js
 ├── log
    ├── DefaultLogHandler.d.ts
    ├── DefaultLogHandler.js
    ├── ILogHandler.d.ts
    ├── ILogHandler.js
    ├── Log.d.ts
    └── Log.js
```

执行命令行脚本 

```bash
  npx api-extractor run --local --verbose 
```

默认在根目录dist下生成 `<projectFolder>.d.ts`

```ts
export declare interface ILogHandler {
    verbose(source: string, message: string): void;
}
export declare class Log {
    private static _logHandler;
    static initialize(logHandler: ILogHandler): void;
    static verbose(source: string, message: string): void;
}
export { }
```

## scripts

> 源代码中通过  const args = require("minimist")(process.argv.slice(2)); 接收命令行参数
>
> 以build脚本的为例，执行获取参数接收都是如下格式， 注意带--与不带--
>
> 执行： yarn build im --formats=cjs
>
> 输出： { _: [ 'im' ], formats: 'cjs' }

## bootstrap

[代码地址](https://github.com/vuejs/vue3/blob/b228abb72fcdb4fc9dced907f3614abcaaacdce5/scripts/bootstrap.js)

**作用：** 用于生成最小化的子包

**参数：** 

**--force ：** 文件存在，也重新生成在packages中新建一个目录如 `a` , 执行`node ./scripts/bootstrap`

不存在以下文件  `src/index.ts, index.js,  README.md, package.json, api-extractor.json` 则自动生成


## checkYarn

**package.json**

在使用yarn安装前触发的钩子

```json
 "scripts": {
     "preinstall": "node ./scripts/checkYarn.js",
 },
```

包依赖都是通过yarn进行管理，所以需要检查当前是否是使用的yarn，通过package.json 的 scripts 的preinstall 命令，会在使用yarn 安装依赖前执行 .`/scripts/checkYarn.js`

```json
"scripts": {
  "preinstall": "node ./scripts/checkYarn.js",
},
```

`./scripts/checkYarn.js:`

通过判断process.env.npm_execpath是否包含yarn.js，来判断，如我使用yarn进行安装时 `npm_execpath: 'E:\\nodejs\\node_modules\\yarn\\bin\\yarn.js'`

```js
 if (!/yarn\.js$/.test(process.env.npm_execpath || "")) {
   console.warn(
     "\u001b[33mThis repository requires Yarn 1.x for scripts to work properly.\u001b[39m\n"
   );
   process.exit(1);
 }
```

## verifyCommit

**作用：** 规范commit提交信息

使用尤大改写的yorkie，yorkie实际是fork husky，然后做了一些定制化的改动，使得钩子能从package.json的 "gitHooks"属性中读取

**package.json**

```json
  "gitHooks": {
     "pre-commit": "lint-staged",
     "commit-msg": "node scripts/verifyCommit.js"
   },
   "devDependencies": {
     "yorkie": "^2.0.0"
  }
```

verifyCommit

```js
 const chalk = require('chalk')
 const msgPath = process.env.GIT_PARAMS
 const msg = require('fs')
   .readFileSync(msgPath, 'utf-8')
   .trim()
 ​
 const commitRE = /^(revert: )?(feat|fix|docs|dx|style|refactor|perf|test|workflow|build|ci|chore|types|wip|release)(\(.+\))?: .{1,50}/
 ​
 if (!commitRE.test(msg)) {
   console.log()
   console.error(
     `  ${chalk.bgRed.white(' ERROR ')} ${chalk.red(
       `invalid commit message format.`
     )}\n\n` +
       chalk.red(
         `  Proper commit message format is required for automated changelog generation. Examples:\n\n`
       ) +
       `    ${chalk.green(`feat(compiler): add 'comments' option`)}\n` +
       `    ${chalk.green(
         `fix(v-model): handle events on blur (close #28)`
       )}\n\n` +
       chalk.red(`  See .github/commit-convention.md for more details.\n`)
   )
   process.exit(1)
 }
```

## dev

**主要作用：** 在监视模式下运行Rollup进行开发

- 要指定要监视的包，只需传递它的名称和所需的构建，要观察的格式(默认为"global")， 
- Name 支持模糊匹配。将监视所有名称包含“dom”的包

 `# yarn dev <name>`， 例： `yarn dev dom`

- 指定要输出的格式
  `yarn dev core --formats cjs`
- 删除所有的DEV块, 也就是
  `__DEV__=false yarn dev`

## release

**主要作用：** 用于发布版本

**流程：** 

1. 发布前运行测试
2. 更新所有包的版本和相互依赖
3. 构建所有带有类型的包
4. 生成更新日志（conventional-changelog-cli包生成CHANGELOG.md)
5. 发布包
6. 推送到GitHub

**参数：**

- --preid  // 用于前缀 premajor、preminor、 prepatch 或 prerelease 版本增量。
- --dry // 配合 skipTests，skipBuild 使用，（只有当skipTests  与dry  都不传时才执行测试），skipBuild  与dry  都不传时才执行打包
- --skipTests    // 跳过测试
- --skipBuild    // 跳过打包

## build & utils

### utils:

**targets ：** 读取packages下所有package.json 中private= false, 并且包含 buildOptions 字段的包文件夹

**fuzzyMatchTarget ：** 模糊匹配包名，用于build 单个包

**源码：**

```js
 const fs = require('fs')
 const chalk = require('chalk')
 ​
 const targets = (exports.targets = fs.readdirSync('packages').filter(f => {
   if (!fs.statSync(`packages/${f}`).isDirectory()) {
     return false
   }
   const pkg = require(`../packages/${f}/package.json`)
   if (pkg.private && !pkg.buildOptions) {
     return false
   }
   return true
 }))
 ​
 exports.fuzzyMatchTarget = (partialTargets, includeAllMatching) => {
   const matched = []
   partialTargets.forEach(partialTarget => {
     for (const target of targets) {
       if (target.match(partialTarget)) {
         matched.push(target)
         if (!includeAllMatching) {
           break
         }
       }
     }
   })
   if (matched.length) {
     return matched
   } else {
     console.log()
     console.error(
       `  ${chalk.bgRed.white(' ERROR ')} ${chalk.red(
         `Target ${chalk.underline(partialTargets)} not found!`
       )}`
     )
     console.log()
 ​
     process.exit(1)
   }
 }
```

### build:

[源码地址](https://github.com/vuejs/vue3/blob/HEAD/scripts/build.js)

**主要作用：** 使用 rollup 打包所有或单个包，合并.d.ts至根目录

**使用：** `yarn build [<包名>]   [参数]`          //    包名满足模糊匹配即可，   没传包名则默认打包全部

**参数：** 

- --formats //   输出的文件类型 (amd, cjs, esm, iife, umd)
- --d |  --devOnly  // 指定环境为development，默认为production
- --p |  --prodOnly // 指定环境为production
- --s | --sourcemap // 开启sourcemap
- --release         // 只构建发布的包
- --t | --types     // 是否使用api-extractor.json合并.d.ts至根目录，此参数需要配置子包中 package.json的 "types": "dist/compiler-dom.d.ts"
- --a | --all       // 模糊匹配 所有的满足包， 不加 默认是匹配到的第一个包

**子包package.json:**
这里是收集vue-next所有包中用到的字段，并不是某个包的字段

```json
"buildOptions": {
  "name": "heartImUtil",  // 打包后的文件名前缀
  "compat": true,         // 用于特定vue-compat包
  "formats": [        // 输出的文件类型, 查看rollup.config.js 中对应outputConfigs的类型
    "esm-bundler",    
    "esm-bundler-runtime",
    "cjs",
    "global",
    "global-runtime",
    "esm-browser",
    "esm-browser-runtime",    
  ],
  "env": "development",   // 指定当前环境
  "prod": false,          // process.env.NODE_ENV 为 production 时prod = false ，直接跳过打包
  "enableNonBrowserBranches": true    // true用于node端， false 用于node端
},
```

## rollup.config.js

[源码地址](https://github.com/vuejs/vue-next/blob/master/rollup.config.js)

**该文件为rollup执行的配置文件， 在build.js或dev.js的中执行rollup 命令时都会调用该配置，所以在打包不同子包时会根据format类型  或者 输出文件名提供不同的环境变量**

**打包类型参数 获取优先级：** process.env.FORMATS >   packageOptions.formats >  defaultFormats

packageOptions.formats 为`子包package.json` 中的字段


**outputConfigs支持的打包的类型如下：**

```js
 const outputConfigs = {
   "esm-bundler": {
     file: resolve(`dist/${name}.esm-bundler.js`),
     format: `es`,
   },
   "esm-browser": {
     file: resolve(`dist/${name}.esm-browser.js`),
     format: `es`,
   },
   cjs: {
     file: resolve(`dist/${name}.cjs.js`),
     format: `cjs`,
   },
   global: {
     file: resolve(`dist/${name}.global.js`),
     format: `iife`,
   },
   // runtime-only builds, for main "vue" package only
   "esm-bundler-runtime": {
     file: resolve(`dist/${name}.runtime.esm-bundler.js`),
     format: `es`,
   },
   "esm-browser-runtime": {
     file: resolve(`dist/${name}.runtime.esm-browser.js`),
     format: "es",
   },
   "global-runtime": {
     file: resolve(`dist/${name}.runtime.global.js`),
     format: "iife",
   },
 };
 ​
```

**根据不同的format类型 或者 输出文件名 给子包中提供不同的环境变量**

**部分代码：** createReplacePlugin 中使用@rollup/plugin-replace 替换变量

```js
 import replace from '@rollup/plugin-replace'
 function createConfig(format, output, plugins = []) {
   if (!output) {
     console.log(require("chalk").yellow(`invalid format: "${format}"`));
     process.exit(1);
   }
 ​
   output.exports = "auto";
   output.sourcemap = !!process.env.SOURCE_MAP;
   output.externalLiveBindings = false;
 ​
  // 判断format 或者 输出文件名 来注入环境变量
   const isProductionBuild =
     process.env.__DEV__ === "false" || /\.prod\.js$/.test(output.file);
   const isBundlerESMBuild = /esm-bundler/.test(format);
   const isBrowserESMBuild = /esm-browser/.test(format);
   const isNodeBuild = format === "cjs";
   const isGlobalBuild = /global/.test(format);
 ​
   if (isGlobalBuild) {
     output.name = packageOptions.name;
   }
     ...省略代码...
   return {
   ...省略代码...
     plugins: [
       createReplacePlugin(
         isProductionBuild,
         isBundlerESMBuild,
         isBrowserESMBuild,
         // isBrowserBuild?
         (isGlobalBuild || isBrowserESMBuild || isBundlerESMBuild) &&
           !packageOptions.enableNonBrowserBranches,
         isGlobalBuild,
         isNodeBuild,
       ),
       ...省略代码...
     ],
   };
 }
 function createReplacePlugin(
   isProduction,
   isBundlerESMBuild,
   isBrowserESMBuild,
   isBrowserBuild,
   isGlobalBuild,
   isNodeBuild,
 ) {
   const replacements = {
     __COMMIT__: `"${process.env.COMMIT}"`,
     __VERSION__: `"${masterVersion}"`,
     __DEV__: isBundlerESMBuild
       ? // preserve to be handled by bundlers
         `(process.env.NODE_ENV !== 'production')`
       : // hard coded dev/prod builds
         !isProduction,
     // this is only used during Vue's internal tests
     __TEST__: false,
     // If the build is expected to run directly in the browser (global / esm builds)
     __BROWSER__: isBrowserBuild,
     __GLOBAL__: isGlobalBuild,
     __ESM_BUNDLER__: isBundlerESMBuild,
     __ESM_BROWSER__: isBrowserESMBuild,
     // is targeting Node (SSR)?
     __NODE_JS__: isNodeBuild,
      ...省略代码...
   };
   // allow inline overrides like
   //__RUNTIME_COMPILE__=true yarn build runtime-core
   Object.keys(replacements).forEach((key) => {
     if (key in process.env) {
       replacements[key] = process.env[key];
     }
   });
   return replace({
     // @ts-ignore
     values: replacements,
     preventAssignment: true,
   });
 }
```