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

如果一个属性值表达式等于 `null` 或者 `false`，那么这个属性不会出现在输出中。

```xml
<div class=(active && 'tab-active')>Hello</div>
```

`active` 的值为 `true`的话，会有如下的输出：

```html
<div class="tab-active">Hello</div>
```

`active` 的值为 `false` 的话，会有如下的输出：
```html
<div>Hello</div>
```

### 动态属性

你可以在打开标签中使用  `${}` 来讲一个对象的值合并作为标签的属性值：

_index.js_

```js
template.render({ attrs:{ class:'active', href:'https://ebay.com/' } });
```

_link.marko_

```xml
<a ${input.attrs} target="_blank">eBay</a>
```
上面的会输出如下的 HTML：

_output.html_

```html
<a class="active" href="https://ebay.com/" target="_blank">eBay</a>
```

### 样式属性

就像在HTML中一样，你可以给 `style` 传递一个字符串作为值，但是 Marko 同样支持给 `style` 属性传递一个对象作为值：

```xml
<div style={ color:'red', fontWeight:'bold' }/>
```

输出：

```html
<div style="color:red;font-weight:bold;"></div>
```

### 类属性

`class` 属性同样支持对象表达式或者数组表达式（除了字符串值以外），如下：

```xml
<!-- array: -->
<div class=['a', null, 'c']/>

<!-- object: -->
<div class={ a:true, b:false, c:true }/>
```

上面两种情况，都会有下面相同的输出：

_output.html_

```html
<div class="a c"></div>
```

## 速记属性

Marko 提供了一个速记方法来在元素中声明class和id：

_source.marko_

```xml
<div.my-class/>
<span#my-id/>
<button#submit.primary.large/>
```

生成下面的HTML：

_output.html_

```html
<div class="my-class"></div>
<span id="my-id"></span>
<button id="submit" class="primary large"></button>
```

## 指令

指令由圆括号来表示，并带有一个参数，而不是一个值。许多指令既可以作为标签用，也可作为属性用。

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

大多数指令支持 JavaScript 表达式，并且甚至有些指出多个参数：

```xml
<include(target, input)/>
```

其他的允许自定义语法：

```xml
<for(item in items)/>
```

指令会在许多我们的 [核心标签](./core-tags.md) 中用来控制流 (`<if>`、 `<else-if>`、 `<for>` 等) 和其他功能。你同样可以在你自己的 [自定义标签中](./custom-tags.md) 使用。

## 内联JavaScript

> **提示：** 如果你发现写了很多的内联 JS，那么你需要考虑将它移到一个外部文件中，并  [`import`](./core-tags.md#codeimportcode) 它。

为了在你的模版中执行 JavaScript，你可以使用  `$ <code>` 语法掺入一段 JavaScript 语句。

每一行以 `$` 开头，接着跟着一个空格，后面的代码就会被执行。

```xml
$ var name = input.name;

<div>
    Hello, ${name}
    $ console.log('The value rendered was', name);
</div>
```

如果新的一行被 `{}`、 `[]`、 `()`、 ``` `` ``` 或者 `/**/` 绑定在一起，那么该语句就会继续存在在这些随后的行中：

```xml
$ var person = {
    name: 'Frank',
    age: 32
};
```

多个语句或一个无界的语句可以绑在一个块中：

```xml
$ {
    var bgColor = getRandomColor();
    var textColor = isLight(bgColor)
        ? 'black'
        : 'white';
}
```

### 静态JavaScript
> ** static：** 当模版加载后，`static` 后面的 JavaScript 代码会执行一次，并被所有的调用渲染时共享使用。它必须在顶层声明，不可以访问渲染器中传递的值。
内联 JavaScript 在你的模版渲染的每一次都会执行，如果你只想初始化一次一些值，可以使用 `static` 关键字：

```xml
static var count = 0;
static var formatter = new Formatter();

static function sum(a, b) {
    return a + b;
};

<div>${formatter.format(sum(2, 3))}</div>
```

像内联 JavaScript 一样，多行语句或者一个无界的语句可以绑定在一个块中：

```xml
static {
    var base = 2;
    function sum(a, b) {
        return base + a + b;
    };
}
```

### 美元符号转义

如果你需要在一行的开始输出 `$`，你可以将它转义：`\$`：

```xml
<p>You can run JS in a Marko template like this:</p>
<code>
    \$ var num = 123;
</code>
```