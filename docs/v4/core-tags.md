# 核心标签和属性

Marko 提供了许多标签

## 控制流

### `<if>`, `<else-if>`, `<else>`

`<if>`、 `<else-if>` 和 `<else>` 标签为模版提供了条件控制流。

```xml
<if(arriving)>
    <div>Hey there</div>
</if>
<else-if(leaving)>
    <div>Bye now</div>
</else-if>
<else>
    <div>What's up?</div>
</else>
```

条件也可以应用在属性上：

```xml
<div if(arriving)>Hey there</div>
<div else-if(leaving)>Bye now</div>
<div else>What's up?</div>
```

并支持复杂表达式：

```xml
<if(Math.random() > 0.5)>
    <div>50-50 chance of seeing this</div>
</if>
```

### `<for>`

`<for>` 标签允许迭代一个数组：

```xml
<ul>
    <for(color in colors)>
        <li>${color}</li>
    </for>
</ul>
```

它也可以在属性中使用：

```xml
<ul>
    <li for(color in colors)>${color}</li>
</ul>
```

上面的任意一个模版中使用下面的  `colors` 值：

```js
var colors = ['red', 'green', 'blue'];
```

都会生成下面的的 HTML：

```html
<ul>
    <li>red</li>
    <li>green</li>
    <li>blue</li>
</ul>
```

#### 循环状态变量

如果你需要知道当前循环索引，`for` 指令支持循环状态变量，例如：

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

##### 循环状态方法

###### `getLength()`

返回数组的长度

###### `getIndex()`

返回当前循环索引

###### `isFirst()`

如果当前索引是第一个，会返回 `true`，不是的话返回 `false`

###### `isLast()`

如果当前索引是最后个，会返回 `true`，不是的话返回 `false`

#### 循环分割器

用来通过字符去分割值。使用 `separator`，第一个元素不会加前缀，最后一个元素不会加后缀：

```xml
<div>
    <!-- Output: red, green, blue -->
    <span for(color in colors | separator=", ") style="color: ${color}">
        ${color}
    </span>
</div>
```

#### 域循环

可以用下面的格式来提供一个域：`<var-name> from <from> to <to>[ step <step>]`。

`from`、 `to` 和 `step` 的值必须是数字表达式。如果没有明确指定，默认步长为1。

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

#### 属性值循环

```xml
<ul>
    <li for(name, value in settings)>
        <b>${name}</b>: ${value}
    </li>
</ul>
```

#### 原始 JavaScript 的 for 循环

```xml
<for(var i=1; i<=3; i++)>
    ${i}
</for>
```

#### 自定义迭代器


```xml
static function reverseIterator(arrayList, callback) {
    for(var i=arrayList.length-1; i>=0; i--){
        callback(arrayList[i]);
    }
}

<div for(item in ['a', 'b', 'c'] | iterator=reverseIterator)>
    ${item}
</div>
```

输出：

```html
<div>c</div>
<div>b</div>
<div>a</div>
```

### `<while>`

使用 `while` 指令，任何元素都可以迭代到满足一个条件为止。这个制定既可以作为元素使用，又可以作为属性使用。

_作为属性使用：_

```xml
$ var n = 0;

<ul>
    <li while(n < 4)>
        ${n++}
    </li>
</ul>
```

_作为元素使用：_

```xml
$ var n = 0;

<ul>
    <while(n < 4)>
        <li>${n++}</li>
    </while>
</ul>
```

### `body-only-if`

如果你发现你有一个封装元素是有条件的，但是它的主体应该永远被渲染，那么你就可以使用 `body-only-if` 属性来处理这种情况。例如，如果有一个有效的URL，为了只渲染封装的 `<a>` 标签，你可以像下面这样做：

```xml
<a href=input.linkUrl body-only-if(!input.linkUrl)>
    Some body content
</a>
```

为 `input.linkUrl` 变量赋一个 `"http://localhost/"` 值，会有如下的输出：

```xml
<a href="http://localhost/">
    Some body content
</a>
```

为 `input.linkUrl` 变量赋一个 `undefined` 值，会有如下的输出：

```xml
Some body content
```

## JavaScript

即便使用 HTML 的语法标签来生成 HTML 输出，下面的标签需要使用[简易语法](./concise.md)。

### `import`
> **Static:** `import` 生成的代码只会在模版加载后执行一次，并且会被渲染器所有的调用共享。它必须在顶层标签声明，并且不能获得在渲染器传递的 `data`、 `state` 或者其他值。

`import` 标签用来从外部文件中获得数据或者方法。它遵循 JavaScript [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)相同的语法规则。

```xml
import sum from './utils/sum';
<div>The sum of 2 + 3 is ${sum(2, 3)}</div>
```

### `<export>`
> **Static:** `export` 生成的代码只会在模版加载后执行一次，并且会被渲染器所有的调用共享。它必须在顶层标签声明，并且不能获得在渲染器传递的 `data`、 `state` 或者其他值。

`export` 标签用来导出模版中的值。它遵循 JavaScript [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)相同的语法规则。

```xml
export var route = '/about';

<!doctype html>
<html>
    <body>
        <h1>About us</h1>
    </body>
</html>
```

## 可重用内容

### `<include>`

include 标签用来将其他模版（或者其他模版部分内容）嵌入到当前模版中。

include 标签可以将模版的路径作为第一个参数：

```xml
<include('./path/to/template.marko')/>
```

第二个参数可以用来传递输入给被包含的模版：

```xml
<include('./path/to/template.marko', { name:'Frank' })/>
```

也可以用属性来传递数据给被包含的模版，且可以和输入参数合并在一起：

```xml
<include('./path/to/template.marko', data) name="Frank"/>
```

#### 嵌套属性布局

除了可以包含外部内容，你可以将额外的内容块注入到外部内容中。这个功能可以用嵌套标签来完成，使用它需要用 `@` 符号来表明：

_page.marko_
```xml
<include('./layout.marko')>
    <@body>
        <h1>Hello Marko</h1>
    </@body>
</include>
```

然后再你的布局模版中，你可以 include 这个注入内容：

_layout.marko_
```xml
<!doctype html>
<html>
    <body>
        <!-- this comes from <@body> -->
        <include(input.body)/>
    </body>
</html>
```

<!--
- You can use many nested attribute tags for multiple injection points
- You can have repeated nested attribute tags by using `marko-tag.json` (components only)
- You can add additional attributes to an nested attribute tag
- You can pass data to a nested attribute tag's body?
-->

### `<include-text>`

```xml
<include-text('./foo.txt')/>
```

特殊的 HTML 字符可以被转义。如果你不想转义的话，你可以使用 `<include-html>` 标签（如下）。

### `<include-html>`

```xml
<include-html('./foo.html')/>
```

特殊的 HTML 字符不会被转义，因为这份文件被认为是一个 HTML 文件。

### `<macro>`

参数化宏指令可以让重用片段在一个HTML模版中。一个宏指令可以用 `<macro>` 命令来定义。

```xml
<macro greeting(name, count)>
    Hello ${name}! You have ${count} new messages.
</macro>
```

上面的宏指令可以作为任何表达式的一部分被引用。下面的样例模版演示了在表达式中如何使用宏函数：

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

## 异步内容

### `<await>`

`<await>` 标签用来从数据提供者那里动态加载内容。数据提供者可以是一个 `Promise` 或者一个 `callback`。一旦数据提供者返回结果，`<await>` 子节点就会被渲染。

await-example.marko
```xml
$ var personPromise = new Promise((resolve, reject) => {
    setTimeout(function() {
        resolve({
            name: 'Frank'
        });
    }, 1000);
});

<await(person from personPromise)>
    <div>Hello ${person.name}!</div>
</await>
```

高级应用：

+ <await> 标签签名
    * 基本用法: <await(results from dataProvider)>...</await>
    * 可选参数
        - client-reorder `boolean`
        - arg `expression`
        - arg-* `string`
        - method `string`
        - timeout `integer`
        - timeout-message `string` 
        - error-message `string`
        - placeholder `string`
        - renderTimeout `function`
        - renderError `function`
        - renderPlaceholder `function`
        - name `string`
        - scope `expression`
        - show-after `string`
    * 可选子标签
        - <await-placeholder>Loading...</await-placeholder>
        - <await-timeout>Request timed out</await-timeout>
        - <await-error>Request errored</await-error>

## 注释

标准的 HTML 注释可以可以在你的模版中使用。HTML注释不会在渲染好的 HTML 中出现。

注释例子：

```xml
<!-- This is a comment that will not be rendered -->
<h1>Hello</h1>
```

```js
// You can also use standard JavaScript-style comments
/*
 Block comments are also supported
 */
-- Hello
```

如果你想让你的注释在最终的输出中出现，那么你可以使用自定义的  `html-comment` 标签。

### `<html-comment>`

_input.marko_

```xml
<html-comment>This is a comment that *will* be rendered</html-comment>
<h1>Hello</h1>
```

_output.html_

```html
<!--This is a comment that *will* be rendered-->
<h1>Hello</h1>
```

此外，你也可以使用 `<marko-compiler-options>` 来给整个模版配置注释：

```xml
<marko-compiler-options preserve-comments/>
```

## 编译器 options

### `marko-preserve-whitespace`

使用 `preserve-whitespace` 属性可以保存空格

```xml
<div marko-preserve-whitespace>
    All of this
    whitespace   will
    be preserved.
</div>
```

此外，你也可以使用 `<marko-compiler-options>` 来给整个模版配置空格：

```xml
<marko-compiler-options preserve-whitespace/>
```

### `marko-body`

`marko-body` 属性可以用来控制主体内容如何解析。支持下面几个值： 

- `html` - 主体内容会解析成 HTML （模版）。
- `static-text` - 主体内容会解析成静态文本（HTML 标签会被忽略）。占位符会被忽略。
- `parsed-text` - 主体内容会解析成文本（HTML会被忽略）。占位符不会被忽略。

_input.marko_

```xml
<div marko-body="static-text">
    This is just one
    <span if(foo)>
            Hello ${THIS IS NOT VALID}!
    </span>
    big text block
</div>
```

_output.html_

```html
<div>
    This is just one
    &lt;span if(foo)&gt;
        Hello ${THIS IS NOT VALID}!
    &lt;/span&gt;
    big text block
</div>
```
