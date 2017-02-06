入门
===========
<!--{TOC}-->
# 安装

首先在项目中安装 `marko`，你可以执行下面这行命令：

```bash
npm install marko --save
```

为了能够使用`markoc`cli命令来编译marko模版文件，你需要用到下面的这行命令：

```bash
npm install marko --global
```

# 模版载入

Marko 提供了基本的CommonJS模块引入方式，他可以其他像标准的JavaScript模块一样引入到Node.js中。例如：

```javascript
// The following line installs the Node.js require extension
// for `.marko` files. Once installed, `*.marko` files can be
// required just like any other JavaScript modules.
require('marko/node-require').install();

// ...

// Load a Marko template by requiring a .marko file:
var template = require('./template.marko');
```

_注1：这个扩展被挂在Node.js的模块加载系统中，用来自动把`.marko`模版文件编译成CommonJS JavaScript模块，这个编译的过程会在该模版被第一次引用的时候完成，然后已经加载的`Template`实例会通过刚编译的模版模块暴露出来。实质上，`require('marko/node-require').install()` 的执行是利用`require.extensions[extension] = function markoExtension(module, filename) { ... }` 来注册这个必需的Node.js扩展。_

_注2：这个必需的扩展至需要安装一次，当然，安装多次也不会造成什么影响。例如，app本身已经注册了这个扩展，它的依赖包也同时注册了。该扩展可以分配合适的`marko`引擎模块给相应的模版使用，所以一个应用中是可以共存多个版本的`marko`的。_

如果你不想用该扩展来自动编译Marko，你也可以用下面的代码来完成相同的工作：

```javascript
var template = require('marko').load(require.resolve('./template.marko'));
```

一个加载好的Marko模版可以用多种方式来渲染，下面将具体介绍。

# 模版渲染

## 回调 API

```javascript
var template = require('./template.marko');

template.render({
        name: 'Frank',
        count: 30
    },
    function(err, output) {
        if (err) {
            console.error('Rendering failed');
            return;
        }

        console.log('Output HTML: ' + output);
    });
```

## 流 API

```javascript
var template = require('./template.marko');
var out = require('fs').createWriteStream('index.html', {encoding: 'utf8'});

// Render the template to 'index.html'
template.stream({
        name: 'Frank',
        count: 30
    })
    .pipe(out);
```

或者，你可以直接渲染一个已经存在的流，从而避免去新建一个中间流：

```javascript
var template = require('./template.marko');
var out = require('fs').createWriteStream('index.html', {encoding: 'utf8'});

// Render the template to 'index.html'
template.render({
        name: 'Frank',
        count: 30
    }, out);
```

注：这会终止这个目标输出流。

## 同步渲染 API

如果你知道你渲染的的模版不需要异步渲染，你可以用同步渲染的API来把模版渲染成字符串：

```javascript
var template = require('./template.marko');

var output = template.renderSync({
        name: 'Frank',
        count: 30
    });

console.log('Output HTML: ' + output);
```

## 异步渲染 API

```javascript
var fs = require('fs');
var template = require('./template.marko');

var out = marko.createWriter(fs.createWriteStream('index.html', {encoding: 'utf8'}));

// Render the first chunk asynchronously (after 1s delay):
var asyncOut = out.beginAsync();
setTimeout(function() {
    asyncOut.write('BEGIN ');
    asyncOut.end();
}, 1000);

// Render the template to the original writer:
template.render({
        name: 'World'
    },
    out);

// Write the last chunk synchronously:
out.write(' END');

// End the rendering out
out.end();
```

虽然第一块渲染是异步的，上述程序可以按照正确的顺序以流的方式输出到`index.html`中：

```html
BEGIN Hello World! END
```

更多细节，请看关于[async-writer](https://github.com/marko-js/async-writer) 模块的文档。

# 模版热加载

在开发过程中，不用重新启动服务就可以映射已经加载好的模版是一件十分方便的事。Marko支持服务端的模版热加载，但是这个功能必须要开启。如何开启热加载功能可以参考下面的应用事例：[marko-js-samples/marko-hot-reload](https://github.com/marko-js-samples/marko-hot-reload)