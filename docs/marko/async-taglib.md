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

- **`await:begin`** - 当 `<await>` 标签开始等待它的promise/callback时，发送一个添加了 `name`、`dataProvider` 和 `clientReorder` 键的对象。
- **`await:beforeRender`** - 当promise/callback已经返回，并且 `<await>` 准备开始渲染内容是，发送添加了 `out` （异步输出流）键的相同对象。
- **`await:error`** - 如果出现一个错误，发送一个添加了 `error` 键的相同对象。
- **`await:timeout`** - 如果超时，发送一个添加了 `timedout` （一个设为 `true` 的布尔值）键的相同对象。
- **`await:finish`** - 当 `<await>` 结束时，发送一个添加了 `finished ` （一个设为 `true` 的布尔值）键的相同对象。

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

* __`arg`__ (表达式): 提供给数据提供者函数的属性对象。
* __`arg-<arg_name>`__ (字符串): An argument to add to the `arg` object provided to the data provider function.
* __`client-reorder`__ (布尔): 如果为 `true`，await实例会被按顺序刷新到结束，运行在客户端的JavaScript被在DOM中按顺序，用来移动await实例到合适的HTML。 默认值为 `false`。
* __`error-message`__ (字符串): 数据提供者出错时的输出消息。特别指出的是，这会防止渲染溢出。
* __`name`__ (字符串): 分配到await实例的名字。用来调试的同时，也会被 `show-after` 属性使用（见下文）。 
* __`placeholder`__ (字符串): 当等待数据提供者结束时的占位符文本。只有 `client-reorder` 设为 `true` 时才生效。
* __`show-after`__ (字符串): 当 `client-reorder` 设为 `true` 时，显示这个实例的内容会被延迟，直到相关的await实例被显示出来。 
* __`timeout`__ (整形): 用这个参数来替代默认的10秒延迟。这是以毫秒为单位，所以 `timeout=40000` 会有20秒延迟。
* __`timeout-message`__ (字符串): 数据提供者超时时的输出消息。特别指出的是，这会防止渲染溢出。

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
