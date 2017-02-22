浏览器端渲染
======================

下面的代码会被用来在浏览器端渲染一个模版：

_run.js_:
```javascript
var template = require('./hello.marko');

template.render({
        name: 'John'
    },
    function(err, output) {
        document.body.innerHTML = output;
    });
```

你可以绑定上面的程序运行，在浏览器中运行可以用 [Lasso.js](https://github.com/lasso-js/lasso) (recommended) 或者 [browserify](https://github.com/substack/node-browserify)。



# 使用 Lasso.js

`lasso` 命令被用来生成资源包，这些资源包包括所有的应用模块和所有想关的Marko模版文件，使用类似下面的命令：

```bash
# 首先安装lasso和lass-marko插件
npm install lasso --global
npm install lasso-marko

lasso --main run.js --name my-page --plugins lasso-marko
```

这样就会得到一个名为 `build/my-page.html.json` 的JSON文件，这个文件包含HTML标记，这歌HTML标记碑用来包含页面优化得来的JavaScript和CSS。

同样，你可以把HTML标记插入到一个静态HTML文件，使用类似下面的命令：

```bash
lasso --main run.js --name my-page --plugins lasso-marko --inject-into my-page.html
```


# 使用 Browserify

`markoify` 从browserify转变而来，它必须被启动，用来自动编译和包含相关的Marko模版文件。

```bash
# 从npm安装 markoify：
npm install markoify --save

# 构建浏览器资源包：
browserify -t markoify run.js > browser.js
```