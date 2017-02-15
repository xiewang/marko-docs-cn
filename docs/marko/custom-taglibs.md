自定义标签库
==============

# 标签渲染器

每个标签应该映射到一个带有 `render(input, out)` 方法的对象。这个渲染方法就是一个带有两个参数的方法：`input` 和 `out`。`input` 参数是一个给渲染器用的任意对象，它包含输入数据。`out` 参数是包含一个输出流的 [asynchronous writer](https://github.com/marko-js/async-writer)。输出可能会用到 `out.write(someString)`，因此当实现一个标签渲染器的时候，Marko没有用到类层次或者相关的代码。一个简单标签渲染器可参考下面代码：

```javascript
exports.render = function(input, out) {
    out.write('Hello ' + input.name + '!');
}
```

如果，仅仅是如果，一个标签内有嵌套的内容，那么一个特殊的 `renderBody` 方法会被加到 `input` 对象中。如果一个渲染器想要渲染这个嵌套的主体内容，那么它就必须调用这个 `renderBody` 方法。如下：

```javascript
exports.render = function(input, out) {
    out.write('BEFORE BODY');
    if (input.renderBody) {
        input.renderBody(out);
    }
    out.write('AFTER BODY');
}
```

对于Marko Widgets的用户来说： 引用 `input.renderBody` 和用标签中的 `w-body` 属性是一样的（结合 `getInitialBody()` 生命周期方法；参考 [getInitialBody()](https://github.com/marko-js/marko-widgets#getinitialbodyinput-out)）。

一个标签渲染器应该通过创建一个`marko.json`，映射到一个自定义标签，这个 `marko.json` 会在下面几部分中介绍。

# marko.json

## 简单的标签

```json
{
    "tags": {
        "my-hello": {
            "renderer": "./hello-renderer",
            "attributes": {
                "name": "string"
            }
        }
    }
}
```

Marko同样支持简略写法来声明标签和属性。下面这个 `marko.json` 和上面的 `marko.json` 是等价的。

```json
{
    "<my-hello>": {
        "renderer": "./hello-renderer",
        "@name": "string"
    }
}
```

接下来的文档都会用简略的写法。

# 定义标签

标签会通过在 `marko.json` 添加 `"<tag_name>": <tag_def>` 属性来定义。

```json
{
    "<my-hello>": {
        "renderer": "./hello-renderer",
        "@name": "string"
    },
    "<my-foo>": {
        "renderer": "./foo-renderer",
        "@*": "string"
    },
    "<my-bar>": "./path/to/my-bar/marko-tag.json",
    "<my-baz>": {
        "template": "./baz-template.marko"
    },
}
```

每个标签应该结合一个渲染器或者一个模版。当一个自定义标签在模版中使用时，渲染器（或者模版）会在渲染阶段被引用，用来生成HTML输出。如果自定义标签指向的是一个 `marko-tag.json` 字符串，那么这个目标 `marko-tag.json` 会被加载进来用于定义标签。

# 定义属性

如果定义一些属性的话，Marko编译器会校验以确保定义的属性是被支持的。通配符(`"@*"`)允许通过任意的属性。下面是定义属性的样例：

_多属性：_

```javascript
{
    "@message": "string",     // String
    "@my-data": "expression", // JavaScript expression
    "@*": "string"            // Everything else will be added to a special "*" property
}
```

# 扫描标签

通过下面的介绍的一些约定，让Marko支持目录扫描，可以让标签库的维护变得更容易：

* 当编译一个模版时，Marko可以从模版文件目录开始搜索，直到包含 `components/` 目录的的项目根目录为止（类似寻找 `node_modules/` 一样）
* `components/` 目录包含为每个标签设置的单独目录
* 一个标签一个目录
* 所有的标签目录应该是 `components/` 直接子目录
* 标签目录的名字应该和标签的名字相同
* 每个标签目录必须包含一个 `renderer.js` 文件，这个文件用来渲染标签，或者包含一个 `template.marko`文件
* 每个标签目录可以包含一个 `marko-tag.json` 文件或者标签的定义可以内嵌在 `renderer.js` 中

参照下面的目录结构：
* __components/__
    * __my-foo/__
        * template.marko
    * __my-bar/__
        * renderer.js
        * marko-tag.json
* __page/__
    * __components/__
        * __my-hello/__
            * renderer.js
    * __page.marko__

下面的三个标签在渲染 `page.marko` 的时候可以看到：

* `<my-hello>`
* `<my-foo>`
* `<my-bar>`

文件夹扫描仅支持一个目录一个标签，并且它只会在一个层级深度的所有目录下进行。标签的定义可以内嵌在 `renderer.js` 中，也可以放在一个独立的 `marko-tag.json` 文件中。例如：

_在 `renderer.js`中:_

```javascript
exports.tag = {
    "@name": "string"
}
```

_在 `marko-tag.json`中:_

```javascript
{
    "@name": "string"
}
```

_注: 没有必要一定要声明 `renderer` ，因为扫描器会自动使用 `renderer.js` 作为渲染器。_

## 设置标签库目录

你可以在你的 `marko.json` 标签中声明 `tags-dir` 值来设置目录名：

```json
{
    "tags-dir": "./components"
}
```

如果你有多个文件夹来组织标签库，`tags-dir` 同样可以用数组来支持该功能：

```json
{
    "tags-dir": ["./components", "./modules"]
}
```
_注：如果一个 `marko.json` 已经存在，那么 `components/` 文件夹不会被自动探测到_

### 排除目录

通过排除一个目录，Marko不再搜索 `components/` 目录或者 `marko.json` 文件。

```js
require('marko/compiler/taglib-finder').excludeDir('./')
```

# 嵌套标签

通常情况下，给标签设置父／子 或者 祖先／子孙关系是很有必要的。比如：

```xml
<ui-tabs orientation="horizontal">
    <ui-tabs:tab title="Home">
        Content for Home
    </ui-tabs:tab>
    <ui-tabs:tab title="Profile">
        Content for Profile
    </ui-tabs:tab>
    <ui-tabs:tab title="Messages">
        Content for Messages
    </ui-tabs:tab>
</ui-tabs>
```

嵌套标签可以父标签的 `marko-tag.json` 文件定义，如下：

___ui-tabs/marko-tag.json___

```json
{
    "@orientation": "string",
    "@tabs <tab>[]": {
        "@title": "string"
    }
}
```

这样的话，通过使用 `<ui-tabs:tab>` 标签或者把tabs作为 `tabs` 的属性（例如 `<ui-tabs tabs="[tab1, tab2, tab3]"`），可以获得一个 `tabs`。这个嵌套的 `<ui-tabs:tab>` 标签可以让渲染用来作为父标签 `<ui-tabs>` 的 `tabs` 值的一部分。由于 `<tab>[]` 的后缀 `[]`，这个tabs值将会是数组类型，而不是单个对象。这就是说，`[]` 后缀被用来声明：嵌套标签是可以重复的。下面是获得嵌套标签的渲染器例子：

___ui-tabs/renderer.js___

```javascript
var template = require('./template.marko');

exports.renderer = function(input, out) {
    var tabs = input.tabs;

    // Tabs will be in the following form:
    // [
    //     {
    //         title: 'Home',
    //         renderBody: function(out) { ... }
    //     },
    //     {
    //         title: 'Profile',
    //         renderBody: function(out) { ... }
    //     },
    //     {
    //         title: 'Messages',
    //         renderBody: function(out) { ... }
    //     }
    // ]
    console.log(tabs.length); // Output: 3

    template.render({
        tabs: tabs
    }, out);

};
```

最后，这个用来渲染 `<ui-tabs>` 组件的模版和下面实现时类似的：

___ui-tabs/template.marko___

```xml
<div class="tabs">
    <ul class="nav nav-tabs">
        <li class="tab" for(tab in data.tabs)>
            <a href="#${tab.title}">
                ${tab.title}
            </a>
        </li>
    </ul>
    <div class="tab-content">
        <div class="tab-pane" for(tab in data.tabs)>
            <invoke tab.renderBody(out)/>
        </div>
    </div>
</div>
```

下面是用没有重复的嵌套标签的例子：

```xml
<ui-overlay>
    <ui-overlay:header class="my-header">
        Header content
    </ui-overlay:header>

    <ui-overlay:body class="my-body">
        Body content
    </ui-overlay:body>

    <ui-overlay:footer class="my-footer">
        Footer content
    </ui-overlay:footer>
</ui-overlay>
```

用于 `<ui-overlay>` 标签的 `marko-tag.json` 类似如下：

___ui-overlay/marko-tag.json___

```json
{
    "@header <header>": {
        "@class": "string"
    },
    "@body <body>": {
        "@class": "string"
    },
    "@footer <footer>": {
        "@class": "string"
    }
}
```

用于 `<ui-overlay>` 标签的渲染器类似如下：

```javascript
var template = require('./template.marko');

exports.renderer = function(input, out) {
    var header = input.header;
    var body = input.body;
    var footer = input.footer;

    // NOTE: header, body and footer will be of the following form:
    //
    // {
    //     'class': 'my-header',
    //     renderBody: function(out) { ... }
    // }

    template.render({
        header: header,
        body: body,
        footer: footer
    }, out);

};
```

最后，这个渲染 `<ui-overlay>` 标签的样例模版如下：

```xml
<div class="overlay">
    <!-- Header -->
    <div class="overlay-header ${data.header['class']}" if(data.header)>
        <invoke data.header.renderBody(out)/>
    </div>

    <!-- Body -->
    <div class="overlay-body ${data.body['class']}" if(data.body)>
        <invoke data.body.renderBody(out)/>
    </div>

    <!-- Footer -->
    <div class="overlay-footer ${data.footer['class']}" if(data.footer)>
        <invoke data.footer.renderBody(out)/>
    </div>
</div>
```

# 标签库 Discovery

下面的模版文件，`marko` 会通过搜索相关的模版文件，自动找到所有的标签。标签库发现程序会自动引入所有的在 `package.json` 中依赖包里的所有标签库。

作为例子，给定一个在 `/my-project/src/pages/login/template.marko` 下的模版和一个类似下面的 `/my-project/package.json` 文件：

```json
{
    "name": "my-package",
    "dependencies": {
        "foo": "1.0.0"
    },
    "devDependencies": {
        "bar": "1.0.0"
    }
}
```

搜索路径如下所示：

1. `/my-project/src/pages/login/marko.json`
2. `/my-project/src/pages/marko.json`
3. `/my-project/src/marko.json`
4. `/my-project/marko.json`
5. `/my-project/node_modules/foo/marko.json`
6. `/my-project/node_modules/bar/marko.json`

如果你从marko.json发现程序中隐藏特定的目录或者node_module，你可以通过下面的代码来从搜索域中排除：

```javascript
    require('marko/compiler').taglibFinder.excludeDir(dirPath);
    // Where 'dirPath' is an absolute path to the folder containing marko.json

    require('marko/compiler').taglibFinder.excludePackage(packageName);
    // Where 'packageName' is the name of the node_module containing marko.json
```

这些声明应该在marko渲染进程过程触发之前使用。
