# 使用方式
> 由于活动道具是页面写死的，并且是从表格中复制，原本需要一个个复制写成json,效率太低

[在线体验地址](https://codesandbox.io/s/json-helper-fmzgy?file=/index.html)

##  根据复制内容生成
**如下：**
1. 从飞书表格中复制至源数据框内
2. 数据对应列的keys (keys的长度需要与列数保持一致，并且keys以`空格`分隔)
3. 点击生成即可生成目标JSON
4. 点击复制就可以用了
   
![image.png](https://i.loli.net/2021/11/28/eBrVN4Ixvi9RQCS.png)

## 根据范围自动生成
道具都还拥有图片，一般是链接的地址，命名从规则一般为`1-n`。

1. 勾选 `开启范围`
2. 输入前缀，后缀 （也可不加，看具体需求）
3. 在源数据框中输入 `1-n` ，同时如果此时目标JSON框中有数据，n不能大于 `目标JSON` 的长度

![image.png](https://i.loli.net/2021/11/28/Q7pcCPoXyBDw5q1.png)

## 保存历史记录
## 合并JSON对象数组
> 如果你不小心已经生成了JSON, 又没有复制，页面又刷新了，不用担心，每次生成都会持久化缓存

> 你可以快速的将历史记录作为源数据或者目标数据

> 如果你将历史记录（也就是输入的是JSON对象数组） 记得关闭 `开启范围` ， `开启范围`  只在生成范围时使用

![image.png](https://i.loli.net/2021/11/28/wMNgyRoLvOeFAm2.png)

**点击生成**

可以看到img已经合并到，每个json的对象的最下面一个了， 所以注意，源数据中的`键值`始终是合并到`目标JSON`的最后面

![image.png](https://i.loli.net/2021/11/28/xYDcA6anX3UTIkV.png)

对于图片本身还是需要自己上传到服务器，并按顺序命好名。
<details>
  <summary>展开查看源码</summary>

  ```html
     <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>活动道具JSON生成器</title>
        <style>
            .page-title {
                text-align: center;
                border: 1px solid #ccc;
                padding: 10px 0 ;
            }
            .container {
                margin: 0 auto;
                display: flex;
                gap: 10px;
            }
            .history {
                flex: 1;
                border: 1px solid #ccc;
                padding: 10px;
            }
            .history-list {
                display: flex;
                flex-direction: column;
                gap: 10px;
                overflow-y: auto;
                max-height: 672px;
            }
            .history-item {
                border-bottom: 1px solid #ccc;
            }
            .item-label {
                display: inline-block;
                width: 80px;
            }
            .primary {
                flex: 2;
                border: 1px solid #ccc;
                padding: 10px;
            }
            .tool {
                text-align: center;
                margin-bottom: 10px;
            }
            textarea {
                padding: 0;
            }
            .page-main {
                display: flex;
                flex-direction: row;
                gap: 10px;
            }
            .item {
                flex: 1;
            }
            .source {
                width: 100%;
                height: 500px;
            }
            .target {
                display: inline-block;
                width: 100%;
                min-height: 500px;
                overflow-y: auto;
                border: 1px solid #ccc;
                margin: 0;
                max-height: 600px;
            }
        </style>
    </head>
    <body>
        <div class="container-wrapper">
            <h1 class="page-title">活动道具JSON生成器</h1>
            <div class="container">
                <div class="history">
                    <p>历史记录: <button class="clear-history">清除历史</button></p>
                    <div class="history-list">
                    </div>
                </div>
                <div class="primary">
                    <div class="tool">
                        <button class="creater">生成</button>
                        <button class="clear">清除</button>
                        <button class="rule">使用方式</button>
                    </div>
                    keys: <input class="keys" />
                    <div class="page-main">
                        <div class="item">
                            <p>源数据:</p>
                            开启范围：<input type="checkbox" class="auto" />
                            <p class="prefix-box"  style="display:none;" >prefix: <input type="text" class="prefix" /></p>
                            <p class="suffix-box"  style="display:none;" >suffix: <input type="text" class="suffix" /></p>
                            <textarea class="source"></textarea>
                        </div>
                        <div class="item">
                            <p>目标JSON:</p>
                            <p class="copy-box"><button class="copy">复制到剪贴板</button></p>
                            <pre><code class="target"></code></pre>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <script>
            // 存入localStorage中的键
            let localHistoryKey = 'tool-generator-history';
            function $(el) {
                return document.querySelector(el)
            }
            // 封装监听事件，默认为点击事件
            function on(el, cb, event = 'click') {
                el.addEventListener(event, cb)
            }
            // 转换
            function transform(source, keys) {
                let result = source.map(item => {
                    let list = item.split('\t');
                    return list.reduce((tol, cur, index)=> {
                        if(cur === '') return tol;
                        return { ...tol, [keys[index]]: cur }
                    }, {})
                })
                return result
            }
            function getKeys(keys) {
                if(!keys) return 
                return keys.split(' ')
            }
            // 合并两个对象数组，每个对象的键也合并
            function mergeObjectList(objList1 = [], objList2 = []) {
                return objList1.reduce((tol, cur, index )=> {
                    return [...tol, {...cur, ...objList2[index]}]
                },[])
            }
            // 多选框是否选中
            function isChecked(el) {
                return el.checked
            }
            // 生成源数据
            function generatorSource() {
                let source = $(".source").value;
                if(!source)  return [false, 0];
                // 若果不是使用数字自动生成， 则直接返回源数据
                if(!isChecked($(".auto"))) {
                    try {
                        // 尝试转换为JSON对象，转换成功则返回该对象
                        return [JSON.parse(source), 1]
                    } catch (error) {
                        // 转换失败则返回切割字符
                        return [source.split(/[( )+|\n]/), 0];
                    }
                }
                if(!/\d+-\d+/.test(source)) {
                    alert("格式有误，请输入 数字-数字，如 1-5")  
                    return [false, 0];
                }
                source = source.split('-')
                let start = parseInt(source[0])
                let end = parseInt(source[1])
                let prefix = $(".prefix").value
                let suffix = $(".suffix").value
                let s = ''
                for(let i = start; i <= end; i++) {
                    s = s + (prefix ? prefix : '') + i + (suffix ? suffix : '') + (i === end ? '': '\n')
                }
                return [s.split(/[( )+|\n]/), 0];
            }
            // 验证key与source是否一样长
            function isKeyAndSourceMatching(source, flag, keys) {
                if(flag === 1) {
                    return true;
                } else {
                    return source[0].split('\t').filter(item => item !== '').length === keys.length
                }
            }
            // 生成JSON数据
            function generator() {
                let [source, flag] = generatorSource();
                if(!source) return false;
                let keys = getKeys($(".keys").value);
                // 需要将每一行通过\t分割并且去除最后的 '' 的长度与key的长度进行对比
                if(!isChecked($(".auto")) && !isKeyAndSourceMatching(source, flag, keys)) {
                    alert("键的个数 与 源数据列数 不匹配！")
                    return false
                }
                // 如果有目标数据，说明是合并数据
                let preResult = $(".target").innerText;
                if(preResult) {
                    let preResultList = JSON.parse(preResult);
                    if(preResultList.length !== source.length && isChecked($(".auto"))) {
                        alert("最大数值与目标数据长度不一致")
                        return false
                    }
                } else {
                    if(!keys) return false;
                }
                // flag === 1 时,代表是两个对象在合并，而不用分割
                let result = !!flag ? source : transform(source, keys);
                if($(".target").innerText !== '') {
                    console.log("result", result)
                    result = mergeObjectList( JSON.parse(preResult), result)
                }
                $(".target").innerText = JSON.stringify(result, null, 4)
                return true;
            }
            let historyList = [];
            // 添加历史
            function addHisory() {
                let keys = $(".keys").value;
                let source = $(".source").value;
                let target = $(".target").innerText;
                historyList.push({keys, source, target});
                $(".history-list").innerHTML += generatorHistoryTemplate(keys, source, target);
            }
            // 历史列表模板
            function generatorHistoryTemplate(keys, source, target) {
                return  `<div class="history-item">
                    <div><span class="item-label">keys: </span><textarea class="history-keys">${keys}</textarea></div>
                    <div><span class="item-label">source: </span><textarea class="history-source">${source}</textarea></div>
                    <div><span class="item-label">target: </span><textarea class="history-target">${target}</textarea></div>
                    <div><button onClick="copyToResult(this)">作为基本目标数据</button><button onClick="copyToSource(this)">作为源数据</button></div>
                </div>`
            }
                // 复制到剪贴板
            function copy() {
                const input = document.createElement('input')
                document.body.appendChild(input)
                input.setAttribute('value', $(".target").innerText)
                input.select();
                if (document.execCommand('copy')) {
                    document.execCommand('copy')
                }
                document.body.removeChild(input)
            }
            function copyToResult(el) {
            $(".target").innerText = el.parentElement.previousElementSibling.children[1].value
            }
            function copyToSource(el) {
            $(".source").value = el.parentElement.previousElementSibling.children[1].value
            }
            on($(".copy"), copy)
            on($(".creater"), function() {
                generator() &&  addHisory()
                // $(".source").value = '';
                // $(".keys").value = '';
            })
            // 清空输入框
            on($(".clear"), function() {
                $(".keys").value = ''
                $(".source").value = ''
                $(".target").innerText = ''
                $(".prefix").value = ''
                $(".suffix").value = ''
            })
            // 清除历史
            on($(".clear-history"), function() {
                localStorage.clear(localHistoryKey)
                $(".history-list").innerHTML = ''
            })
            // auto ,使用 n-n 数字生成 
            on($('.auto'), function(){
                if(isChecked(this)) {
                    $(".prefix-box").style.display = 'block'
                    $(".suffix-box").style.display = 'block'
                } else {
                    $(".prefix-box").style.display = 'none'
                    $(".suffix-box").style.display = 'none'
                }
            }, 'change')
            // 使用规则
            on($(".rule"), function() {
                alert(
    `输入自定义keys: 如id name count(注意，键以空格分开,前后不要有空格)
    输入源数据： 从飞书文档复制即可
    点击生成`
                )
            })
            function getLocalHistory() {
                let parseList = JSON.parse(localStorage.getItem(localHistoryKey))
                return Array.isArray(parseList) ? parseList : [];
            }
            // 离开时，将历史记录保存到本地，避免丢失
            on(window, function(){
                let list = getLocalHistory();
                localStorage.setItem(localHistoryKey, JSON.stringify([...list, ...historyList]))
            }, 'beforeunload')

            // 页面加载完成，从缓存中获取历史列表
            on(window, function(){
                let list = getLocalHistory();
                $(".history-list").innerHTML = list.reduce((tol , cur) => tol + generatorHistoryTemplate(cur.keys, cur.source, cur.target), '')
            }, 'load')
        </script>
    </body>
    </html>
  ```
</details>

总体来说效率还是提高很多的