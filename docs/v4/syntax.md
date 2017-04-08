# 语法

Marko 的语法基于 HTML，所以你大体上已经了解它了。Marko 继承了 THML 语言并添加了一些非常棒的功能，我们会在接下来作介绍。

> **注:** Marko 同样支持[简易语法](./concise.md).

<!--
If you'd prefer to see our documentation using this syntax, just click the `switch syntax` button in the corner of any Marko code sample.-->

## 文本替换

当你渲染一个 Marko 模版的时候，你传递的输入数据会通过 `input` 在模版中得到。你可以用  `${}` 将值插入模版中：

```xml
<div>
    Hello ${input.name}
</div>
```

你实际上在这里可以传递任何 JavaScript表达式，并且这个表达式的结果会被插入到HTML输出中：

```xml
<div>
    Hello ${'world'.toUpperCase()}
</div>
```

这些值会自动转义，所以你不用担心小心插入了恶意代码。如果你需要传递非转义的 HTML，你可以使用 `$!{}`：

```xml
<div>
    Hello $!{htmlThatWillNotBeEscaped}
</div>
```

### 转义占位符

如果有需要，你可以用一个反斜杠来转移 `$`，这样的话 `$` 就会被当作文本，而不是一个占位符号：

```xml
<div>
    Placeholder example: <code>\${input}</code>
</div>
```

## 根级别文本

在模版根部的文本（在标签外部）必须用 [简易语法的 `--`](./concise.md#text) 作为前缀，以此来表明它是一个文本。解析器要以简易模式开头，否则它会尝试将你认为的文本解析成一个简易的标签声明。

```xml
-- Root level text
```

## 类型属性

Marko 关于 HTML 的一个重大改进就是提供了类型属性（而不仅仅是字符串）。

```xml
<div class=input.myClassName/>
<input type="checkbox" checked=input.isChecked/>

<tag string="Hello"/>
<tag number=1/>
<tag template-string=`Hello ${name}`/>
<tag boolean=true/>
<tag array=[1, 2, 3]/>
<tag object={hello: 'world'}/>
<tag variable=name/>
<tag function-call=input.foo()/>
```

### 属性表达式

任何 JavaScript 表达式是一个有效的属性值，它需要满足以下的条件：

_它不包含任何空格_

```xml
<tag sum=1+2 difference=3-4/>
```
```marko
tag sum=1+2 difference=3-4
```

_空格应该包含在 `()`、 `[]` 或者 `{}` 内_

```xml
<tag sum=(1 + 2) difference=(3 - 4)/>
```
```marko
tag sum=(1 + 2) difference=(3 - 4)
```

_或者，用逗号来界定属性_

```xml
<tag sum=1 + 2, difference=3 - 4/>
```
```marko
tag sum=1 + 2, difference=3 - 4
```

> **注：** 如果要使用逗号来分开两个属性，你必须使用逗号来为标签分开 _所有_ 属性。

#### 属性空格

如果选择使用空格，只能在一个属性的等号旁使用：

```xml
<tag value = 5/>
```
```marko
tag value = 5
```

### 条件属性

If an attribute value expression evaluates to `null` or `false` then the attribute is not included in the output.

```xml
<div class=(active && 'tab-active')>Hello</div>
```

With a value of `true` for `active`, the output would be the following:

```html
<div class="tab-active">Hello</div>
```

With a value of `false` for `active`, the output would be the following:

```html
<div>Hello</div>
```

### 动态属性

You can use the `${}` syntax inside an open tag to merge in the properties of an object as attributes to a tag:

_index.js_

```js
template.render({ attrs:{ class:'active', href:'https://ebay.com/' } });
```

_link.marko_

```xml
<a ${input.attrs} target="_blank">eBay</a>
```

would output the following HTML:

_output.html_
```html
<a class="active" href="https://ebay.com/" target="_blank">eBay</a>
```

### 样式属性

You can pass a string as the value of `style` just as you would in HTML, but Marko also supports passing an object as the value of the `style` attribute:

```xml
<div style={ color:'red', fontWeight:'bold' }/>
```

Output:

```html
<div style="color:red;font-weight:bold;"></div>
```

### 类属性

The `class` attribute also support object expressions or an array expressions (in addition to a string value) as shown below:

```xml
<!-- array: -->
<div class=['a', null, 'c']/>

<!-- object: -->
<div class={ a:true, b:false, c:true }/>
```

In both cases, the output will be the same:

_output.html_
```html
<div class="a c"></div>
```

## 速记属性

Marko provides a shorthand for declaring classes and ids on an element:

_source.marko_
```xml
<div.my-class/>
<span#my-id/>
<button#submit.primary.large/>
```

Yields this HTML:

_output.html_
```html
<div class="my-class"></div>
<span id="my-id"></span>
<button id="submit" class="primary large"></button>
```

## 命令

Directives are denoted by parenthesis and take an argument instead of a value.  Many directives may be used as both tags and attributes.

```xml
<if(true)>
    <strong>Marko is awesome</strong>
</if>
```

```xml
<strong if(true)>
    Marko is awesome
</strong>
```

Most directives support JavaScript expressions, and some even support multiple arguments:

```xml
<include(target, input)/>
```

Others allow a custom syntax:
```xml
<for(item in items)/>
```

Directives are used by many of our [Core Tags](./core-tags.md) for control-flow (`<if>`, `<else-if>`, `<for>`, etc.) and other features.  You can also use them in your own [Custom Tags](./custom-tags.md).

## 内联JavaScript

> **ProTip:** If you find yourself writing a lot of inline JS, consider moving it out to an external file and then [`import`](./core-tags.md#codeimportcode) it.

To execute JavaScript in your template you can insert a Javascript statement using the `$ <code>` syntax.

A line that starts with a `$` followed by a space will execute the code that follows.

```xml
$ var name = input.name;

<div>
    Hello, ${name}
    $ console.log('The value rendered was', name);
</div>
```

A statement may continue onto subsequent lines if new lines are bounded by `{}`, `[]`, `()`, ``` `` ```, or `/**/`:

```xml
$ var person = {
    name: 'Frank',
    age: 32
};
```

Multiple statements or an unbounded statement may be used by wrapping the statement(s) in a block:

```xml
$ {
    var bgColor = getRandomColor();
    var textColor = isLight(bgColor)
        ? 'black'
        : 'white';
}
```

### 静态JavaScript
> **Static:** The JavaScript code that follows `static` will run once when the template is loaded and be shared by all calls to render. It must be declared at the top level and does not have access to values passed in at render.

Inline JavaScript will run each time your template is rendered, if you only want to initialize some values once, use the `static` keyword:

```xml
static var count = 0;
static var formatter = new Formatter();

static function sum(a, b) {
    return a + b;
};

<div>${formatter.format(sum(2, 3))}</div>
```

Like inline Javascript, multiple statements or an unbounded statement may be used by wrapping the statement(s) in a block:

```xml
static {
    var base = 2;
    function sum(a, b) {
        return base + a + b;
    };
}
```

### 美元符号转义

If you need to output a `$` at the beginning of a line, you can escape it: `\$`.

```xml
<p>You can run JS in a Marko template like this:</p>
<code>
    \$ var num = 123;
</code>
```