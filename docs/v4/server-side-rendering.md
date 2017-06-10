# 服务器端渲染

Marko 允许任何 Marko／UI 组件在服务端或者浏览器端渲染。一个页面可以渲染成一个  `可写` 的流作为HTTP的响应流，如下所示：

```js
var template = require('./template'); // Import ./template.marko

module.exports = function(req, res) {
    res.setHeader('Content-Type', 'text/html; charset=utf-8');
    template.render({ name: 'Frank' }, res);
};
```

Marko 也能提供 `可读` 的流。

```js
var template = require('./template'); // Import ./template.marko

module.exports = function(req) {
    // Return a Readable stream for someone to do something with:
    return template.stream({ name: 'Frank' });
};
```

> **提示:** Marko 也提供了和一些服务端框架集成方案：
> 
> - [express](/docs/express)
> - [hapi](/docs/hapi)
> - [koa](/docs/koa)

## Bootstrapping 组件

当一个在服务端渲染好的页面加载到浏览器中的时候，Marko可以很容易地自动探测到，并创建以及挂载到浏览器中，同事赋以正确的 `state` 和 `input`。

### Bootstrapping: Lasso

如果你使用 [Lasso.js](https://github.com/lasso-js/lasso)，那么 bootstrapping 会在通过 `<lasso-body>` 标签引入 JavaScript 时自动执行。一个典型的HTML页面结构如下：

_routes/index/template_

```xml
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Marko + Lasso</title>

        <!-- CSS includes -->
        <lasso-head/>
    </head>
    <body>
        <!-- Top-level UI component: -->
        <app/>

        <!-- JS includes -->
        <lasso-body/>
    </body>
</html>
```

> **提示:** 我们提供了一些样例应用来帮组你使用 Marko + Lasso
> 
> - [marko-lasso](https://github.com/marko-js-samples/marko-lasso)
> - [ui-components-playground](https://github.com/marko-js-samples/ui-components-playground)


### Bootstrapping: Non-Lasso

如果 JavaScript 模块绑定没有使用到 Lasso，那么你需要在浏览器中添加一些客户端代码来完成应用的启动，你需要做到如下几点：

1. Load/import/require 所有在服务器端渲染的 UI 组件（加载顶层的 UI 组件通常足够了） 
2. 调用 `require('marko/components').init()`

例如，如果 `client.js` 是你客户端应用的入口：

_routes/index/client.js_
```js
// Load the top-level UI component:
require('./components/app/index');

// Now that all of the JavaScript modules for the UI component have been
// loaded and registered we can tell marko to bootstrap/initialize the app

// Initialize and mount all of the server-rendered UI components:
require('marko/components').init();
```

> **提示:** 我们提供一些样例应用来帮你使用：
> 
> - [marko-webpack](https://github.com/marko-js-samples/marko-webpack)
> - [marko-browserify](https://github.com/marko-js-samples/marko-browserify)
> - [marko-rollup](https://github.com/marko-js-samples/marko-rollup)
