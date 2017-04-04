# 渲染

为了渲染Marko view，你需要 `require` 它。 
To render a Marko view, you need to `require` it.

_example.js_

```js
var fancyButton = require('./components/fancy-button');
```

> **注：** 如果你在node.js中使用，为了能够引用 `.marko` 文件，你需要启动 [require扩展](./installing.md#require-marko-views)，或者你需要使用[Marko开发工具](https://github.com/marko-js/marko-devtools)预编译所有的的模版。如果你在浏览器中使用，你需要使用类似 [`lasso`](./lasso.md)、 [`webpack`](./webpack.md)、 [`browserify`](./browserify.md) 或者 [`rollup`](./rollup.md) 的打包工具

一旦你有了一个view，你就可以传递输入数据，并渲染它：

_example.js_

```js
var button = require('./components/fancy-button');
var html = button.renderToString({ label:'Click me!' });

console.log(html);
```

这个输入数据可以在view中使用 `input` 来获得，所以 `fancy-button.marko` 如下所示：

_./components/fancy-button.marko_

```xml
<button>${input.label}</button>
```

输出的HTML如下：

```html
<button>Click me!</button>
```

## 渲染方法

我们上面使用 `renderToString` 方法来渲染view，但是这里有好几个不同的方法前面可以用来渲染。

这些方法中许多都返会一个 [`RenderResult`](#renderresult)，它是一个拥有一些辅助方法的对象，用在渲染输出上。

### `renderSync(input)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据 |
| return value | [`RenderResult`](#renderresult) | 渲染结果 |

使用 `renderSync` 让渲染完全同步。如果一个标签尝试同步运行，就会抛出一个错误。

```js
var view = require('./view'); // Import `./view.marko`
var result = view.renderSync({});

result.appendTo(document.body);
```

### `render(input)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据 |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | 异步 `out` 的渲染目标 |

`render` 方法返回一个异步 `out`，用来在服务器端中或者浏览器的虚拟DOM中生成HTML。在另一种情况下，异步 `out` 有个遵循 Promises/A+ 原则的 `then` 方法，所以它可以就像 Promise 一样使用。这个 promise 返回一个 [`RenderResult`](#renderresult)。

```js
var view = require('./view'); // Import `./view.marko`
var resultPromise = view.render({});

resultPromise.then((result) => {
    result.appendTo(document.body);
});
```

### `render(input, callback)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据  |
| `callback` | `Function` | 用来在渲染结束时调用的方法 |
| callback value | [`RenderResult`](#renderresult) | 渲染结果 |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | 异步 `out` 的渲染目标 |

```js
var view = require('./view'); // Import `./view.marko`

view.render({}, (err, result) => {
    result.appendTo(document.body);
});
```

### `render(input, stream)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据  |
| `stream` | `WritableStream` | 一个可写的流 |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | 异步 `out` 的渲染目标 |

HTML输出被写入到 `stream` 中。

```js
var http = require('http');
var view = require('./view'); // Import `./view.marko`

http.createServer((req, res) => {
    res.setHeader('content-type', 'text/html');
    view.render({}, res);
});
```

### `render(input, out)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据 |
| `out` | `AsyncStream`/`AsyncVDOMBuilder` | 异步 `out` 渲染的地方 |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | 被传递的`out` |

`render` 方法同样允许传递一个已经存在的的异步 `out`。如果你这么做的话， `render` 不会自动结束这个异步的 `out`（这样会让在另个view中间来渲染一个view）。如果这个异步的 `out` 不会被其他方法结果，你需要自行结束它。

```js
var view = require('./view'); // Import `./view.marko`
var out = view.createOut();

view.render({}, out);

out.on('finish', () => {
    console.log(out.getOutput());
});

out.end();
```


### `renderToString(input)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据 |
| return value | `String` | 渲染器生成的HTML字符串 |

返回一个HTML字符串，并强制渲染同步结束。如果某个标签尝试异步只想，就会跑出一个错误。

```js
var view = require('./view'); // Import `./view.marko`
var html = view.renderToString({});

document.body.innerHTML = html;
```

### `renderToString(input, callback)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据  |
| callback value | `String` | 渲染器生成的HTML字符串 |
| return value | `undefined` | N/A |

HTML字符串会被传递到回调中。

```js
var view = require('./view'); // Import `./view.marko`

view.renderToString({}, (err, html) => {
    document.body.innerHTML = html;
});
```

### `stream(input)`

`stream` 方法返回一个node.js风格的输出HTML的流。这个方法在服务器端可用，但是默认在浏览器中不可用。如果你需要在浏览器中使用流，你可能需要将 `require('marko/stream')` 作为你客户端绑定的一部分。

```js
var fs = require('fs');
var view = require('./view'); // Import `./view.marko`
var writeStream = fs.createWriteStream('output.html');

view.stream({}).pipe(writeStream);
```

## 渲染结果

### `getComponent()`
### `getComponents(selector)`
### `afterInsert(doc)`
### `getNode(doc)`
### `getOutput()`
### `appendTo(targetEl)`
### `insertAfter(targetEl)`
### `insertBefore(targetEl)`
### `prependTo(targetEl)`
### `replace(targetEl)`
### `replaceChildrenOf(targetEl)`

## 全局数据

如果你需要给所有调用上面的渲染方法渲染的到的视图结果添加全局数据，你可以在输入数据对象中传递一个 `$global` 属性值。这个数据会从 `input` 移除，然后合并到 `out.global` 属性值中。

```js
view.render({
    $global: {
        flags: ['mobile']
    }
});
```
