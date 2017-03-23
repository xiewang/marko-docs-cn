概述
=================

Marko是eBay开源的的基于HTML模版引擎，它可以在服务器端渲染模版，也可以在浏览器端渲染。它是[相当快](https://github.com/marko-js/templating-benchmarks)并且轻量的模版引擎，同时，它也支持流和异步渲染。开发者可以通过用自定义标签和自定义属性扩展的HTML语法，来生成新的可重用的构建模块。Marko编译器生成Node.js兼容的JavaScript模块，这些模块[易读](https://gist.github.com/patrick-steele-idem/0514b480219d1c9ed8d4#file-template-marko-js)，易懂并且方便调试。不同于其他模版引擎，Marko避免使用[全局变量](http://archive.oreilly.com/pub/a/javascript/excerpts/javascript-good-parts/awful-parts.html)和全局辅助功能。

Marko的构造哲学是，基于HTML的模版语言对于生成HTML来说应该更自然、直观。因为Marko编译器可以理解HTML文档结构，模版文件中directives隐秘且更强大。Marko通过模版允许内置的JavaScript表达式，同样保持完整的JavaScript能力和灵活性。

Marko支持[逐步HTML渲染](http://markojs.com/docs/marko/async-taglib/)，它可以通过直接写流输出的方式更快地传输。Marko可以自动分配模版的异步传输部分，以便HTML可以在优化后的数个模块下被传送。因为Marko是一个异步模版语言，即便在渲染开始之后，一些额外的数据才会被异步抓取。这些特性让Marko成为一个构建高性能网址的优秀选择。

对于在客户端构建富UI组件，请参照[marko-widgets](https://github.com/marko-js/marko-widgets)项目。

<a href="http://markojs.com/try-online/" target="_blank">在线试用!</a>

# 语法

Marko既支持类似HTML的语法，也支持一个更简易的缩进式语法。两种语法是一样的。不管你选择哪一个语法，最总编译出来的代码是完全相同的。

下面几个编辑器和IDE支持语法高亮:

- Atom: [language-marko](https://atom.io/packages/language-marko)
- Sublime Text: [marko-sublime](https://github.com/merwan7/sublime-marko)
- WebStorm: [marko.tmbundle](https://github.com/marko-js/marko-tmbundle) (See: [Importing TextMate Bundles](https://www.jetbrains.com/phpstorm/help/importing-textmate-bundles.html))
- TextMate: [marko.tmbundle](https://github.com/marko-js/marko-tmbundle)

## HTML语法

```xml
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Marko Templating Engine</title>
    </head>
    <body>
        <h1>
            Hello ${data.name}!
        </h1>

        <ul if(data.colors.length)>
            <li for(color in data.colors)>
                ${color}
            </li>
        </ul>
        <div else>
            No colors!
        </div>
    </body>
</html>
```

## 简易语法

下面的简易模版和上面的模版是相等的：

```xml
<!DOCTYPE html>
html lang="en"
    head
        title - Marko Templating Engine
    body
        h1 - Hello ${data.name}!
        ul if(data.colors.length)
            li for(color in data.colors)
                ${color}
        div else
            - No colors!
```

## 混合语法

你可以在同一个文档中混合使用简易语法和HTML语法。
下面的模版和上面的模版是相等的：

```xml
<!DOCTYPE html>
html lang="en"
    head
        title - Marko Templating Engine
    body
        <h1>
            Hello ${data.name}!
        </h1>
        ul if(data.colors.length)
            li for(color in data.colors)
                ${color}
        div else
            - No colors!
```

# 演示代码

一个有文本替换、循环、条件的基本模版如下：

_hello-world.marko:_

```xml
<h2>Hello ${data.name}!</h2>
<ul if(data.colors.length)>
    <li style="color: ${color}" for(color in data.colors)>
        ${color}
    </li>
</ul>
<div else>
    No colors!
</div>
```

上面的模版可以用下面的样例代码来渲染：

```javascript
require('marko/node-require').install();

var template = require('./hello-world.marko');

template.render({
        name: 'World',
        colors: ["red", "green", "blue"]
    },
    function(err, output) {
        console.log(output);
    });
```

最终上面的程序运行后输出的内容如下（格式化成易读的）：

```html
<h2>Hello World!</h2>
<ul>
    <li style="color: red">red</li>
    <li style="color: green">green</li>
    <li style="color: blue">blue</li>
</ul>
```

为了比较，用下面的colors数组为空的数据：

```javascript
{
    name: 'World',
    colors: []
}
```

输出的内容如下：

```xml
<h2>Hello World!</h2>
<div>
    No colors!
</div>
```

流API可以用来输出到HTTP相应流或者其他可写流。比如结合Express框架：

```javascript
var template = require('./user-profile.marko');

app.get('/profile', function(req, res) {
    // Render directly to the writable HTTP output stream:
    template.render({
            name: 'Frank'
        }, res);
});
```

# 其他模版语言

大多数开发者熟悉[Handlebars](https://github.com/wycats/handlebars.js), [Dust](https://github.com/linkedin/dustjs) 或者 [Mustache](http://mustache.github.io/) 这些模版语言，那么为什么还要用Marko呢？

Marko的不同之处在于它是一个基于HTML模版语言，它能够[让javascript表达式作为属性值](https://github.com/philidem/htmljs-parser)。任何HTML文件都是一个有效的Marko模版。因为Marko能够读懂模版的HTML结构，不可能像Handlerbars,Dust或者Mustache一样是基于文本的模版语言，它支持强大的函数性。Marko允许开发者通过使用自定义的HTML元素和属性来扩张HTML语言。初次之外，利用HTML结构来应用模版指令让模版更易读，以及能够让数据模版更像最终的HTML结构。

让我们来对比一下Handlebars（一个基于文本的模版语言）：

__Handlebars:__

```xml
<h2>Hello {{name}}!</h2>

{{#if colors}}
<ul>
    {{#each colors}}
    <li class="color">
        {{this}}
    </li>
    {{/each}}
</ul>
{{else}}
<div>
    No colors!
</div>
{{/if}}
```

__Marko:__

```xml
<h2>Hello ${data.name}!</h2>
<ul if(data.colors.length)>
    <li class="color" for(color in data.colors)>
        ${color}
    </li>
</ul>
<div else>
    No colors!
</div>
```

Marko模版需要注意的一些事：

* 更少的代码行
* 动更少的代码行让模版变动态
* 仅仅改变开放的标签来完成条件判断和循环

Marko除了是一个基于HTML的模版语言，要记住它同样被设计有极高的性能和可扩展性。Marko编译器让开发者完全控制模版如何编译成JavaScript，而且运行环境被设计的尽可能的高效。为了更好的性能和灵活性，Marko完全遵循JavaScript语言（例如，支持JavaScript表达式来写自定义表达式语言）。

最后，另一个Marko的区别其他模版引擎的功能是：它支持_异步模版渲染_。这个强大的功能让部分模版异步渲染。不用等到所有的数据从远程的服务器传回来才开始渲染模版，你现在可以立刻开始渲染模版和异步模块，只要这些异步数据可用时就会被渲染。

# 设计哲学
* __可读性：__ 模版应该尽可能类似输出的HTML让模版易读。不使用含糊的语法和符号。
* __简单：__ 不应该使用大量的新概念，降低复杂性。
* __可扩展：__ 模版引擎应该在编译时和运行时都容易被扩展。
* __高性能：__ 运行环境以及编译好的代码应该需要优化，让它们降低CPU的使用以及占用更少的内存。所有的表达式都应该是原生JavaScript，避免运行时的解释。
* __没有限制：__ 无论是更少的逻辑还是更多的逻辑，开发者可以在模版中尽情发挥。
* __异步和流输出：__ 渲染HTML应该可以无序，但是输出的HTML应该有序地流出。减少空闲时间以及降低首字节时间。
* __主观性：__ 模版引擎应该尽可能少些引入让人意外的东西。
* __浏览器端和服务器端的兼容性：__ 模版编译出来的JavaScript应该在服务器端和浏览器端都可以运行。
* __可调试：__ 编译好的JavaScript模版应该可调试、易读懂。
* __编译时检查：__ 在编译时，语法、自定义标签和自定义属性应该是有效的。
* __工具的支持：__ 支持工具来完成代码的自动完善和自动验证，用于提高开发效率和安全性。
* __模块化：__ 运行环境已经编译的模版应该基于CommonJS模块来改善依赖管理。解决模版的依赖（比如自定义标签）应该是基于模版文件的系统路径，而不是依靠全局的注册。