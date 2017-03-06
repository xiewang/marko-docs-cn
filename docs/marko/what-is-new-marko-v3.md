---
---
Marko v3新功能
======================

我们在Marko v3版本中做了一些令人激动人心的改进。最主要的几点：

- 语法改进
    - 支持类HTML语法和[简洁的缩进语法](#concise-syntax)
    - [所有的属性值现在都被解析成JavaScript表达式](#attr-javascript-expressions)
    - 新的语法解析器： [htmljs-parser](https://github.com/philidem/htmljs-parser) (thanks [@philidem](https://github.com/philidem)!)
- [指令改进](#improved-directives)
- [编译器API改进](#compiler-api)

我们努力让Marko v3成为有史以来最好的模版引擎（这当然可被论证的）。这次版本囊括了超过[250个Marko提交](https://github.com/marko-js/marko/compare/v2.8.4...v3.0.0-alpha.1) 和 [大约100个解析器提交](https://github.com/philidem/htmljs-parser/commits/master)。Marko和它的新的解析器被超过600个独立测试用例严格测试。感谢一路以来，所有贡献者提供代码和反馈(参阅 [#90](https://github.com/marko-js/marko/issues/90), [#211](https://github.com/marko-js/marko/issues/211), 特别感谢 [@adammcarth](https://github.com/adammcarth), [@BryceEWatson](https://github.com/BryceEWatson), [@crsandeep](https://github.com/crsandeep), [@DanCech](https://github.com/DanCech), [@danrichman](https://github.com/danrichman), [@kristianmandrup](https://github.com/kristianmandrup), [@onemrkarthik](https://github.com/onemrkarthik), [@philidem](https://github.com/philidem), [@pswar](https://github.com/pswar), [@scttdavs](https://github.com/scttdavs), [@SunnyGurnani](https://github.com/SunnyGurnani), [@tindli](https://github.com/tindli), [@vedam](https://github.com/vedam) 和 [@yomed](https://github.com/yomed))!

为了完成这些改进，很有必要作一些突破性的改变。然而，我们提供了一个迁移工具，它可以自动将Marko V2的模版转换为新的v3语法。请参阅：[github.com/marko-js/marko-migrate](https://github.com/marko-js/marko-migrate)

同样很值得注意的是，新的简洁语法深受[Jade/Pug](http://jade-lang.com/)影响。然而，我们给Marko减少了语法规则，让简洁语法更容易理解、更接近HTML。

在Marko全力成为一流的模版引擎的同时，[Marko Widgets](http://markojs.com/docs/marko-widgets/) 目标是成为最简单、最快速的构建UI组建库。Marko负责给UI组件渲染出HTML，Marko Widgets负责添加客户端行为。Marko Widgets提供类似DOM-diffing、批量更新、状态化小组件、声明时间绑定和高效时间委托高级功能。


请使用并提供你的反馈：

```
npm install marko@3 --save
```

如果你在使用Marko Widgets，你会需要安装最新版本的Marko Widgets：
```
npm install marko-widgets@6 --save
```

同样，[Lasso.js](https://github.com/lasso-js/lasso)标签库已经更新，以兼容Marko v3，所以如果你在使用这个工具，你需要升级Lasso.js：

```
npm install lasso@2 --save
```

敬请加入[Gitter上的Marko社区](https://gitter.im/marko-js/marko)，自由提问或提供反馈。

# 新功能


## JavaScript表达式作为属性值 

Marko v3使用新的解析器来支持更多强大的语法。使用Marko v3，一个属性值会被 _永远_ 解析成JavaScript表达式：

```xml
<div class=data.myClassName>
<input type="checkbox" checked=data.isChecked/>

<my-component string="Hello"/>
<my-component number=1/>
<my-component template-string="Hello ${name}"/>
<my-component boolean=true/>
<my-component array=[1, 2, 3]/>
<my-component object={hello: 'world'}/>
<my-component variable=name/>
<my-component function-call=data.foo()/>
<my-component complex-expression=1+2/>
<my-component super-complex-expression=(data.foo() + data.bar(['a', 'b', 'c']))/>
```

以前，属性值会被默认解析成字符串，但是一个标签定义文件（AKA，一个schema文件）会被用来将类型信息和一个属性值联系在一起，以改变如何解释这个值。通过替换严格的HTML解析为更加灵活的[htmljs-parser](https://github.com/philidem/htmljs-parser)，那一个属性值如何被解释就变的很明显。

## 属性参数

Marko v3允许标签和属性可以像如下一样拥有一个 _参数_。 

```xml
<my-custom-tag(some argument)>
<div my-custom-attr(some argument)>
```

参数被用来提升指令的语法，比如循环和条件。

## 编辑器和IDE支持的提升

- Atom: [language-marko](https://atom.io/packages/language-marko)
- Sublime Text: [marko-sublime](https://github.com/merwan7/sublime-marko)
- WebStorm: [marko.tmbundle](https://github.com/marko-js/marko-tmbundle) (See: [Importing TextMate Bundles](https://www.jetbrains.com/phpstorm/help/importing-textmate-bundles.html)) (新!)
- TextMate: [marko.tmbundle](https://github.com/marko-js/marko-tmbundle)
- CodeMirror/Brackets (新!)

<a name="concise-syntax"></a>

## 简洁的缩进语法

Marko v3同时支持类HTML语法和简洁的缩进语法。两种语法同等支持，并且和在同一个Marko模版中混合使用。

下面的代码片段演示了每种语法变体的比较。

### HTML语法

```xml
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Marko Templating Engine</title>
    </head>
    <body>
        <!-- Welcome to Marko! -->
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

### 简洁语法

下面的简洁语法模版和上面的HTML模版等效：

```xml
<!DOCTYPE html>
html lang="en"
    head
        title - Marko Templating Engine
    body
        // Welcome to Marko!
        h1 - Hello ${data.name}!
        ul if(data.colors.length)
            li for(color in data.colors)
                ${color}
        div else
            - No colors!
```

这里有个"gotcha"是你需要注意的。Marko解析起会在简明的模式下进行。因此，下面的模版：

```
Hello World
Welcome to Marko
```

会有如下的输出：

```xml
<Hello World></Hello>
<Welcome to Marko></Welcome>
```

为了能正确解析，你需要在每一个HTML行加上一个连字符的前缀：

```
- Hello World
- Welcome to Marko
```

或者，一个多行HTML块可以使用分隔符来界定：

```
---
Hello World
Welcome to Marko
---
```


### 混合语法

你甚至可以在同一个文件中混合使用简洁语法和HTML语法。下面的模版和上面的效果是一样的：

```xml
<!DOCTYPE html>
html lang="en"
    head
        <title>Marko Templating Engine</title>
    body
        // Welcome to Marko!
        <h1>
            Hello ${data.name}!
        </h1>
        ul if(data.colors.length)
            li for(color in data.colors)
                ${color}
        div else
            - No colors!
```

<a name="improved-directives"></a>
## 指令语法的改进

### 宏指令

___旧语法：___

```xml
<def function="greeting(name, count)">
    Hello ${name}! You have ${count} new messages.
</def>

<invoke function="greeting" name="John" count="${10}"/>
<invoke function="greeting('Frank', 20)"/>
```

___新语法：___


```xml
<macro greeting(name, count)>
  Hello ${name}! You have ${count} new messages.
</macro>

<greeting name="Frank" count=30/>
<greeting('Frank', 30)/>
```

宏指令支持嵌套主体内容，现在它用 `<macro-body>` 得到更清晰的支持：

```xml
<macro section-heading(className)>
  <h1 class=className>
    <macro-body/>
  </h1>
</macro>

<section-heading className="foo">
  Hello World!
</section-heading>

<!-- Output: -->
<h1 class="foo">Hello World!</h1>
```

[Issue #170 - Marko v3: macros](https://github.com/marko-js/marko/issues/170)

### 引用

___旧语法：___

```xml
<include template="./include-target.marko" name="Frank" count="${20}"/>
<include template="./include-target.marko" template-data="{name: 'Frank', count: 20}"/>
```

___新语法：___

```xml
<include("./include-target.marko") name='Frank' count=20/>
<include("./include-target.marko", {name: 'Frank', count: 20})/>
```

同样支持引入主体内容：

```xml
<include("./include-nested-content.marko") name="Frank" count=20>
    Have a
    <b>
        wonderful
    </b>
    day!
</include>
```

当提供了主体内容，`data.renderBody = function(out) { ... }` 这个特别的函数就会被添加到模版数据中，用于目标模版。

同样支持动态模版：

```xml
<script marko-init>
var templateA = require('./include-a.marko');
var templateB = require('./include-b.marko');
</script>

<include(foo ? templateA : templateB) name='Frank' count=20/>
```

[Issue #178 - Marko v3: include tag](https://github.com/marko-js/marko/issues/178)

### 条件

___旧语法：___

```xml
<!-- Applied as tags: -->
<if test="a > b">...</if>
<else-if test="b < a">...</else-if>
<else>...</else>

<!-- Applied as attributes: -->
<div if="a > b">...</div>
<div else-if="b < a">...</div>
<div else>...</div>
```

___新语法：___

```xml
<!-- Applied as tags: -->
<if(a > b)>...</if>
<else-if(b < a)>...</else-if>
<else>...</else>

<!-- Applied as attributes: -->
<div if(a > b)>...</div>
<div else-if(b < a)>...</div>
<div else>...</div>
```

### for(item in items)

___旧语法___

```xml
<!-- Applied as a tag: -->
<ul>
    <for test="color in colors">
        <li>${color}</li>
    </for>
</ul>

<!-- Applied as an attribute: -->
<ul>
    <li for="color in colors">
        ${color}
    </li>
</ul>
```

___新语法___

```xml
<!-- Applied as a tag: -->
<ul>
    <for(color in colors)>
        <li>${color}</li>
    </for>
</ul>

<!-- Applied as an attribute: -->
<ul>
    <li for(color in colors)>
        ${color}
    </li>
</ul>
```

额外的选项如今会在一个 `|` 符号后被传递，如下：
```xml
<for(color in ['red', 'green', 'blue'] | separator=", ")>
    ${color}
</for>
```

Output:

```html
red, green, blue
```

- [Issue #189 - Marko v3: Improve syntax of the "for" directive by keeping everything in parens](https://github.com/marko-js/marko/issues/189)
- [Issue #193 - Marko v3: Custom iterators](https://github.com/marko-js/marko/issues/193)

#### 作为目标的迭代函数

Marko v3 如今允许目标为一个迭代函数：

```xml
<script marko-init>
function myColorsIterator(callback) {
    callback('red');
    callback('green');
    callback('blue');
}
</script>

<ul>
    <for(color in myColorsIterator)>
        <li>${color}</li>
    </for>
</ul>
```

输出：

```html
<ul>
    <li>red</li>
    <li>green</li>
    <li>blue</li>
</ul>
```

[Issue #193 - Marko v3: Allow for target to be an iterator function](https://github.com/marko-js/marko/issues/194)

### for(`<init>`; `<test>`; `<update>`)

Marko v3 如今支持原生的 for 循环，如下：

```xml
<for(var i=1; i<=3; i++)>
    ${i}
</for>
```

输出：

```html
123
```

[Issue #191 - Marko v3: Allow native JavaScript for loops](https://github.com/marko-js/marko/issues/191)

### for(key, value in object)

___旧语法：___

```xml
<for each="(name,value) in {'foo': 'low', 'bar': 'high'}">
    ${name}: ${value}
</for>
```

___新语法：___

```xml
<for(name,value in {'foo': 'low', 'bar': 'high'})>
    ${name}: ${value}
</for>
```

[Issue #175 - Marko v3: Looping over object properties](https://github.com/marko-js/marko/issues/175)

### for(`<range>`)

___旧语法：___

```xml
<for each="i from 0 to 9">
    ${i}
</for>
```

___新语法：___

```xml
<for(i from 0 to 9)>
    ${i}
</for>
```

### while(`<test>`)

Marko v3 如今支持原生的while循环：

```xml
<!-- Applied as a tag: -->
<var n=0/>
<ul>
    <while(n < 4)>
        <li>${n++}</li>
    </while>
</ul>

<!-- Applied as an attribute: -->
<var n=0/>
<ul>
    <li while(n < 4)>
        ${n++}
    </li>
</ul>
```

[Issue #228 - Marko v3: while loop support](https://github.com/marko-js/marko/issues/228)

### 引入


___旧语法：___

```xml
<require module="change-case" var="changeCase"/>
```
___新语法：___

Marko v3 如今支持添加JavaScript初始化代码在编译好的模版顶部。这些代码只会在第一次编译好的模版文件加载好时加载，它可以被用来引用其他JavaScript模块和引入新的静态变量。

```xml
<script marko-init>
var reverse = require('./helpers').reverse;
var changeCase = require('change-case');
</script>
```

[Issue #214 - Marko v3: `<script marko-init>`](https://github.com/marko-js/marko/issues/214)

### 变量


___旧语法___

```xml
<var name="foo" value="'bar'" />
<var name="count" value="0" />
<assign var="count" value="count+1" />
```

___新语法：___

```xml
<var foo="bar" count=0/>
<assign count=count+1/>
```

范围变量同样支持：

```xml
<var name="Frank">
    Hello ${name}!
</var>
<!-- The "name" variable will be `undefined` here -->
```

感谢[@BryceEWatson](https://github.com/BryceEWatson)对该功能作出的贡献！

[Issue #169 - Marko v3: var tag](https://github.com/marko-js/marko/issues/169)
[Issue #171 - Marko v3: assign tag](https://github.com/marko-js/marko/issues/171)

### 脚本

Marko v3 已经让脚本支持更加通用的 `<% ... %>` 语法。如果你需要直接添加JavaScript代码到编译好的模版中，你可以用到该功能。

___旧语法：___

```xml
{% if (true) { %}
    HELLO
{% } %}
{% if (false) { %}
    WORLD
{% } %}
```

___新语法：___

```xml
<% console.log('Hello World'); %>

<% if (true) { %>
    HELLO
<% } %>
<% if (false) { %>
    WORLD
<% } %>
```

[Issue #181 - Marko v3: Scriptlets](https://github.com/marko-js/marko/issues/181)

### 布局标签库

___旧语法：___

```xml
<layout-use template="./layout-default.marko" show-header="$false">
    <layout-put into="body">BODY CONTENT</layout-put>
    <layout-put into="footer">FOOTER CONTENT</layout-put>
</layout-use>
```

___新语法：___

`<layout-use>` 如今需要一个模版参数：

```xml
<layout-use("./layout-default.marko") show-header=false>
    <layout-put into="body">BODY CONTENT</layout-put>
    <layout-put into="footer">FOOTER CONTENT</layout-put>
</layout-use>

<!-- The layout template can also be dynamic and you can also pass a data object: -->
<layout-use(data.layoutDynamic, {showHeader: false})>
    ...
</layout-use>
```

[Issue #209 - Marko v3: Re-introduce support for the layout taglib](https://github.com/marko-js/marko/issues/209)

### Invoke标签

___旧语法：___

```xml
<invoke function="test('World')"/>
<invoke function="console.log('Hello World')"/>
```

___新语法：___

```xml
<invoke test('World') />
<invoke console.log('Hello World')/>
```

[Issue #179 - Marko v3: invoke tag](https://github.com/marko-js/marko/issues/179)

## 空的关闭标签：

下面的标签用空的标签关闭的方法如今是可用的：

```xml
<my-custom-tag>
    Hello world!
</>
```

[`htmljs-parser` - Issue #30 - Allow an empty closing tag ](https://github.com/philidem/htmljs-parser/issues/30)

## 速记ID和类名

指定的IDs和类名十分常见，所以Marko v3引入了速记语法用来匹配CSS选择器语法，如下：

- `#foo` ➔ `<div id="foo">`
- `#foo.bar` ➔ `<div id="foo" class="bar">`
- `.bar` ➔ `<div class="bar">`

```xml
#section
    ul.colors
        li.color - red
        li.color - green
        li.color - blue
    button#submitButton.enabled - Submit Form
```

输出：

```html
<div id="section">
    <ul class="colors">
        <li class="color">red</li>
        <li class="color">green</li>
        <li class="color">blue</li>
    </ul>
    <button id="submitButton" class="enabled">
        Submit Form
    </button>
</div>
```

速记语法同样在可以在冗长的HTML语法中使用：

```xml
<#section>
    <ul.colors>
        <li.color>red</li>
        <li.color>green</li>
        <li.color>blue</li>
    </ul.colors>
    <button#submitButton.enabled>
        Submit Form
    </button>
</>
```

[Issue #220 - Marko v3: Support expansion of CSS selector shorthand for tag names](https://github.com/marko-js/marko/issues/220)
[htmljs-parser - Issue #24 - Expand CSS selector shorthand](https://github.com/philidem/htmljs-parser/issues/24)

## 其他的改进

### 验证解析器

Marko v3的新的解析器不在允许不匹配的开始标签和结束标签。新的解析起会给出一个错误问题，而不是让它在原始的模版中通过解析。例如下面的模版：
```xml
<div>Hello World</foo>
```

你会得到一个友好性的错误信息：

```text
The closing "foo" tag does not match the corresponding opening "div" tag
```

### 所有JavaScript表达式在编译时验证 

以前，如果JavaScript表达式不正确，Marko v2在编译时不会抛出错误。现在，所有的JavaScript表达式都会在编译时被Marko v3验证，这是通过使用[esprima](http://esprima.org/)来完成。这些额外的编译时验证让开发起来更加简单，它能够防止运行时抛错。

### 样式属性

样式属性的值如今可以时一个对象表达式（除了是字符串之外），如下：

```xml
<div style={color: 'red', 'font-weight': 'bold'}>
```

输出：

```html
<div style="color:red;font-weight:bold">
```

[Issue #229 - Marko v3: Special case style attribute to allow object expression](https://github.com/marko-js/marko/issues/229)

### 类属性

类属性如今可以是一个对象表达式或者一个数字表达式，如下：

```xml
<!-- array: -->
<div class=['a', null, 'c']>

<!-- object: -->
<div class={a: true, b: false, c: true}>
```

上面两个都会生成下面的的代码：

```html
<div class="a c">
```

[Issue #230 - Marko v3: Special case class attribute to allow object or array expression](https://github.com/marko-js/marko/issues/230)

### 动态属性

___旧语法：___

```xml
<div attrs="myAttrs"/>
```

___新语法：___

```xml
<var myAttrs={'class': 'foo', 'style': 'background-color: red'}/>
<div ${myAttrs}/>
```

输出：

```html
<div class="foo" style="background-color: red"></div>
```

[Issue #198 - Marko v3: Replace `<div attrs(myAttrs)>` with `<div ${myAttrs}>`](https://github.com/marko-js/marko/issues/198)

### 动态标签名

如今支持动态标签名，它是通过在标签名中添加一个占位符来完成：

```xml
<${foo ? 'div' : 'span'}>
    Hello World!
</>
```

Output:

```xml
<!-- If foo is true: -->
<div>Hello World!</div>

<!-- If foo is false: -->
<span>Hello World!</span>
```

[Issue #226 - Marko v3: Allow placeholders in tag name](https://github.com/marko-js/marko/issues/226)

### 自定义标签中添加输入数据：

```xml
<greeting({name: 'Frank'})/>

<!-- Equivalent to: -->
<greeting name='Frank' />
```

[Issue #173 - Marko v3: Input data object for custom tags](https://github.com/marko-js/marko/issues/173)

### 标签主体内容解析

`marko-body`属性可以用来控制主体内容如何被解析。支持下面几种值：

- `"html"` - 主体内容会被解析成HTML（默认）
- `"static-text"` - 主体内容会被解析成静态文本（HTML标签会被忽略）。占位符会被忽略。
- `"parsed-text"` - 主体内容会被解析成文本（HTML标签会被忽略）。占位符 _不会_ 被忽略。

```xml
<div marko-body="static-text">
    This is just one
    <span if(foo)>
            Hello ${THIS IS NOT VALID}!
    </span>
    big text block
</div>
```

_输出：_

```html
<div>
    This is just one
    <span if(foo)>
            Hello ${THIS IS NOT VALID}!
    </span>
    big text block
</div>
```

### 保留空白符

___旧语法：___

```xml
<div c-space="preserve">
    All of this
    whitespace   will
    be preserved.
</div>
```

___新语法：___

空白符会通过使用 `marko-preserve-whitespace` 属性被保留：
```xml
<div marko-preserve-whitespace>
    All of this
    whitespace   will
    be preserved.
</div>
```

### 编译选项：

`<marko-compiler-options>`标签可以用来保留空白符或者用来在整个模版中保留注释：

___旧语法：___

```xml
<compiler-options whitespace="preserve" />
```

___新语法：___

```xml
<marko-compiler-options preserve-whitespace preserve-comments />
```

[Issue #205 - Marko v3: Provide full control over whitespace](https://github.com/marko-js/marko/issues/205)
[Issue #206 - Marko v3: HTML comments should be handled correctly](https://github.com/marko-js/marko/issues/206)

### Open tag only

Marko v3允许标签被声明为 "open tag only"。如果一个自定义标签被声明成 "open tag only"，那么当遇到一个结束标签或者这个标签有内嵌内容时，解析器会报告一个错误。

Tag definition:

```json
{
    "<my-custom-tag>": {
        "open-tag-only": true,
        ...
    }
}
```

使用方法：

```xml
<!-- Allowed: -->
<my-custom-tag>
<my-custom-tag/>

<!-- Not allowed: -->
<my-custom-tag>Foo</my-custom-tag>
```

[Issue #222 - Marko v3: Allow open only tags to be defined in tag definition](https://github.com/marko-js/marko/issues/222)

### 编译好的代码可读性的改进

花费了很大精力在如何生成非常干净的JavaScript代码。如果你有发现你需要调试编译好的模版，这些代码被格式化的非常好并且容易读懂。

例如，下面的输入模版：

```xml
<my-custom-tag name="World"/>

<ul if(data.colors.length)>
    <li for(color in data.colors)>
        ${color}
    </li>
</ul>
<div else>
    No colors!
</div>
```

编译好的输出会类似下面的内容：

```javascript
function create(__helpers) {
  var str = __helpers.s,
      empty = __helpers.e,
      notEmpty = __helpers.ne,
      escapeXml = __helpers.x,
      __loadTag = __helpers.t,
      my_custom_tag = __loadTag(require("./components/my-custom-tag/renderer")),
      forEach = __helpers.f;

  return function render(data, out) {
    my_custom_tag({
        name: "World"
      }, out);

    if (data.colors.length) {
      out.w("<ul>");

      forEach(data.colors, function(color) {
        out.w("<li>" +
          escapeXml(color) +
          "</li>");
      });

      out.w("</ul>");
    } else {
      out.w("<div>No colors!</div>");
    }
  };
}

(module.exports = require("marko").c(__filename)).c(create);
```

### 性能的提升

Marko运行环境已经被稍微作了啦些性能改进，现在它更加快速和轻量！你可以在这里找到更新过的基准测试程序数值：https://github.com/marko-js/templating-benchmarks

这里是结果的部分片段，展示了Marko和其他模版的比较：

| Template Engine | Results                      |
|-----------------|------------------------------|
| __marko__       | __187,729 op/s (fastest)__   |
| dot             | 183,161 op/s (2.43% slower)  |
| handlebars      | 104,634 op/s (44.26% slower) |
| dust            | 83,773 op/s (55.38% slower)  |
| swig            | 54,866 op/s (70.77% slower)  |
| jade            | 32,929 op/s (82.46% slower)  |
| nunjucks        | 32,306 op/s (82.79% slower)  |
| react           | 3,651 op/s (98.06% slower)   |

# 删除的功能

## 已删除: `$<variable-name>`

使用 `${<variable-name>}` 作为替代.当类似jQuery这样的库使用 `$` 来解析行脚本时， 我们发现没有花括弧的 `$` 是很麻烦的事。 

## 已删除: `$!<variable-name>`

Use `$!{<variable-name>}` instead.

## 已删除: “简单条件”语法 

Marko v3 _删除了_对下面“简单条件”语法的支持：

___旧语法：___

```xml
<div class="{?data.isActive; active; inactive}">
```

___新语法：___

在Marko v3中，JavaScript青睐新的语法，使用一个Javascript条件表达式：

```xml
<div class=(data.isActive ? 'active' : 'inactive')>
```

相应的，当为 `true` 时只需要一个值：

```xml
<div class=(data.isActive && 'active')>
```

## 已删除: 嵌套属性

嵌套属性不在被支持（很庆幸它也从来没呗使用过）：

```xml
<test-popover>
    <attr name="title">Popover Title</attr>
    <attr name="content">Popover Content</attr>

    Link Text
</test-popover>
```

## 已删除: `<require>` tag

用 `<script marko-init>` 作为替代。

___旧语法：___

```xml
<require module="./my-include-target.marko" var="myIncludeTarget" />
```

___新语法：___

```xml
<script marko-init>
var myIncludeTarget = require('./my-include-target.marko');
</script>
```

## 已删除: `c-data`/`c-input` 属性

自定义标签数据应该用一个参数传递：

```xml
<my-custom-tag({name: 'Frank'})/>
```

## 已删除: `c-space`/`c-whitespace` 属性

用 `marko-preserve-whitespace` 属性作为替代：

```xml
<div marko-preserve-whitespace>
    This whitespace
    will be preserved.
</div>
```

## 已删除: `c-escape-xml` 属性

不再可用。

## 已删除: `c-parse-body-text` 属性

用 `marko-body="<body-type>"` 属性作为替代。

## 已删除: `attrs` 属性

使用有个开标签的占位符作为替代：

```xml
<div ${myAttrs}>
```

## 已删除: `with` 标签和属性

相应的，用有嵌套内容 `<var>` 标签来新建范围变量。

## 已删除: JavaScript操作符别名

下面的JavaScript操作符别名不再使用：

JavaScript操作符     | Marko 等价体
------------------- | -----------------
`&&`                 | `and`
<code>&#124;&#124;</code>                | `or`
`===`               | `eq`
`!==`               | `ne`
`<`                 | `lt`
`>`                 | `gt`
`<=`                | `le`
`>=`                | `ge`

相应的，你必须使用有效的JavaScript操作符，例如：
Instead, you must use valid JavaScript operators. For example:

___旧语法：___

```xml
<div if="searchResults.length gt 100">
    Show More
</div>
```

___新语法：___

```xml
<div if(searchResults.length > 100)>
    Show More
</div>
```

## 一个动态引用对象一定是一个已经加载的模版（而不是一个字符串路径）

如果使用动态模版，表达式肯定是一个完全加载好的模版实例（而不是一个字符串路径）。在Marko v2中，下面的方法是被允许的：

```javascript
var templateData = {
    includeTarget: './include.marko'
}
```

```xml
<include template="${data.includeTarget}"/>
```

这个在Marko v3不再被允许，你必须像下面一样做才行：

```javascript
var includeTemplate = require('./include.marko');

// ...

var templateData = {
    includeTarget: includeTemplate
}
```

```xml
<include(data.includeTarget})/>
```

# 其他值得记录的改变

## marko-taglib.json → marko.json

标签库的定义文件现在必须是 `marko.json` （而不是`marko-taglib.json`）。

此外，`marko.json` 文件的规则也已经变了（参阅：[Marko v3: Improve taglib discovery #224](https://github.com/marko-js/marko/issues/224)）

[Issue #216 - Marko v3: Transition from marko-taglib.json to marko.json](https://github.com/marko-js/marko/issues/216)

## 嵌套标签分割符的改变

分割嵌套标签的符号已经从 `.` 变成了 `:`。这样做是为了避免和速记类标签的冲突（例如：`<div.foo>`）。
The symbol for separating nested tags has changed from `.` (period) to `:` (colon). This was done to avoid conflicts with the new shorthand class syntax (e.g. `<div.foo>`).

___旧语法___

```xml
<tabs>
    <tabs.tab title="Tab 1">
        Content for tab 1
    </tabs.tab>
    <tabs.tab title="Tab 2">
        Content for tab 2
    </tabs.tab>
</tabs>
```

___新语法：___

```xml
<tabs>
    <tabs:tab title="Tab 1">
        Content for tab 1
    </tabs:tab>
    <tabs:tab title="Tab 2">
        Content for tab 2
    </tabs:tab>
</tabs>
```

[Issue #219 - Marko v3: Use ":" instead of "." for nested tags](https://github.com/marko-js/marko/issues/219)

## 需要使用Node.js v4+来编译

不想 Marko v2， Marko v3需要Node.js v4+的支持，来编译Marko模版。新的Marko编译器使用了ECMAScript 2015 (ES6)的功能来编写，这些功能只有在 Node.js v4+中才有。无论如何，Marko运行环境仍然在所有JavaScript运行环境中可运行。

## 大小写明感的解析器

Marko解析起和编译器如今大小写明感。下面的标签在Marko v3中不一样：

```xml
<my-custom-tag/>
<My-CUSTOM-Tag/>
```

[Issue #215 - Marko v3: Marko should be case sensitive with tag names and attributes](https://github.com/marko-js/marko/issues/215)

<a name="compiler-api"></a>
## 新的编译API

Marko编译器贯穿了Marko v3的重构，因此引入了一个新的解析器用来在编译时识别类型。新的编译时API更加简单和强大。更多关于Marko编译器和Marko编译时的扩展相关的信息，请参阅下面：

- [Compiler Advanced](http://markojs.com/docs/marko/compiler/advanced/)
- [The Compiler API](http://markojs.com/docs/marko/compiler/api/) (正在做)
- [Compile-time Tags](http://markojs.com/docs/marko/compiler/compile-time-tags/)


# 后续

如果你有如何提高Marko的建议，请随时告知我们。我们欢迎新的合作者，如果你愿意提供帮助，请在 [Gitter chat room for Marko](gitter.im/marko-js/marko), [file an issue on Github](https://github.com/marko-js/marko) 加入我们，或者给我们发pull request。