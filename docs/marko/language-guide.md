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

The `unless` directive is also supported as an alternative to `if` in cases where the condition should be negated.

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

*Applied as elements:*

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

## Shorthand Conditionals

Regular JavaScript can be used to achieve shorthand conditional values inside attributes or wherever expressions are allowed:


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

_NOTE: If an attribute value expression evaluates to `null` or `false` then the attribute is not included in the output._

A ternary condition can also be used:

```xml
<div class=(active ? 'tab-active' : 'tab-inactive')>Hello</div>
```

With a value of `false` for `active`, the output would be the following:

```html
<div class="tab-inactive">Hello</div>
```

## Conditional Attributes

Marko supports conditional attributes when the value of an attribute is an expression. Marko also supports [HTML `boolean` attributes](https://html.spec.whatwg.org/#boolean-attributes) (e.g., `<input type="checkbox" checked>`)  If an attribute value resolves to `null`, `undefined`, or `false` then the attribute will not be rendered. If an attribute value resolves to `true` then only the attribute name will rendered.

For example, given the following data:

```javascript
{
    active: true,
    checked: false,
    disabled: true
}
```

And the following template:

```xml
<div class=(data.active && "tab-active")/>

<input type="checkbox" checked=data.checked disabled=data.disabled/>
```

The output HTML will be the following:

```html
<div class="tab-active"></div>

<input type="checkbox" disabled>
```

# Looping

## for

Any element can be repeated for every item in an array using the `for` directive. The directive can be applied as an element or as an attribute.

_Applied as an attribute:_

```xml
<ul>
    <li for(item in items)>${item}</li>
</ul>
```

_Applied as an element:_

```xml
<ul>
    <for(item in items)>
        <li>${item}</li>
    </for>
</ul>
```


Given the following value for items:

```javascript
["red", "green", "blue"]
```

The output would be the following:

```html
<ul>
    <li>red</li>
    <li>green</li>
    <li>blue</li>
</ul>
```

### Loop Status Variable

The `for` directive also supports a loop status variable in case you need to know the current loop index. For example:

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

### Loop Separator

```xml
<for(color in colors | separator=", ")>${color}</for>
<div>
    <span for(color in colors | separator=", ") style="color: ${color}">
        ${color}
    </span>
</div>
```

### Range Looping

A range can be provided in the following format; `<var-name> from <from> to <to>[ step <step>]`.

The `from`, `to` and `step` values must be numerical expressions. If not specified, step defaults to 1.

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


### Property Looping

```xml
<ul>
    <li for(name,value in settings)>
        <b>${name}</b>: ${value}
    </li>
</ul>
```

### Native JavaScript for-loop

```xml
<for(var i=1; i<=3; i++)>
    ${i}
</for>
```

### Custom Iterator

A custom iterator function can be passed as part of the view model to the template to control looping over data.


A sample custom iterator function that loops over an array in reverse is shown below:

```javascript
{
    reverseIterator: function(arrayList, callback) {
        for(var i=arrayList.length-1; i>=0; i--){
            callback(arrayList[i]);
        }
    }
}
```

The custom iterator can then be used in a template as shown below:

_Applied as part of a `for` attribute:_

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

_Applied as part of a `<for>` element:_

```xml
<for(item in ['a', 'b', 'c'] | iterator=data.reverseIterator)>
    ${item}
</for>
```

```html
cba
```

Custom iterators also support providing a custom status object for each loop iteration:

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

_Applied as part of a `for` attribute:_

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

_Applied as part of a `<for>` element:_

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

Any element can be repeated until a condition is met by using the `while` directive. The directive can be applied as an element or as an attribute.

_Applied as an attribute:_

```xml
<var n=0/>
<ul>
    <li while(n < 4)>
        ${n++}
    </li>
</ul>
```

_Applied as an element:_

```xml
<var n=0/>
<ul>
    <while(n < 4)>
        <li>${n++}</li>
    </while>
</ul>

```

In both cases the output would be the following:

```html
<ul>
    <li>0</li>
    <li>1</li>
    <li>2</li>
</ul>
```

# Macros

Parameterized macros allow for reusable fragments within an HTML template. A macro can be defined using the `<macro>` directive.

## macro

The `<macro>` directive can be used to define a reusable function within a template.

```xml
<macro greeting(name, count)>
    Hello ${name}! You have ${count} new messages.
</macro>
```

The above macro can then be invoked as part of any expression. Alternatively, the [`<invoke>`](#invoke) directive can be used invoke a macro function using named attributes. The following sample template shows how to use macro functions inside expressions:

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


# Structure Manipulation

## Dynamic attributes

The `attrs` attribute allows attributes to be dynamically added to an element at runtime. The value of the attrs attribute should be an expression that resolves to an object with properties that correspond to the dynamic attributes. For example:

```xml
<var myAttrs={style: "background-color: #FF0000;", "class": "my-div"} />

<div ${myAttrs}>
    Hello World!
</div>
```

Output:

```html
<div style="background-color: #FF0000;" class="my-div">
    Hello World!
</div>
```

## body-only-if

If you find that you have a wrapper element that is conditional, but whose body should always be rendered then you can use the `body-only-if` attribute to handle this use case. For example, to only render a wrapping `<a>` tag if there is a valid URL then you could do the following:

```xml
<a href=data.linkUrl body-only-if(!data.linkUrl)>
    Some body content
</a>
```

Given a value of `"http://localhost/"` for the `data.linkUrl` variable: , the output would be the following:

```xml
<a href="http://localhost/">
    Some body content
</a>
```

Given a value of `undefined` for the `data.linkUrl` variable: , the output would be the following:

```xml
Some body content
```

# Comments

Standard HTML comments can be used to add comments to your template. The HTML comments will not show up in the rendered HTML.

Example comments:

```xml
<!-- This is a comment that will not be rendered -->
<h1>Hello</h1>
```

If you would like for your HTML comment to show up in the final output then you can use the custom `html-comment` tag:
```xml
<html-comment>This is a comment that *will* be rendered</html-comment>
<h1>Hello</h1>
```

Output:

```xml
<!--This is a comment that *will* be rendered-->
<h1>Hello</h1>
```

# Whitespace

The Marko compiler will remove unnecessary whitespace based on some builtin rules, by default. These rules are partially based on the rules that browser's use to normalize whitespace and partially based on the goal of allowing nicely indented markup with minified output. These rules are as follows:

- For text before the first child element: `text.replace(/^\n\s*/g, '')`
- For text after the last child element: `text.replace(/\n\s*$/g, '')`
- For text between child elements: `text.replace(/^\n\s*$/g, '')`
- Any contiguous sequence of whitespace characters is collapsed into a single space character

In addition, whitespace within the following tags is preserved by default:

- `<pre>`
- `<textarea>`
- `<script>`

Example template:

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

Example output:

```xml
<div><a href="/home">Home</a><a href="/Profile">My Profile</a><textarea>
Hello
World</textarea</div>
```

The following options are available to control whitespace removal:

__Option 1)__ Disable whitespace removal using the `marko-compiler-options` tag:

```xml
<marko-compiler-options preserve-whitespace/>
<div>
    <img src="foo.jpg"/>
    <img src="foo.jpg"/>
</div>
```

__Option 2)__ Disable whitespace removal using the `marko-preserve-whitespace` attribute:

```xml
<div marko-preserve-whitespace>
    <img src="foo.jpg"/>
    <img src="foo.jpg"/>
</div>
```

NOTE: When whitespace preservation is enabled, whitespace will be preserved for text at any level below the tag that the `marko-preserve-whitespace` attribute is attached to.

__Option 3)__ Disable _all_ whitespace removal by changing a compiler option

```javascript
require('marko/compiler').defaultOptions.preserveWhitespace = true;
```

__Option 4)__ Control whitespace removal for specific tags (in `marko.json`/`marko-tag.json`)

Adding the `"preserve-whitespace": true` property to a tag definition will result in the Marko compiler preserving whitespace wherever that tag is encountered in a template.

```javascript
{
    "<my-custom-tag>": {
        "preserve-whitespace": true
    }
}
```

NOTE: When whitespace preservation is enabled, whitespace will be preserved for text at any level below the tag.

# Helpers

Since Marko template files compile into CommonJS modules, any Node.js module can be "imported" into a template for use as a helper module. For example, given the following helper module:

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

The above module can then be imported into a template as shown in the following sample template:

_src/template.marko_:

```xml
<script marko-init>
    var util = require("./util");
</script>

<div>${util.reverse('reverse test')}</div>
```

It's also possible to pass helper functions to a template as part of the view model:


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

Usage inside template:

```xml
<div>${data.reverse('reverse test')}</div>
```

# Miscellaneous

## invoke

The `<invoke>` directive can be used to invoke a standard JavaScript function during rendering:

```xml
<invoke console.log('Hello World')/>
```

# Global Properties

The `$global` property is used to add data that is available to all templates encountered during rendering by having the data hang off the wrapped writer.

```javascript
template.render({
    $global: {
        name: 'Frank'
    }
}, res);
```

Given the following template:

```xml
<div>
    Hello ${out.global.name}!
</div>
```

The output would be the following:

```xml
<div>
    Hello Frank
</div>
```

<a name="custom-tags-and-attributes"></a>

# Custom Tags and Custom Attributes

Marko supports extending the language with custom tags and attributes. A custom tag or a custom attribute __must have at least one dash__ to indicate that is not part of the standard HTML grammar.

Below illustrates how to use a simple custom tag:

```xml
<div>
    <my-hello name="Frank" message-count="30"/>
</div>
```


The output of the above template might be the following:

```html
<div>
    Hello Frank! You have 30 new messages.
</div>
```

With Marko, attribute values can be JavaScript expressions:

```xml
<div>
    <my-hello name=data.name.toUpperCase() message-count=data.messageCount/>
</div>
```

You can also pass a data object by providing a JavaScript expression as an argument to the custom tag:

```xml
<div>
    <my-hello({name: "Frank", messageCount: "30"})/>
</div>
```

For information on how to use and create custom taglibs, please see the documentation for [Custom Taglibs](http://markojs.com/docs/marko/custom-taglibs/).

# Async Taglib

The async taglib provides an `<await>` tag that allows portions of your template to be rendered asynchronously. An asynchronous fragment can be bound to a function that accepts an "args" objects and callback argument. When the data provider function completes and invokes the callback with the resulting data, the body of the async fragment is then rendered with the asynchronous data assigned to the specified variable. As an additional feature, asynchronous fragments allow parts of your page to render out-of-order while still providing the final HTML in the correct order allowing to have very reactive websites with almost instant visual feedback. Features like out-of-order rendering, that are based on client-reordering, require the use of JavaScript. Websites that have to render completely without JavaScript should avoid using this additional feature (they can still use asynchronous fragments though).

Example:

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

For more details, please see [Marko Async Taglib](http://markojs.com/docs/marko/async-taglib/).

# Layout Taglib

Marko provides a `layout` taglib to support separating out layout from content. The usage of the `layout` taglib is shown in the sample code below:

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

_Usage of `default-layout.marko`:_

```xml
<layout-use("./default-layout.marko") show-header=true>
    <layout-put into="title">My Page</layout-put>
    <layout-put into="body">BODY CONTENT</layout-put>
</layout-use>
```


For more details, please see [Marko Layout Taglib](http://markojs.com/docs/marko/layout-taglib/).
