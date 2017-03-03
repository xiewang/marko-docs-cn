语言指南
==============

<!--{TOC}-->

# 模版指令概述

几乎所有的Marko模版指令都可以被用来作为属性，或者作为元素使用。例如：

_作为属性使用：_

```xml
<!-- Colors available -->
<ul if(colors.length)>
    <li for(color in colors)>
        ${color}
    </li>
</ul>
<!-- No colors available-->
<div else>
    No colors!
</div>
```

_作为元素使用：_

```xml
<!-- Colors available -->
<if(colors.length)>
    <ul>
        <for(color in colors)>
            <li>
                ${color}
            </li>
        </for>
    </ul>
</if>

<!-- No colors available -->
<else>
    <div>
        No colors!
    </div>
</else>
```

使用元素来控制结构逻辑的劣势在于它改变了嵌套的元素，这样就会影响阅读性。基于此，通常把指令作为属性使用更合适。

# 文本替换

动态文本可支持使用 `${<javascript-expression>}`。

样例：
```xml
Hello ${data.name}!
Hello ${data.name.toUpperCase()}!
```

默认的，所有特殊的HTML字符在动态文本中会被转义，这是为了方式跨域攻击（XSS）。为了使转义实效，你可以使用 `$!`，具体代码如下所示：

```xml
Hello $!{data.name}! <!-- Do not escape -->
```

如果必须要转义，你可以反斜杠来转移  `$`，让它可以作为一个文本而不是占位符记号被信任：

```xml
Test: \${hello}
<!-- Rendered Output:
Test: ${hello}
-->
```

# 属性

所有属性值都被解析成 _JavaScript 表达式_。此外，占位符（`${<javascript-expression>}`）既允许用单引号，也可以用双引号。

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

此外，Marko对 `class` 和 `style` 属性有特别的支持，具体如下：

## Style属性

style属性的值可以是一个对象表达式（还有一个字符串值），如下：

```xml
<div style={color: 'red', 'font-weight': 'bold'}>
```

输出：

```html
<div style="color:red;font-weight:bold">
```

## Class属性

class属性的值可以是一个对象表达式，也可以是一个数组表达式（还有一个属性值），如下：

```xml
<!-- array: -->
<div class=['a', null, 'c']>

<!-- object: -->
<div class={a: true, b: false, c: true}>
```

上面两种情况都会输出如下结果：

```html
<div class="a c">
```

# 表达式

无论在什么地方使用表达式，它们都会被视作JavaScript表达式，并会被原模原样得输出到编译好的模版中。例如：

```xml
<div if(searchResults.length > 100)>
    Show More
</div>
```

# 引用

Marko支持 引用/partials。其他Mako文件可以可以用 `<include>` 标签和一个相对路径被引用。例如：

```xml
<include("./greeting.marko") name="Frank" count=30/>
```

相应的，你可以通过提供一个JavaScript表达式作为第二参数，传递模版数据给引用的模版：

```xml
<include("./greeting.marko", {name: "Frank", count: 30})/>
```

一个动态JavaScript表达式也可以提供用来作为第一参数提供给模版，这个表达式是一个加载好的模版，如下：

在你的JavaScript控制器中：

```javascript
var myIncludeTarget = require('./my-include-target.marko');
var anotherIncludeTarget = require('./another-include-target.marko');

template.render({
		myIncludeTarget: myIncludeTarget,
		anotherIncludeTarget: anotherIncludeTarget
	},
	...);
```

然后在你的模版中：

```xml
<include(data.myIncludeTarget) name="Frank" count=30/>
<include(data.anotherIncludeTarget) name="Frank" count=30/>
```

你也可以选择通过如下调用模版，来加载引用：

```xml
<script marko-init>
var myIncludeTarget = require('./my-include-target.marko');
</script>
...
<include(data.myIncludeTarget) name="Frank" count=30/>
```

## 应用静态文明

```xml
<include-text('./foo.txt')/>
```

注：特殊的HTML字符会被转义。如果你不想转义，可以使用 `<include-html>` 标签（见下文）

## 加载静态HTML

```xml
<include-html('./foo.html')/>
```

注：特殊的HTML字符_不会_被转义，因为这个文件会被认为是一个HTML文件。

# 变量

输入的数据传递到模版可以通过使用特殊的 `data` 变量来完成。你可能会定义你自己的变量，类似下面的示例代码：

```xml
<var name=data.name.toUpperCase()/>
```

为了给一个已经存在的变量分配一个新的值，可以 `<assign>` 标签完成，类似下面的示例代码：

```xml
<assign name=data.name.toLowerCase()/>
```

`<var>`指令也可以被用来新建作用域内的变量，类似下面的示例代码：

```xml
<var nameUpper=data.name.toUpperCase() nameLower=data.name.toLowerCase()>
    Hello ${nameUpper}!
    Hello ${nameLower}!
</var>
<!--
nameUpper and nameLower will be undefined here
-->
```

# 条件句

## if...else-if...else

任何元素或者HTML片段都可以用如下的指令来变得有条件：

-	`if`
-	`else-if`
-	`else`

*作为属性使用：*

```xml
<!-- Simple if -->
<div if(someCondition)>
    Hello World
</div>
<!-- Simple unless -->
<div unless(someCondition)>
    Hello World
</div>

<!-- Complex if -->
<div if(test === "a")>
    A
</div>
<div else-if(test === "b")>
    B
</div>
<div else-if(test === "c")>
    C
</div>
<div else>
    Something else
</div>

<!-- Complex unless -->
<div unless(test === "a")>
    A
</div>
<div else-if(test === "b")>
    B
</div>
<div else>
    Something else
</div>
```

*作为元素使用：*

```xml
<!-- Colors available -->
<!-- Simple if -->
<if(someCondition)>
    <div>
        Hello World
    </div>
</if>

<!-- Complex if -->
<if(test === "a")>
    <div>
        A
    </div>
</if>
<else-if(test === "b")>
    <div>
        B
    </div>
</else-if>
<else-if(test === "c")>
    <div>
        C
    </div>
</else-if>
<else>
    <div>
        Something else
    </div>
</else>
```

## unless...else-if...else

当条件是否定的时候，`unless` 指令同样可支持用来作为 `if` 的替代品。

```xml
<!-- Simple unless -->
<div unless(someCondition)>
    Hello World
</div>

<!-- Complex unless -->
<div unless(test === "a")>
    A
</div>
<div else-if(test === "b")>
    B
</div>
<div else>
    Something else
</div>
```

*作为元素使用：*

```xml
<!-- Simple unless -->
<unless(someCondition)>
    <div>
        Hello World
    </div>
</unless>

<!-- Complex unless -->
<unless(test === "a")>
    <div>
        A
    </div>
</unless>
<else-if(test === "b")>
    <div>
        B
    </div>
</else-if>
<else-if(test === "c")>
    <div>
        C
    </div>
</else-if>
<else>
    <div>
        Something else
    </div>
</else>
```

## 速记条件

在属性里活着允许写表达式的地方，常规的JavaScript可被用来获得速记条件值：


```xml
<div class=(active && 'tab-active')>Hello</div>
```

`active` 的值为 `true`时，输出如下内容：

```html
<div class="tab-active">Hello</div>
```

`active` 的值为 `false`时，输出如下内容：

```html
<div>Hello</div>
```

_注：如果属性值表达式得到 `null` 或者 `false`，那么输出中不会包含该属性。_

三元条件同样可以使用：

```xml
<div class=(active ? 'tab-active' : 'tab-inactive')>Hello</div>
```

`active` 的值为 `false`时，输出如下内容：

```html
<div class="tab-inactive">Hello</div>
```

## 条件属性

当属性的值为一个表达式时，Marko支持条件属性。Marko同样支持 [HTML `boolean` attributes](https://html.spec.whatwg.org/#boolean-attributes) (例如， `<input type="checkbox" checked>`)，如果属性值为 `null`、 `undefined` 或者 `false`，那么这个属性不会被渲染。如果属性值为 `true`，那么只有这个属性名会被渲染。

例如，给定下面的数据：

```javascript
{
    active: true,
    checked: false,
    disabled: true
}
```

和下面的模版：

```xml
<div class=(data.active && "tab-active")/>

<input type="checkbox" checked=data.checked disabled=data.disabled/>
```

输出的HTML如下所示：

```html
<div class="tab-active"></div>

<input type="checkbox" disabled>
```

# 循环

## for

通过使用 `for` 元素，任何元素可以在一个数组的每一个值中被重复。这个指令即可用作元素，也可以用作属性。

_作为属性使用：_

```xml
<ul>
    <li for(item in items)>${item}</li>
</ul>
```

_作为元素使用：_

```xml
<ul>
    <for(item in items)>
        <li>${item}</li>
    </for>
</ul>
```


给定如下一系列值：

```javascript
["red", "green", "blue"]
```

会有如下输出：

```html
<ul>
    <li>red</li>
    <li>green</li>
    <li>blue</li>
</ul>
```

### 循环状态变量

在你需要知道当前循环索引的时候，`for` 指令同样支持循环状态变量。例如：

```xml
<ul>
    <li for(color in colors | status-var=loop)>
        ${color}
        ${loop.getIndex()+1}) of ${loop.getLength()}
        <if(loop.isFirst())> - FIRST</if>
        <if(loop.isLast())> - LAST</if>
    </li>
</ul>
```

### 循环分割器

```xml
<for(color in colors | separator=", ")>${color}</for>
<div>
    <span for(color in colors | separator=", ") style="color: ${color}">
        ${color}
    </span>
</div>
```

### 范围循环

可用如下格式来提供一个返回； `<var-name> from <from> to <to>[ step <step>]`.

`from`、 `to` 和 `step` 值必须是数字表达式。如果没有特殊情况，步长默认为1。

```xml
<ul>
    <li for(i from 0 to 10)>
        ${i}
    </li>
</ul>
```

```xml
<ul>
    <li for(i from 0 to 10 step 2)>
        ${i}
    </li>
</ul>
```

```xml
<ul>
    <li for(i from 0 to myArray.length-1)>
        ${myArray[i]}
    </li>
</ul>
```


### 值循环

```xml
<ul>
    <li for(name,value in settings)>
        <b>${name}</b>: ${value}
    </li>
</ul>
```

### 原生JavaScript的for循环

```xml
<for(var i=1; i<=3; i++)>
    ${i}
</for>
```

### 自定义迭代器

一个自定义迭代器功能可以作为视图模型的一部分被传递到模版中，用来控制数据的循环。


一个实现逆向循环一个数组功能的迭代器样例如下：

```javascript
{
    reverseIterator: function(arrayList, callback) {
        for(var i=arrayList.length-1; i>=0; i--){
            callback(arrayList[i]);
        }
    }
}
```

如何在模版中使用自定义迭代器，如下所示：

_作为 `for` 属性的一部分使用：_

```xml
<div for(item in ['a', 'b', 'c'] | iterator=data.reverseIterator)>
    ${item}
</div>
```

Output:

```html
<div>c</div>
<div>b</div>
<div>a</div>
```

_作为 `for` 元素的一部分使用：_

```xml
<for(item in ['a', 'b', 'c'] | iterator=data.reverseIterator)>
    ${item}
</for>
```

```html
cba
```

自定义迭代器同样支持提供一个自定义状态对象给每个循环迭代使用：

```javascript
{
    reverseIterator: function(arrayList, callback){
        var statusVar = {first: 0, last: arrayList.length-1};
        for(var i=arrayList.length-1; i>=0; i--){
            statusVar.index = i;
            callback(arrayList[i], statusVar);
        }
    }
}
```

_作为 `for` 属性的一部分使用：_

```xml
<div for(item in ['a', 'b', 'c'] | iterator=data.reverseIterator status-var=status)>
    ${status.index}${item}
</div>
```

Output:

```html
<div>2c</div>
<div>1b</div>
<div>0a</div>
```

_作为 `for` 元素的一部分使用：_

```xml
<for(item in ['a', 'b', 'c'] | iterator=data.reverseIterator status-var=status)>
    ${status.index}${item}
</for>
```

Output:

```html
2c1b0a
```

## while

通过使用 `while` 指令，任何元素可以被不断重复，直到满足一个条件为止。这个指令可以被当作元素使用，也可以被当作属性使用。

_作为属性使用：_

```xml
<var n=0/>
<ul>
    <li while(n < 4)>
        ${n++}
    </li>
</ul>
```

_作为元素使用：_

```xml
<var n=0/>
<ul>
    <while(n < 4)>
        <li>${n++}</li>
    </while>
</ul>

```

上面两种情况都会输出如下的代码：

```html
<ul>
    <li>0</li>
    <li>1</li>
    <li>2</li>
</ul>
```

# 宏指令

参数化的宏指令涉及到一个HTML模版的重用部分。一个宏指令可以通过 `<macro>` 指令来定义。

## macro

`<macro>` 指令可以用来定义一个模版的可重用功能。
The `<macro>` directive can be used to define a reusable function within a template.

```xml
<macro greeting(name, count)>
    Hello ${name}! You have ${count} new messages.
</macro>
```

上面的宏指令可以作为任何表达式的一部分被调用。相应的，[`<invoke>`](#invoke)指令通过使用命名属性，可以被用来调用宏指令功能。下面的样例模版演示了如何在表达式中使用宏指令功能：

```xml
<macro greeting(name, count)>
    Hello ${name}! You have ${count} new messages.
</macro>
<p>
    <greeting("John", 10)/>
</p>
<p>
    <!-- Or, using named attributes: -->
    <greeting name="Frank" count=20/>
</p>
```


# 结构化操作

## 动态属性

`attrs` 属性允许在运行的时候，属性被动态地添加到一个元素中。attrs属性的值应该是一个有内容的对象表达式，它对应了动态的属性。例如：

```xml
<var myAttrs={style: "background-color: #FF0000;", "class": "my-div"} />

<div ${myAttrs}>
    Hello World!
</div>
```

输出：

```html
<div style="background-color: #FF0000;" class="my-div">
    Hello World!
</div>
```

## body-only-if

如果你发现你封装元素是有条件的，但是它的body应该永远被渲染，那么你就应该使用 `body-only-if` 属性来处理这种情况。例如，如果有一个有效的URL，为了渲染一个封装的 `<a>` 标签，你应该像下面这么做：

```xml
<a href=data.linkUrl body-only-if(!data.linkUrl)>
    Some body content
</a>
```

给定一个 `"http://localhost/"` 值给 `data.linkUrl` 变量，会输出如下的结果：

```xml
<a href="http://localhost/">
    Some body content
</a>
```

给定一个 `undefined` 值给 `data.linkUrl` 变量，会输出如下结果：

```xml
Some body content
```

# 注释

标准的HTML注释可以被用来添加注释到你的模版中。HTML注释不会出现在渲染好的模版中。

注释样例：

```xml
<!-- This is a comment that will not be rendered -->
<h1>Hello</h1>
```

如果你想让你的HTML注册可以显示在最终的输出中，那么你可以用时这个 `html-comment` 自定义标签：

```xml
<html-comment>This is a comment that *will* be rendered</html-comment>
<h1>Hello</h1>
```

输出：

```xml
<!--This is a comment that *will* be rendered-->
<h1>Hello</h1>
```

# 空格

Marko编译器会机遇一些内置的规则，默认去除不必要的空格。这些规则，一部分是基于浏览器的规范化空格使用，一部是基于完美地使用最小化缩进标记的目标。这些规则如下：

- 第一个自元素前面的文本 ： `text.replace(/^\n\s*/g, '')`
- 最后一个自元素后面的文本 ： `text.replace(/\n\s*$/g, '')`
- 自元素之间的文本 ： `text.replace(/^\n\s*$/g, '')`
- 任何相邻的一系列控制字符会缩进味一个单独的空白字符 

另外，下面的标签的空格是默认保留的：

- `<pre>`
- `<textarea>`
- `<script>`

样例模版：

```xml
<div>
    <a href="/home">
        Home
    </a>
    <a href="/Profile">
        My   Profile
    </a>
    <textarea>
Hello
World</textarea>
</div>
```

输出：

```xml
<div><a href="/home">Home</a><a href="/Profile">My Profile</a><textarea>
Hello
World</textarea</div>
```

下面选项可用来控制空格的删除：

__选项1)__ 使用 `marko-compiler-options` 标签让删除空格实效：

```xml
<marko-compiler-options preserve-whitespace/>
<div>
    <img src="foo.jpg"/>
    <img src="foo.jpg"/>
</div>
```

__选项2)__ 使用 `marko-preserve-whitespace` 属性让删除空格实效：

```xml
<div marko-preserve-whitespace>
    <img src="foo.jpg"/>
    <img src="foo.jpg"/>
</div>
```

注：当保留空格功能生效的时候，任何级别的文本中的空格都会被保留，只要标签下面有 `marko-preserve-whitespace` 属性：

__选项3)__ 通过改变编译器的条件，可以让 _所有_ 空格删除实效：

```javascript
require('marko/compiler').defaultOptions.preserveWhitespace = true;
```

__选项4)__ 控制个别的标签（在 `marko.json`/`marko-tag.json`中）的空格删除

添加 `"preserve-whitespace": true` 属性到一个标签定义中，这会让Marko编译器无论标签在模版任何地方都保留空格：

```javascript
{
    "<my-custom-tag>": {
        "preserve-whitespace": true
    }
}
```

注：当空格保留生效的时候，该标签下面任何级别的文本中的空格都会被保留。


# Helpers

由于Marko模版文件编译成CommonJS模块，任何Node.js模块都可以被 "imported" 到一个模版中作为辅助模块。例如，给定下面的辅助模块：

_src/util.js_:
```javascript
exports.reverse = function(str) {
    var out = "";
    for (var i=str.length-1; i>=0; i--) {
        out += str.charAt(i);
    }
    return out;
};
```

上面的模块可以被插入到如下样例模版中：

_src/template.marko_:

```xml
<script marko-init>
    var util = require("./util");
</script>

<div>${util.reverse('reverse test')}</div>
```

同样可以传递辅助功能给一个模版，用来作为试图模型的一部分：


```javascript
var template = require('./template.marko');

template.render({
        reverse: function(str) {
            var out = "";
            for (var i=str.length-1; i>=0; i--) {
                out += str.charAt(i);
            }
            return out;
        }
    },
    function(err, html) { ... });
```

在模版中使：

```xml
<div>${data.reverse('reverse test')}</div>
```

# 混合使用

## invoke

`<invoke>` 指令可以在渲染过程中，被用来调用一个标准的JavaScript功能：

```xml
<invoke console.log('Hello World')/>
```

# 全局属性

`$global` 属性被给所有模版渲染时遇在一起，添加数据使用，通过让数据剥离封装的writer来实现。

```javascript
template.render({
    $global: {
        name: 'Frank'
    }
}, res);
```

给定如下的模版：

```xml
<div>
    Hello ${out.global.name}!
</div>
```

输入如下的结果：

```xml
<div>
    Hello Frank
</div>
```

<a name="custom-tags-and-attributes"></a>

# 自定义标签和自定义属性

Marko支持通过自定义标签和属性来扩展语言。一个自定义标签或者自定义属性 __必须只有一个破折号__ 来表明它不是标准HTML语法的内容。

下面展示了如何使用一个简单的自定义标签：

```xml
<div>
    <my-hello name="Frank" message-count="30"/>
</div>
```

上面的模版输出可能如下所示：

```html
<div>
    Hello Frank! You have 30 new messages.
</div>
```

使用Marko，属性值可能是一个JavaScript表达式：

```xml
<div>
    <my-hello name=data.name.toUpperCase() message-count=data.messageCount/>
</div>
```

你同样可以通过提供一个JavaScript表达式，传递一个数据对象给自定义标签：

```xml
<div>
    <my-hello({name: "Frank", messageCount: "30"})/>
</div>
```
如何使用和建立自定义的标签库的详细信息，请看该文档[Custom Taglibs](http://markojs.com/docs/marko/custom-taglibs/)。

# 异步标签

异步标签提供一个 `<await>` 标签来运行你部分模版被异步渲染。一个异步片段可以被绑定到一个接受 "args" 对象和回调声明的函数中。单数据提供者功能完成并调用有结果数据的回调时，异步片段的实体部分会连同被分配了指定变量的异步数据一起渲染。

作为一个补充功能，异步碎片允许你的页的每部分被无序渲染，但仍然一个最终有序的HTML，让你有一个几乎立刻反应的高效网站。想无序渲染这样的功能，是基于客户端再排序的，需要使用JavaScript。网站完全禁用JavaScript的，不应该使用该功能（尽然如此，但它仍能够使用异步碎片）。

例子：

```javascript
template.render({
        userProfilePromise: new Promise(function (resolve, reject) {
            // ...
        })
    }, ...);
```

```xml
<await(userProfile from data.userProfilePromise)>

    <ul>
        <li>
            First name: ${userProfile.firstName}
        </li>
        <li>
            Last name: ${userProfile.lastName}
        </li>
        <li>
            Email address: ${userProfile.email}
        </li>
    </ul>

</await>
```

更多细节，请参阅[Marko Async Taglib](http://markojs.com/docs/marko/async-taglib/)。

# 布局标签

Marko提供 `layout` 标签库来支持布局和内容的分量。`layout` 标签库的使用如下样例代码所示：

_default-layout.marko:_

```xml
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title><layout-placeholder name="title"/></title>
</head>
<body>
    <h1 if(data.showHeader !== false)>
        <layout-placeholder name="title"/>
    </h1>
    <p>
        <layout-placeholder name="body"/>
    </p>
    <div>
        <layout-placeholder name="footer">
            Default Footer
        </layout-placeholder>
    </div>
</body>
</html>
```

_使用 `default-layout.marko`:_

```xml
<layout-use("./default-layout.marko") show-header=true>
    <layout-put into="title">My Page</layout-put>
    <layout-put into="body">BODY CONTENT</layout-put>
</layout-use>
```


更多细节，请参阅[Marko Layout Taglib](http://markojs.com/docs/marko/layout-taglib/)。
