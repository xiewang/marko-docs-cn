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

### All JavaScript expressions are validated at compile time

Previously, Marko v2 would not throw an error at compile time if a JavaScript expression was invalid. Now, with Marko v3, all JavaScript expressions are parsed and validated at compile time using [esprima](http://esprima.org/). These extra compile-time checks make development a lot easier and it prevents errors from showing up at runtime.

### Style attribute

The value of the style attribute can now resolve to an object expression (in addition to a string value) as shown below:

```xml
<div style={color: 'red', 'font-weight': 'bold'}>
```

Output:

```html
<div style="color:red;font-weight:bold">
```

[Issue #229 - Marko v3: Special case style attribute to allow object expression](https://github.com/marko-js/marko/issues/229)

### Class attribute

The value of the class attribute can now be an object expression or an array expression as shown below:

```xml
<!-- array: -->
<div class=['a', null, 'c']>

<!-- object: -->
<div class={a: true, b: false, c: true}>
```

In both cases, the output will be the same:

```html
<div class="a c">
```

[Issue #230 - Marko v3: Special case class attribute to allow object or array expression](https://github.com/marko-js/marko/issues/230)

### Dynamic attributes

___Old syntax:___

```xml
<div attrs="myAttrs"/>
```

___New syntax:___

```xml
<var myAttrs={'class': 'foo', 'style': 'background-color: red'}/>
<div ${myAttrs}/>
```

Output:

```html
<div class="foo" style="background-color: red"></div>
```

[Issue #198 - Marko v3: Replace `<div attrs(myAttrs)>` with `<div ${myAttrs}>`](https://github.com/marko-js/marko/issues/198)

### Dynamic tag names

Dynamic tag names are now supported by putting a placeholder in the tag name:

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

### Input data for custom tags:

```xml
<greeting({name: 'Frank'})/>

<!-- Equivalent to: -->
<greeting name='Frank' />
```

[Issue #173 - Marko v3: Input data object for custom tags](https://github.com/marko-js/marko/issues/173)

### Tag body content parsing

The `marko-body` attribute can be used to control how body content is parsed. The following values are supported:

- `"html"` - Body content will be parsed HTML (the default)
- `"static-text"` - Body content will be parsed as static text (HTML tags will be ignored). Placeholders will be ignored.
- `"parsed-text"` - Body content will be parsed as text (HTML tags will be ignored). Placeholders will _not_ be ignored.

```xml
<div marko-body="static-text">
    This is just one
    <span if(foo)>
            Hello ${THIS IS NOT VALID}!
    </span>
    big text block
</div>
```

_Output:_

```html
<div>
    This is just one
    <span if(foo)>
            Hello ${THIS IS NOT VALID}!
    </span>
    big text block
</div>
```

### Preserve whitespace

___Old syntax:___

```xml
<div c-space="preserve">
    All of this
    whitespace   will
    be preserved.
</div>
```

___New syntax:___

Whitespace can be preserved using the `marko-preserve-whitespace` attribute:

```xml
<div marko-preserve-whitespace>
    All of this
    whitespace   will
    be preserved.
</div>
```

### Compiler options

The `<marko-compiler-options>` tag can be used to enable whitespace preservation and/or HTML comments preservation for the entire template.

___Old syntax:___

```xml
<compiler-options whitespace="preserve" />
```

___New syntax:___

```xml
<marko-compiler-options preserve-whitespace preserve-comments />
```

[Issue #205 - Marko v3: Provide full control over whitespace](https://github.com/marko-js/marko/issues/205)
[Issue #206 - Marko v3: HTML comments should be handled correctly](https://github.com/marko-js/marko/issues/206)

### Open tag only

Marko v3 allows tags to be declared as "open tag only". If a custom tag is declared as being "open tag only" then the parser will report an error if an ending tag is found or if the tag has nested body content.

Tag definition:

```json
{
    "<my-custom-tag>": {
        "open-tag-only": true,
        ...
    }
}
```

Usage:

```xml
<!-- Allowed: -->
<my-custom-tag>
<my-custom-tag/>

<!-- Not allowed: -->
<my-custom-tag>Foo</my-custom-tag>
```

[Issue #222 - Marko v3: Allow open only tags to be defined in tag definition](https://github.com/marko-js/marko/issues/222)

### Improved readability of compiled code

A lot of attention was put on producing very clean output JavaScript code. If you ever find that you need to debug through the code of a compiled template you will find that the code is very well formatted and readable.

For example, given the following input template:

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

The compiled output will be similar to the following:

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

### Improved performance

The Marko runtime has been slightly tweaked to improve performance and it is now faster and smaller! You can find the updated benchmarks here: https://github.com/marko-js/templating-benchmarks

Here's a partial snippet of the results that shows how Marko stacks up to the competition:

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

# Removed features

## Removed: $<variable-name>

Use `${<variable-name>}` instead. We found that allowing `$` without the curly braces was problematic when parsing inline script that used the `$` variable from libraries like jQuery.

## Removed: $!<variable-name>

Use `$!{<variable-name>}` instead.

## Removed: "simple conditional" syntax

Marko v3 _removes_ support for the following "simple conditional" syntax:

___Old syntax:___

```xml
<div class="{?data.isActive; active; inactive}">
```

___New syntax:___

In Marko v3, JavaScript is favored over a new syntax so, instead, use a JavaScript conditional expression:

```xml
<div class=(data.isActive ? 'active' : 'inactive')>
```

Alternatively, when only a value is needed when `true`:

```xml
<div class=(data.isActive && 'active')>
```

## Removed: nested attributes

Nested attributes are no longer supported (likely never used):

```xml
<test-popover>
    <attr name="title">Popover Title</attr>
    <attr name="content">Popover Content</attr>

    Link Text
</test-popover>
```

## Removed: `<require>` tag

Use `<script marko-init>` instead.

___Old syntax:___

```xml
<require module="./my-include-target.marko" var="myIncludeTarget" />
```

___New syntax:___

```xml
<script marko-init>
var myIncludeTarget = require('./my-include-target.marko');
</script>
```

## Removed: `c-data`/`c-input` attribute

Custom tag data should be passed using an argument:

```xml
<my-custom-tag({name: 'Frank'})/>
```

## Removed: `c-space`/`c-whitespace` attribute

Use `marko-preserve-whitespace` attribute instead:

```xml
<div marko-preserve-whitespace>
    This whitespace
    will be preserved.
</div>
```

## Removed: `c-escape-xml` attribute

No longer applicable.

## Removed: `c-parse-body-text` attribute

Use the `marko-body="<body-type>"` attribute instead.

## Removed: `attrs` attribute

Use placeholder within open tag instead:

```xml
<div ${myAttrs}>
```

## Removed: `with` tag and attribute

Instead, use `<var>` tag with nested content to create scoped variables.

## Removed: JavaScript operator aliases

The following JavaScript operator aliases are no longer supported:

JavaScript Operator | Marko Equivalent
------------------- | -----------------
`&&`                 | `and`
<code>&#124;&#124;</code>                | `or`
`===`               | `eq`
`!==`               | `ne`
`<`                 | `lt`
`>`                 | `gt`
`<=`                | `le`
`>=`                | `ge`

Instead, you must use valid JavaScript operators. For example:

___Old syntax:___

```xml
<div if="searchResults.length gt 100">
    Show More
</div>
```

___New syntax:___

```xml
<div if(searchResults.length > 100)>
    Show More
</div>
```

## A dynamic include target must resolve to a loaded template (not a string path)

If using dynamic templates, the expression must resolve to a fully loaded template instance (not a string path). In Marko v2, the following was allowed:

```javascript
var templateData = {
    includeTarget: './include.marko'
}
```

```xml
<include template="${data.includeTarget}"/>
```

This is no longer allowed in Marko v3 and, instead, you must do the following:

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

# Other notable changes

## marko-taglib.json → marko.json

Taglib definition files must now be named `marko.json` (not `marko-taglib.json`).

In addition, the rules for resolving `marko.json` files have changed (see: [Marko v3: Improve taglib discovery #224](https://github.com/marko-js/marko/issues/224))

[Issue #216 - Marko v3: Transition from marko-taglib.json to marko.json](https://github.com/marko-js/marko/issues/216)

## Nested tags separator changed

The symbol for separating nested tags has changed from `.` (period) to `:` (colon). This was done to avoid conflicts with the new shorthand class syntax (e.g. `<div.foo>`).

___Old syntax:___

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

___New syntax:___

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

## Node.js v4+ is now required for compiling

Unlike Marko v2, Marko v3 requires Node.js v4+ to compile Marko templates. The new Marko compiler was written using ECMAScript 2015 (ES6) features that are only found in Node.js v4+. The Marko runtime, however, still works in all JavaScript runtimes.

## Case-sensitive HTML parser

The Marko parser and compiler are now case sensitive. The following tags are not equal with Marko v3:

```xml
<my-custom-tag/>
<My-CUSTOM-Tag/>
```

[Issue #215 - Marko v3: Marko should be case sensitive with tag names and attributes](https://github.com/marko-js/marko/issues/215)

<a name="compiler-api"></a>
## New compiler API

The Marko compiler went through a major refactor with Marko v3 as a result of introducing a new parser that recognizes types at compile time. The new compile-time API is much simpler and more powerful. For more information on the Marko compiler and extending Marko at compile-time, please check out:

- [Compiler Advanced](http://markojs.com/docs/marko/compiler/advanced/)
- [The Compiler API](http://markojs.com/docs/marko/compiler/api/) (work-in-progress)
- [Compile-time Tags](http://markojs.com/docs/marko/compiler/compile-time-tags/)


# Next steps

If you have ideas on how to improve Marko please let us know. We welcome new contributors so if you would like to help out please join us in the [Gitter chat room for Marko](gitter.im/marko-js/marko), [file an issue on Github](https://github.com/marko-js/marko) or send us a pull request.