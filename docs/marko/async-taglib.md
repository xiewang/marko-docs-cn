异步标签库
=====================

> 注：这个功能之前是用的是 `<async-fragment>`，现在被`<await>`替代。

# 样例

**给模版传递一个promise（或者一个回调）**：

```javascript
var template = require('./template.marko');
var request = require('request-promise');

module.exports = template.stream({
    userDataProvider: request.get('https://api.example.com/users/123')
});
```

**并等待数据**：

```html
<await(user from data.userDataProvider)>
    <ul>
        <li>Name: ${user.firstName} ${user.lastName}</li>
        <li>Email address: ${user.email}</li>
    </ul>
</await>
```

# 基本原理

Marko包含这么一个标签库，它支持更高效、更简洁的"Pull Model"的方法来给模版提供model数据。

* __推模型:__ 在构建试图模型之前，提前请求所有需要的数据并等待接受所有的数据，然后再渲染模版。 
* __拉模型:__ 在开始渲染模版时，立刻传递所有的异步数据提供者的方法。让模版在渲染时 _拉_ 数据。

模版渲染的Pull Model方法需要使用一个模版引擎，这个模版引擎支持异步模版渲染（例如[marko](https://github.com/marko-js/marko) 和 [dust](https://github.com/linkedin/dustjs)）。这是因为在渲染模版开始之前，不是所有的数据可能被完全取回。部分取决于那些还没获得数据的模版会用拉模型方法被异步渲染。

# 推模型对比拉模型

传统推模型方法的问题是：模版渲染会被推迟到 _所有_ 数据依据被完全收到。这会降低首字节的时间，而且当等待所有数据从远程服务加载的时候，这也会导致服务的空闲。此外，如果模版不再需要某些数据，那么只有模版需要改写，而不需要控制器也改写。

使用新的拉模型方法，模版渲染会立刻开始。此外，模版的各个部件取决于数据提供者什么被异步渲染，同时，`await` 只会数据提供者的结束状态联系在一起。模版渲染只会被真正模版需要的数据所延迟。


# 无序刷新

Marko异步标签支持无序刷新。启动无序刷新需要两个步骤：

1. 添加 `client-reorder` 属性到 `<await>` 标签：

    ```html
    <await(user from data.userDataProvider) client-reorder=true>
        <ul>
            <li>Name: ${user.firstName} ${user.lastName}</li>
            <li>Email address: ${user.email}</li>
        </ul>
    </await>
    ```

2. 添加 `<await-reorderer>` 到页面底部：

    ```html
    <html>
    ...
    <body>
        ...
        <await-reorderer/>
    </body>
    </html>
    ```

如果 `client-reorder` 为 `true`，那么一个占位符元素后被渲染到输出中，而不是最终await实例的HTML。这个实例将会被渲染到页面底部，并且客户端的JavaScript代码会将等待渲染的内容插入到DOM的合适位置。在它们被移动到相应的位置之前，`<await-reorderer>` 就是无序实例需要被渲染的地方。如果有任何的无序实例，那么内敛的JavaScript代码就会被内置到页面的这个地方，这些代码会移动DOM节点到DOM的合适位置。

# 事件

你也许需要监听这些来自模版渲染方法异步流上的事件，或者如果一个包裹的流是一个事件发射器（如：node里的http `res` 流），你也需要监听。

- **`await:begin`** - 当 `<await>` 标签开始等待它的promise/callback是，发送一个有 `name`、`dataProvider` 和 `clientReorder` 键的对象。
- **`await:beforeRender`** - emits the same object with the key `out` (the async output stream) added once the promise/callback has returned and the `<await>` tag is about to render its contents.
- **`await:error`** - emits the same object with the key `error` (the `Error`) added, if an error occurs
- **`await:timeout`** - emits the same object with the key `timedout` (a boolean set to `true`) added, if a timeout occurs
- **`await:finish`** - emits the same the key `finished` (a boolean set to `true`) added once the `<await>` tag finishes

# 标签库API

## `<await>`

**必需的参数：**
```js
<await(var Name from data.provider)>
```

* __`var`__: 当使用数据提供者的数据时，name变量会被使用 
* __`data provider`__: 需要等待的原数据。必须是下面任意一个的引用。
    - `Function(callback)`
    - `Function(args, callback)`
    - `Promise`
    - Data


**支持的属性：**

* __`arg`__ (expression): The argument object to provide to the data provider function.
* __`arg-<arg_name>`__ (string): An argument to add to the `arg` object provided to the data provider function.
* __`client-reorder`__ (boolean): If `true`, then the await instances will be flushed in the order they complete and JavaScript running on the client will be used to move the await instances into the proper HTML order in the DOM. Defaults to `false`.
* __`error-message`__ (string): Message to output if the data provider errors out.
Specifying this will prevent the rendering from aborting.
* __`name`__ (string): Name to assign to this await instance. Used for debugging purposes as well as by the `show-after` attribute (see below).
* __`placeholder`__ (string): Placeholder text to show while waiting for a data provider to complete. Only applicable if `client-reorder` is set to `true`.
* __`show-after`__ (string): When `client-reorder` is set to `true` then displaying this instance's content will be delayed until the referenced await instance is shown.
* __`timeout`__ (integer): Override the default timeout of 10 seconds with this param. Units are inmilliseconds so `timeout=40000` would give a 40 second timeout.
* __`timeout-message`__ (string): Message to output if the data provider times out. Specifying this will prevent the rendering from aborting.

## `<await-placeholder>`

当一个无序的await实例正在等待被加载时，这个标签被用来控制什么文本可以被显示。当 `client-reorder` 被设置 `true` 才能生效。

样例：

```html
<await(user from data.userDataProvider) client-reorder>
    <await-placeholder>
        Loading user data...
    </await-placeholder>

    <ul>
        <li>First name: ${user.firstName}</li>
        <li>Last name: ${user.lastName}</li>
    </ul>

</await>
```

## `<await-error>`

当数据提供者出错时，这个标签被用来控制什么文本可以被显示。

样例：

```html
<await(user from data.userDataProvider)>
    <await-error>
        An error occurred!
    </await-error>

    <ul>
        <li>First name: ${user.firstName}</li>
        <li>Last name: ${user.lastName}</li>
    </ul>
</await>
```

## `<await-timeout>`

当数据提供者超时时，这个标签被用来控制什么文本可以被显示。

样例：

```html
<await(user from data.userDataProvider)>
    <await-timeout>
        A timeout occurred!
    </await-timeout>

    <ul>
        <li>First name: ${user.firstName}</li>
        <li>Last name: ${user.lastName}</li>
    </ul>
</await>
```

## `<await-reorderer>`

所有无序await实例的容器。如果任何 `<await>` 标签的 `client-reorder` 被设为true时，那么这个标签需要加到页面模版里（一般的，需要在 `</body>` 标签里）。

样例

```html
<!DOCTYPE html>
<html>
...
<body>
    ...
    <await-reorderer/>
</body>
</html>
```
