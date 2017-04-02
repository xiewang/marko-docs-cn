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

The output HTML would be:

```html
<button>Click me!</button>
```

## Rendering methods

We used the `renderToString` method above to render the view, but there are a number of different method signatures that can be used to render.  

Many of these methods return a [`RenderResult`](#renderresult) which is an object with helper methods for working with the rendered output.

### `renderSync(input)`

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
| return value | [`RenderResult`](#renderresult) | The result of the render |

Using `renderSync` forces the render to complete synchronously.  If a tag attempts to run asynchronously, an error will be thrown.

```js
var view = require('./view'); // Import `./view.marko`
var result = view.renderSync({});

result.appendTo(document.body);
```

### `render(input)`

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | the async `out` render target |

The `render` method returns an async `out` which is used to generate HTML on the server or a virtual DOM in the browser. In either case, the async `out` has a `then` method that follows the Promises/A+ spec, so it can be used as if it were a Promise.  This promise resolves to a [`RenderResult`](#renderresult).

```js
var view = require('./view'); // Import `./view.marko`
var resultPromise = view.render({});

resultPromise.then((result) => {
    result.appendTo(document.body);
});
```

### `render(input, callback)`

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
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

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
| `stream` | `WritableStream` | a writeable stream |
| return value | `AsyncStream`/`AsyncVDOMBuilder` | the async `out` render target |

The HTML output is written to the passed `stream`.

```js
var http = require('http');
var view = require('./view'); // Import `./view.marko`

http.createServer((req, res) => {
    res.setHeader('content-type', 'text/html');
    view.render({}, res);
});
```

### `render(input, out)`

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
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

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
| return value | `String` | The HTML string produced by the render |

Returns an HTML string and forces the render to complete synchronously.  If a tag attempts to run asynchronously, an error will be thrown.

```js
var view = require('./view'); // Import `./view.marko`
var html = view.renderToString({});

document.body.innerHTML = html;
```

### `renderToString(input, callback)`

| params  | type | description |
| ------- | ---- | ----------- |
| `input` | `Object` | the input data used to render the view |
| callback value | `String` | The HTML string produced by the render |
| return value | `undefined` | N/A |

An HTML string is passed to the callback.

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

## Global data

If you need to make data available globally to all views that are rendered as the result of a call to one of the above render methods, you can pass the data as a `$global` property on the input data object.  This data will be removed from `input` and merged into the `out.global` property.

```js
view.render({
    $global: {
        flags: ['mobile']
    }
});
```
