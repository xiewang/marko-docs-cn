# 安装

## 尝试Marko

如果你只想在浏览器中玩玩，到我们的[在线试用](https://markojs.com/try-online)功能里看看。你可以在你的浏览器中直接开发一个Marko应用。

## 建立新的应用

如果你打算从零开始开发，[`marko-devtools`](https://www.npmjs.com/package/marko-devtools) 提供一个启动应用能让你很快上手。如下：

```bash
npm install marko-devtools --global
marko create hello-world
cd hello-world
npm install # or yarn
npm start
```

## 直接使用方法

### 安装

Marko编译器在[Node.js](https://nodejs.org/)中运行，它可使用[npm](https://www.npmjs.com/package/marko/tutorial)来安装：

```
npm install marko --save
```

或者用[yarn](https://yarnpkg.com)来安装：

```
yarn add marko
```

### 在浏览器中

比如我们有个简单的view要在浏览器中渲染：

_hello.marko_

```xml
<h1>Hello ${input.name}</h1>
```

首先，让我们创建引用该view的 `client.js` 文件，他将view渲染到body中：

_client.js_

```js
var helloComponent = require('./hello');

helloComponent.renderSync({ name:'Marko' })
    .appendTo(document.body);
```

我们也会创建一个原始的HTML页面来托管你的应用：

_index.html_

```
<!doctype html>
<html>
<head>
    <title>Marko Example</title>
</head>
<body>

</body>
</html>
```

现在，我们需要捆绑这些用于浏览器中的文件。我们使用一个叫 [`lasso`](https://github.com/lasso-js/lasso) 的工具来做这个工作，所以先要安装有关lasso的Marko插件：
```
npm install --global lasso-cli
npm install --save lasso-marko
```

现在我们就可以构建我们的的浏览器绑定了：

```
lasso --main client.js --plugins lasso-marko --inject-into index.html
```

这里会创建一个 `client.js` ，放到新建的 `static/` 文件夹下，并将需要的 `<script>` 标签库注射到HTML页面中，用来在浏览器中加载我们的应用。如果我们的view需要css，那么 `<link>` 标签库同样会被添加进来。

在浏览器加载这个页面，你应该会看到 `Hello Marko` 呈现在你面前。

### 在服务器中

#### 引用 Marko views

Marko 提供了一个自定义Node.js引用扩展，它让你就像引用一个标准的JavaScript模块一样 `require` Marko views。例如下面的  `server.js`：

_hello.marko_

```xml
<div>
    Hello ${input.name}!
</div>
```

_server.js_

```js
// The following line installs the Node.js require extension
// for `.marko` files.  This should be called once near the start
// of your application before requiring any `*.marko` files.
require('marko/node-require');

var fs = require('fs');

// Load a Marko view by requiring a .marko file:
var hello = require('./hello');
var out = fs.createWriteStream('hello.html', { encoding: 'utf8' });
hello.render({ name: 'Frank' }, out);
```

使用Node.js引用扩展完全是可选功能。如果你宁愿不使用该扩展，那么你需要使用[Marko DevTools](https://github.com/marko-js/marko-devtools)预编译所有的Marko模版：

```bash
marko compile hello.marko
```

这样会在原始模版旁边生成一个 `hello.js` 文件。这个生成的 `.js` 文件会被 Node.js 运行时加载。引用Marko模版的时候应该不加 `.marko` 后缀，这样的话 `.js` 会被正确解析，这一点很重要。

如果你希望只在开发环境中引用扩展，你可以按条件引用。

```js
if (!process.env.NODE_ENV) {
    require('marko/node-require');
}
```

#### 提供一个简单页面

让我们从一个http服务器中为view更新 `server.js` 文件：

_server.js_

```js
// Allow requiring `.marko` files
require('marko/node-require');

var http = require('http');
var hello = require('./hello');
var port = 8080;

http.createServer((req, res) => {
    // let the browser know html is coming
    res.setHeader('content-type', 'text/html');

    // render the output to the `res` output stream
    hello.render({ name:'Marko' }, res);
}).listen(port);
```

并给 `hello.marko` 添加一些内容：

_hello.marko_

```xml
<h1>Hello ${input.name}</h1>
```

启动服务(`node server.js`)，然后在浏览器中打开[http://localhost:8080](http://localhost:8080)，你会看到标题 `Hello Marko`。

#### 初始化服务队渲染组建

Marko 自动注入了一系列的需要挂载到浏览器中的组件，这些组件就在结束标签 `</body>` 之前（像这样的话，就需要在你的渲染输出中包含一个 `<body>`）。

然而，你仍需要为你的页面绑定CSS和JavaScript，并包括合适的 `link`、`style`和`script` 标签。幸运的是，`lasso` 标签库会为你完成所有这些繁重的任务。

首先需要安装 `lasso` 和 `lasso-marko`：

```
npm install --save lasso lasso-marko
```

然后，在你的页面活着布局view中，添加 `lasso-head` 和 `lasso-body` 标签：

_layout.marko_

```xml
<!doctype>
<html>
<head>
    <title>Hello world</title>
    <lasso-head/>
</head>
<body>
    <include(input.body)/>
    <lasso-body/>
</body>
</html>
```

最后，配置你的服务器，以达到可以为这些 `lasso` 生成的静态文件服务的目的：

_server.js_

```js
app.use(require('lasso/middleware').serveStatic());
```
