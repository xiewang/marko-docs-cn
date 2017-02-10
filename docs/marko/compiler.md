Marko 编译器
====================

Marko编译器最终输出的是兼容Node.js的标准CommonJS模块。编译好模版模块得益一个有情景感知的加载器，它可以通过使用[Lasso.js](https://github.com/lasso-js/lasso) 或者 [Browserify](https://github.com/substack/node-browserify)，很方便地传输到浏览器上运行，这就是格式化输出的优势。

`marko`模块可以自动编译已经在你服务器上加载的模版，但是你也可以预编译所有的模版。这样的话就可以作为一个版本或者测试步骤早些捕捉错误。

你可以用命令行或者JavaScript API来编译一个Marko模版文件。为了能够使用CLI，你必须先用下面的命令在全局安装`marko`模块：

```bash
npm install marko --global
```
你可以用下面的命令来编译单独的模版文件：

```bash
markoc hello.marko
```

这就会生成一个名为`hello.marko.js`的文件，位于原文件的同目录下。

你同样可以递归当前目录下的所有模版文件（`mode_modules`和`.*`目录会被默认忽略）

```bash
markoc .
```

你也可以指定多个目录或者多个文件

```bash
markoc foo/ bar/ template.marko
```

你可以添加`--clean`作为参数来删除所有已经生成的`*.marko.js`文件。例如：

```bash
markoc . --clean
```

或者，你可以用JavaScript API来编译一个模块，具体可参考下面的代码：

```javascript
require('marko/compiler').compileFile(path, function(err, src) {
    // Do something with the compiled output
});
```

# 已编译模版样例

```javascript
function create(__helpers) {
  var str = __helpers.s,
      escapeXml = __helpers.x,
      forEach = __helpers.f,
      escapeXmlAttr = __helpers.xa;

  return function render(data, out) {
    out.w('Hello ' +
      escapeXml(data.name) +
      '! ');

    if (data.colors.length) {
      out.w('<ul>');

      forEach(data.colors, function(color) {
        out.w('<li style="color: ' +
          escapeXmlAttr(color) +
          '">' +
          escapeXml(color) +
          '</li>');
      });

      out.w('</ul>');
    }
    else {
      out.w('<div>No colors!</div>');
    }
  };
}
(module.exports = require("marko").c(__filename)).c(create);
```

编译成功的文件被设计成既有非常已读的，又有最小化的，最小化的文件如下所示：

```javascript
function create(a){var d=a.ne,c=a.x,e=a.f,f=a.xa;return function(a,b){b.w("Hello "+c(a.name)+"! ");d(a.colors)?(b.w("<ul>"),e(a.colors,function(a){b.w('<li style="color: '+f(a)+'">'+c(a)+"</li>")}),b.w("</ul>")):b.w("<div>No colors!</div>")}}(module.exports=require("marko").c(__filename)).c(create);
```

_文件大小: 压缩到 223 bytes (未压缩的是 300 bytes)_

# 编译器 API

链接: [Marko Compiler - API](http://markojs.com/docs/marko/compiler/api/)

# 编译时标签

链接: [Marko Compiler - Compile-time Tags](http://markojs.com/docs/marko/compiler/compile-time-tags/)

# 编译器高级功能

链接: [Marko Compiler - Advanced](http://markojs.com/docs/marko/compiler/advanced/)