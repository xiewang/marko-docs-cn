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
| return value | `AsyncStream`/`AsyncVDOMBuilder` | 异步渲染结果 |

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
| `callback` | `Function` | a function to call when the render is complete |
| callback value | [`RenderResult`](#renderresult) | The result of the render |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | the async `out` render target |

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
| `stream` | `WritableStream` | a writeable stream |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | the async `out` render target |

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
| `out` | `AsyncStream`/`AsyncVDOMBuilder` | The async `out` to render to |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | The `out` that was passed |

The `render` method also allows passing an existing async `out`.  If you do this, `render` will not automatically end the async `out` (this allows rendering a view in the middle of another view).  If the async `out` won't be ended by other means, you are responsible for ending it.

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
| return value | `String` | The HTML string produced by the render |

Returns an HTML string and forces the render to complete synchronously.  If a tag attempts to run asynchronously, an error will be thrown.

```js
var view = require('./view'); // Import `./view.marko`
var html = view.renderToString({});

document.body.innerHTML = html;
```

### `renderToString(input, callback)`

| 参数  | 类型 | 描述 |
| ------- | ---- | ----------- |
| `input` | `Object` | 渲染view的输入数据  |
| callback value | `String` | The HTML string produced by the render |
| return value | `undefined` | N/A |

一个HTML被传递到回调中。

```js
var view = require('./view'); // Import `./view.marko`

view.renderToString({}, (err, html) => {
    document.body.innerHTML = html;
});
```

### `stream(input)`

The `stream` method returns a node.js style stream of the output HTML.  This method is available on the server, but is not available by default in the browser.  If you need to use streams in the browser, you may `require('marko/stream')` as part of your client-side bundle.

```js
var fs = require('fs');
var view = require('./view'); // Import `./view.marko`
var writeStream = fs.createWriteStream('output.html');

view.stream({}).pipe(writeStream);
```

## RenderResult

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

If you need to make data available globally to all views that are rendered as the result of a call to one of the above render methods, you can pass the data as a `$global` property on the input data object.  This data will be removed from `input` and merged into the `out.global` property.

```js
view.render({
    $global: {
        flags: ['mobile']
    }
});
```
