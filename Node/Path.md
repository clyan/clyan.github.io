## 简介
**主要包括以下几个部分**
1. 获取路径
2. 获取文件名
3. 获取拓展名
4. 路径组合
5. 路径分解与解析

## 获取路径(dirname)
```js
const filePath = "E:\\learnMaterials\\Node系统性学习\\package.json";
const dirname = path.dirname(filePath);
console.log('dirname: ', dirname);  //  E:\learnMaterials\Node系统性学习
```

## 获取文件名 (basename)
> 注意严格来讲是获取地址的最后一部分
```js
const filePath = "E:\\learnMaterials\\Node系统性学习\\package.json";
const basename = path.basename(filePath);
console.log('basename: ', basename); // package.json
```
将`filePath` 改成`E:\\learnMaterials\\Node系统性学习` 那么输出的是 `Node系统性学习` 所以请注意是返回最后一部分`而不是文件名`

只想获取文件名，不包括文件扩展， 可以使用第二个参数
```js
const basename = path.basename(filePath, '.json');
console.log('basename: ', basename); // package.json
```

## 获取文件拓展名(extname)
> 原理，从最后一个`.`开始截取，如果没有`.`返回 `''`, `.`位于第一位返回`''`, `.` 位于最后一位返回`.`

```js
let filePath = 'package.json'
const extname = path.extname(filePath);
console.log('extname: ', extname); // .json

const NotHasPointExtname = path.extname('package');
console.log('NotHasPointExtname: ', NotHasPointExtname); // ''

const FirstHasPointExtname = path.extname('.package');
console.log('FirstHasPointExtname: ', FirstHasPointExtname); // ''

const LastHasPointExtname = path.extname('package.');
console.log('LastHasPointExtname: ', LastHasPointExtname); // .

const TwoPointExtname = path.extname('package.ts.js');
console.log('TwoPointExtname: ', TwoPointExtname);  // .js
```