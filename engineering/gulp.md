## 基本使用

1. 安装
```bash
npm init -y
npm install --global gulp-cli
npm install gulp -D
```

2. 根目录下添加`gulpfile.js`, 并导出foo，表示建立一个foo任务
```js
exports.foo = function(done) {
    console.log("foo task")
    done() // 标识任务完成
}
exports.default = function(done) {
    console.log("default task")
    done() // 标识任务完成
}
```
在命令行执行

执行`gulp foo`命令，表示执行foo任务

![](https://i.loli.net/2021/11/29/7LVFeBiTPI83zbu.png)

执行`gulp`命令 默认执行`exports.default` 任务

![](https://i.loli.net/2021/11/29/K8WziwYUbPmcAH9.png)

在`gulp4.0`之前, 使用`task api`添加任务, `4.0`之后依旧兼容，但不再推荐

在`gulpfile.js`中添加如下代码
```js
    const gulp = require('gulp')
    gulp.task('bar', done => {
        console.log('bar task')
    })
```
执行 `gulp bar`

![](https://i.loli.net/2021/11/29/zkeufcBGLlTda9R.png)

## 组合任务
修改`gulpfile.js`, 创建3个异步任务，并且使用`series`, `parallel` 分别创建`foo`, `bar`两个任务

```js
const { series, parallel } = require('gulp')

const taskl = done => {
    setTimeout(()=> {
        console.log('taskl')
        done()
    }, 1000)
}

const task2 = done => {
    setTimeout(()=> {
        console.log('task2')
        done()
    }, 1000)
}

const task3 = done => {
    setTimeout(()=> {
        console.log('task3')
        done()
    }, 1000)
}
//  series 顺序执行
exports.foo = series(taskl,  task2, task3)

//  parallel 顺序执行
exports.bar = parallel(taskl,  task2, task3)
```

执行 `gulp foo`

![](https://i.loli.net/2021/11/29/QzyM9Ighib1PWFL.png)

可以看到，`series` 创建任务是串行的，一个任务结束后，另一个任务才开始

执行 `gulp bar` 

可以看到，`parallel` 创建任务是并行的，所以任务都是同时开始

![](https://i.loli.net/2021/11/29/jHkt9uer1ybiDqI.png)

**`series`与`parallel`应用场景**

`series`: 执行部署的任务，需要先执行编译任务，即需要串行的执行
`parallel`: 编译css、js互不干扰，即可使用并行任务，提高构建效率

## 异步任务
> gulp当中的任务都是异步任务, 支持callback， promise, async , stream

修改`gulpfile.js`
```js
const { time } = require("console")

exports.callback = done => {
    console.log("callback")
    done()
}

// 执行任务中报出一个错误，阻止剩下的任务执行, 如果是多个任务同时执行，后续任务不会再执行
exports.callback_error = done => {
    console.log("callback")
    done(new Error("task failed"))
}

exports.promise = ()=> {
    console.log("promise")
    return Promise.reject(new Error("promise task"));
}

exports.promise_error = ()=> {
    console.log("promise_error")
    return Promise.reject(new Error("promise_error task failed"));
}

const timeout = (time)=> {
    return new Promise(resolve => {
        setTimeout(resolve, time)
    })
}
exports.async = async ()=> {
    await timeout(time)
    console.log("async task")
}

// 返回readStream, gulp 会通过readStream 的end事件结束该任务，
exports.stream = async ()=> {
    const readStream = fs.createReadStream('package.json')
    const writeStream = fs.createWriteStream('temp.txt')
    readStream.pipe(writeStream)
    // readStream.on('end', ()=> {done()})
    return readStream
}
```

## 核心构建过程

如下， 使用`node api` 将 `reset.css` 进行压缩，核心过程 `读取` -> `转换` -> `写入`,

修改`gulpfile.js`
```js
const fs = require('fs');
const { Transform } = require('stream')
exports.default = function(done) {
    // 读取文件流
    const read = fs.createReadStream('reset.css')
    // 写入文件流
    const write = fs.createWriteStream('reset.min.css')
    // 文件转换流
    const transform = new Transform({
        transform: (chunk, encoding, callback) => {
            // 核心转换过程实现
            // chunk  => 读取流中读取到的内容
            const input = chunk.toString()
            const output = input.replace(/\s+/g, '').replace(/\/\*.+?\*\//, '')
            // 错误有限
            callback(null, output)
        }
    })
    read
    .pipe(transform) // 先进行转换
    .pipe(write)    // 再写入
    return read
}
```

**效果如下：**

![](https://i.loli.net/2021/11/29/cfsgkHy1Ko9EhiB.png)

## Gulp文件操作API
> Gulp 提供了文件操作的API，像比如node API 更加强大，对于转换流使用独立的插件来提供

安装用于`压缩css`与`重命名`的插件 (也就相当于转换流)
```bash
npm install gulp-clean-css -D

npm install gulp-rename -D
```

修改`gulpfile.js`
```js
const { src, dest} = require('gulp');
const cleanCss = require('gulp-clean-css');
const rename = require('gulp-rename');
exports.default = () => {
    return src('src/css/*.css') // 匹配src/css下的所有css文件
    .pipe(cleanCss())
    .pipe(rename({ extname: '.min.css' })) //修改文件名
    .pipe(dest('dist'))    // 输出目录
}
```
执行结果如下： 

![](https://i.loli.net/2021/11/29/oWxugGyfUI4VhlP.png)

## 构建一个完整的应用

**先准备好一个案例：**

`6-gulp-aplication`目录结构如下：
![](https://i.loli.net/2021/11/29/x14htZsHcdUJPzE.png)

### 样式编译

以编译`sass`为例,安装`gulp-sass`
```bash
npm i gulp-sass -D
```

在`gulpfile.js`中添加如下代码
```js
const { src, dest } = require('gulp')
const sass = require('gulp-sass')
const style = () => {
  return src('src/assets/styles/*.scss', { base: 'src' })
    .pipe(sass({ outputStyle: 'expanded' })) //outputStyle: 'expanded' 生成的css代码 } 换行，具体查gulp-sass文档
    .pipe(dest('dist'))
}

module.exports = {
  style
}
```
**如下：** 以`_` 开头的文件默认不会单独的抽离出一个文件

![](https://i.loli.net/2021/11/29/dMYbu6EiARtl7Nn.png)

### 脚本编译

使用`babel`转换高版本`js`, 安装`gulp-babel`, ` @babel/core`, `@babel/preset-env`
```bash
npm i gulp-babel @babel/core @babel/preset-env -D
```

在`gulpfile.js`中添加如下代码
```js
const babel = require('gulp-babel')
const script = () => {
  return src('src/assets/scripts/*.js', { base: 'src' })
    .pipe(babel({ presets: ['@babel/preset-env'] })) //注意传入preset-env进行转换
    .pipe(dest('dist'))
}
module.exports = {
  script
}
```
执行`gulp script`

![](https://i.loli.net/2021/11/29/AcRB7gYKX9oPqT5.png)

### 页面模板编译
> 为了使页面中的一些内容被重用，那么使用模板引擎，这里的模板引擎为`swig`

```bash
npm i gulp-swig -D
```

在`gulpfile.js`中添加如下代码
```js
// 注入到模板中的值
const data = {
  menus: [
    {
      name: 'Home',
      icon: 'aperture',
      link: 'index.html'
    },
    {
      name: 'Features',
      link: 'features.html'
    },
    {
      name: 'About',
      link: 'about.html'
    },
    {
      name: 'Contact',
      link: '#',
      children: [
        {
          name: 'Twitter',
          link: 'https://twitter.com/'
        },
        {
          name: 'About',
          link: 'https://weibo.com/'
        },
        {
          name: 'divider'
        },
        {
          name: 'About',
          link: 'https://github.com/'
        }
      ]
    }
  ],
  pkg: require('./package.json'),
  date: new Date()
}
const page = () => {
  return src('src/*.html', { base: 'src' })
    .pipe(swig({ data, defaults: { cache: false } })) // 防止模板缓存导致页面不能及时更新
    .pipe(dest('dist'))
}
```

执行`gulp page`

### 组合运行上面3个任务
一个个执行命令，太不方便，使用组合任务一次执行

因为以上三个任务没有任何的依赖关系，所以可以使用`parallel` 创建一个并行任务

在`gulpfile.js`中添加如下代码
```js
import { src, dest, parallel } from 'gulp'

const compile = parallel(style, script, page) // 组合上面3个任务
module.exports = {
  compile // 导出compile 即可，style, script, page不用导出
}
```

### 图片和字体文件转换
```bash
npm i gulp-imagemin -D
```

在`gulpfile.js`中添加如下代码
```js
import imagemin from 'gulp-imagemin'
const image = () => {
  return src('src/assets/images/**', { base: 'src' })
    .pipe(imagemin())
    .pipe(dest('dist'))
}

const font = () => {
  return src('src/assets/fonts/**', { base: 'src' })
    .pipe(imagemin()) // 压缩font目录下的img
    .pipe(dest('dist'))
}
const compile = parallel(style, script, page, image, font) // 增加组合
module.exports = {
  compile
}
```
执行： `gulp compile`

### 其他文件及文件清除
1. 在生成新的文件前先清除目标目录， 
2. 将其他文件直接复制到目标目录

使用第三方包，删除目标目录的文件
```bash
npm i del -D
```
使用`series` 创建一个新的build任务, 顺序执行，先清除，再进行打包
```js
const { src, dest, parallel, series } = require('gulp')
const del = require('del')
// 清除任务
const clean = () => {
  return del(['dist'])
}
// 复制任务
const extra = () => {
  return src('public/**', { base: 'public' })
    .pipe(dest('dist'))
}
const compile = parallel(style, script, page, image, font) // 增加组合
// 使用 series 创建一个顺序的任务，先清除，才能进行打包
const build =  series(
  clean,
  parallel(
    compile,
    extra
  )
)
// 导出
module.exports = {
  compile,
  build
}
```

### 自动加载插件
上面的每个步骤，每次安装一次插件，都需要require一次插件，很麻烦

如：以上使用了这些插件
```js
const sass = require('gulp-sass')
const babel = require('gulp-babel')
const swig = require('gulp-swig')
const imagemin = require('gulp-imagemin')
```

可以使用`gulp-load-plugins`简化引用， 
```js
npm i gulp-load-plugins -D
```
所有的插件都作为`plugins` 上的属性
```js
const loadPlugins = require('gulp-load-plugins')
const plugins = loadPlugins()

// 以font举例
const font = () => {
  return src('src/assets/fonts/**', { base: 'src' })
    .pipe(plugins.imagemin()) // imagemin() 改成 plugins.imagemin()
    .pipe(dest('dist'))
}
```

### 开发服务器
在开发阶段,实时调试我们的代码，就需要实时编译

安装 `browser-sync`
```bash
npm install browser-sync -D
```

很好理解，使用`watch`监听文件，改变后刷新`browser`服务器
```js
const { src, dest, parallel, series, watch } = require('gulp')
const serve = () => {
  watch('src/assets/styles/*.scss', style) // 监听文件，重新执行任务
  watch('src/assets/scripts/*.js', script)
  watch('src/*.html', page)
  watch([
    'src/assets/images/**',
    'src/assets/fonts/**',
    'public/**'
  ], bs.reload)

  bs.init({
    notify: false, 
    port: 2080,
    // open: false,
    // files: 'dist/**', // 监听哪些文件改变后，刷新浏览器
    server: {
      baseDir: ['dist', 'src', 'public'], // 监听
      // 开发阶段，如果html引入了node_modules中的资源，需要指定routes
      routes: {
        '/node_modules': 'node_modules' // 指向项目下的 node_modules
      }
    }
  })
}
```

### 集成ESlint
用于在打包编译前，格式化我的代码，同时格式化出错时，终止运行

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

### 封装工作流
如果我们开发多个项目，多个项目的构建任务一般都是相同的，涉及到gulpfile复用问题，可以在多个项目中使用同一个gulpfile

弊端： gulpfile散落在各个项目中，如果有部分出问题或升级，需要对各个项目做相同的操作，不利于整体的维护


封装gulp方便使用，gulefile + gulp = 构建工作流


#### 准备工作
新建项目`gulp-packaging`写好项目基本目录
```txt
- gulp-packaging
  - lib
    - index.js
  - ChangeLOG.md
  - package.json
```
并修改package.json
```js
{
  main:"./lib/index.js"
}
```


使用changelog 自动生成commit记录
```bash
npm install -g conventional-changelog-cli
```

修改package.json
```js
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
  },
```
#### 提取gulpfile

将上面`构建一个完整的应用`案例中的完整`gilefile`复制至lib/index.js中, 然后修改`gulp-packaging`包中的`lib/index.js` 文件

本地调试`gulp-packaging`， 执行`npm link`, 然后在`6-gulp-aplication`目录下执行`npm link gulp-packaging` 

同时修改`6-gulp-aplication`中的`gulpfile`如下
```js
module.exports = require("gulp-packaging")
```
在`6-gulp-aplication`目录下执行`gulp build`（如果提示gulp不是可执行的命令，则执行npx gulp build） 依旧可以执行打包任务

**但存在几个问题：**
1. 项目目录不能够自定义
2. gulpfile仅仅只起到一个引入导出的作用，显得冗余

### 扩展配置
> 使使用者可以自定义打包路径及项目目录
在`gulp-packaging`的`lib/index.js`中添加如下内容，用于读取用户使用时目录下的`gulp.packaging.config.js`

```js
  // 获取node运行的当前目录
  const cwd = process.cwd()
  let config = {
    build: {
      src: "src",
      dist: 'dist',
      temp: 'temp',
      public: 'public',
      paths: {
        styles: 'assets/styles/*.css',
        scripts: 'assets/scripts/*.js',
        pages: '*.html',
        images: 'assets/images/**',
        fonts: 'assets/fonts/**'
      }
    }
  }
  try {
    const loadConfig = require(`${cwd}/gulp.packaging.config.js`)
    // 默认配置进行合并
    config = {...config, loadConfig}
  } catch (error) {
    
  }
  ......省略
  const clean = () => {
    // 使用config.build.dist动态配置路径
    return del([config.build.dist, config.build.temp])
  }
  ......省略
  const style = () => {
    return src(config.build.paths.styles, { base: config.build.src, cwd: config.build.src })
      .pipe(plugins.sass({ outputStyle: 'expanded' }))
      .pipe(dest(config.build.temp))
      .pipe(bs.reload({ stream: true }))
  }
```
根据约定大于配置, 约定在项目目录下需要建立 `gulp.packaging.config.js`为`gulp-packaging`提供配置

`6-gulp-aplication` 下新建 `gulp.packaging.config.js`
```js
module.exports = {
    build: {
        src: "src",
        dist: 'release',
        temp: '.temp',
        public: 'public',
        paths: {
            styles: 'assets/styles/*.css',
            scripts: 'assets/scripts/*.js',
            pages: '*.html',
            images: 'assets/images/**',
            fonts: 'assets/fonts/**'
        }
    }
}
```

修改完以后，就可以自定义项目目录及打包路径了~~~

### 自定义命令
> 使用自定义命令，使用时将gulpfile省略

**先看下不使用自定义命令怎么省略gulpfile配置**

首先把`6-gulp-aplication`的gulpfile删除，执行`npx gulp build`, 提示错误，未找到gulpfile

![](https://s2.loli.net/2021/12/07/h4wXKSDvOpQ93eE.png)

那我们就指定一个gulpfile就好了，在`6-gulp-aplication`目录中已经执行过`npm link gulp-packaging`

那么就可以在`node_modules`中找到`gulp-packaging`下的 `lib/index.js`

![](https://s2.loli.net/2021/12/07/67ATk4rVy3bhzIP.png)

执行`npx gulp build --gulpfile ./node_modules/gulp-packaging/lib/index.js`

![](https://s2.loli.net/2021/12/07/Y6hQz1EjKUAy2cx.png)

那么也能够成功编译构建。

**使用自定义命令简化开发**

由上面可以知道，我们只需要指定一个gulpfile文件即可

首先在`gulp-packaging`的package.json中添加
```js
  "bin": {
    "gulp-pack": "./bin/gulp-packaging.js"
  },
```
新增`bin/gulp-packaging.js`
```js
#!/usr/bin/env node
process.argv.push('--cwd')
process.argv.push(process.cwd())
process.argv.push('--gulpfile')
// require.resolve() 找到这个模块所对应的路径, ..默认会找当前目录下
process.argv.push(require.resolve('..'))
// console.log(process.argv)
require('gulp/bin/gulp')
```
首先使用`process.argv` 指定gulp运行参数，然后再引入`gulp/bin/gulp` 执行gulp命令，就达到了一个自动指定gulpfile并执行gulp的一个作用

接着再执行`npm link`(如果出错，先执行npm unlink), 在`6-gulp-aplication`中 `npm link gulp-packaging`

修改`6-gulp-aplication`中的package.json
```json
  "scripts": {
    "clean": "gulp-pack clean",
    "build": "gulp-pack build",
    "develop": "gulp-pack develop"
  },
```
执行`npm run build`

![](https://s2.loli.net/2021/12/07/hT1doyMS3FL4BJg.png)


### 发布gulp-packaging

`npm login`: 输入自己的npm账号密码

`npm publish`: 发布

过一小会，就能在自己的npm仓库看到啦~~~

![](https://s2.loli.net/2021/12/07/hUKNOjzgBcfe6wX.png)
# 结语
对于开发者而言，一开始需要技能，然后需要想法，想法建立在技能的基础之上，当技能能够满足想法的时候，想法越多越好