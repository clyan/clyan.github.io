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

目录结构如下：
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
可以使用`gulp-load-plugins`
```js
npm i gulp-load-plugins -D
```
