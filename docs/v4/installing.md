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
This builds a `client.js` file to the newly created `static/` directory and injects the required `<script>` tags into our HTML page to load our application in the browser.  If we had css in the view then `<link>` tags would have also been added.

Load up that page in your browser and you should see `Hello Marko` staring back at you.

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

这样会在原始模版旁边生成一个 `hello.js` 文件。这个生成的 `.js` 文件会被 Node.js 运行时加载。
This will produce a `hello.js` file next to the original template. The generated `.js` file will be what gets loaded by the Node.js runtime. It is important to leave off the `.marko` extension when requiring a Marko template so that the `.js` will be resolved correctly.

If you wish to only use the require extension in development, you can conditionally require it.

```js
if (!process.env.NODE_ENV) {
    require('marko/node-require');
}
```

#### Serving a simple page

Let's update `server.js` to serve the view from an http server:

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

And give `hello.marko` some content:

_hello.marko_
```xml
<h1>Hello ${input.name}</h1>
```

Start the server (`node server.js`) and open your browser to [http://localhost:8080](http://localhost:8080) where you should see the heading `Hello Marko`.

#### Initializing server-rendered components

Marko automatically injects a list of components that need to be mounted in the browser, right before the closing `</body>` tag (as such, it required that you include a `<body>` in your rendered output).  

However, you still need to bundle the CSS & JavaScript for your page and include the proper `link`, `style`, and `script` tags.  Luckily, the `lasso` taglib will do all the heavy lifting for you.

First install `lasso` and `lasso-marko`:

```
npm install --save lasso lasso-marko
```

Next, in your page or layout view, add the `lasso-head` and `lasso-body` tags:

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

Finally, configure your server to serve the static files that `lasso` generates:

_server.js_
```js
app.use(require('lasso/middleware').serveStatic());
```
