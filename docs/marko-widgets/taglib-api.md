Marko Widgets 标签库
====================

# 自定义属性

## w-bind

这个属性是用来绑定一个组件到DOM元素中。

### 例子

绑定名为 `./widget.js` 的JavaScript模块，它导出组件的定义：

```xml
<div w-bind="./widget">...</div>
```

绑定名为 `./widget.js` 或 `./index.js` （按顺序搜索） 的JavaScript模块，它导出组件的定义：

```xml
<div w-bind>...</div>
```

## w-id

用来分配一个 _作用域_ ID给一个嵌套的组件或者一个嵌套的DOM元素。这个ID是父组件ID的串联，它由 `w-id` 的值提供。

### 例子


#### Using `w-id` with an HTML element

```xml
<div w-bind="./widget">
    <button w-id="myButton" type="button">My Button</button>
</div>
```

这会生成类似如下的代码：


```html
<div>
    <button id="w0-myButton" type="button">My Button</button>
</div>
```

容器组件可用下面的代码引用嵌套DOM元素：

```javascript
var myButton = this.getEl('myButton');
```

#### 嵌套的组件使用 `w-id`

```xml
<div w-bind="./widget">
    <app-button w-id="myButton" label="My Button" />
</div>
```

这会生成类似如下的代码：

```html
<div>
    <button id="w0-myButton" type="button">My Button</button>
</div>
```

容器组件可用下面的代码引用嵌套DOM元素：

```javascript
var myButton = this.getWidget('myButton');
```

## w-on*

`w-on*` 用来声明式地给DOM元素或者组件绑定事件监听器。

注：为了性能，对冒泡的高效DOM事件委托的DOM事件会被自动用来阻止添加直接的事件监听器。

### 例子

#### 嵌套HTML元素使用 `w-on*`

```xml
<div w-bind="./widget">
    <button w-onclick="handleMyButtonClick" type="button">My Button</button>
</div>
```

当这个HTML按钮元素被电击，组件的 `handleMyButtonClick` 方法会被引用：

```javascript
module.exports = require('marko-widgets').defineComponent({
    // ...

    handleMyButtonClick: function(event, el) {
        // event will be the native DOM event
        // el will be the native DOM element
    }
})
```

容器组件可用下面的代码引用嵌套DOM元素：

```javascript
var myButton = this.getEl('myButton');
```

#### 嵌套组件使用 `w-on*`

```xml
<div w-bind="./widget">
    <app-button w-onSomeCustomEvent="handleSomeCustomEvent" label="My Button" />
</div>
```

上面的例子中，假定嵌套组件会用类似下面的代码发出自定义事件：

```javascript
this.emit('handleSomeCustomEvent', { foo: bar });
```

<a name="w-preserve"></a>

## w-preserve

当重渲染Ui组件的时候，保护DOM子树相关的DOM元素或者组件之类的不会被更改。

例子：

```xml
<div>
    <table w-preserve> <!-- Don't ever rerender this table -->
        ...
    </table>
</div>
```

```xml
<div>
    <app-map w-preserve/> <!-- Don't ever rerender this UI component -->
</div>
```

## w-preserve-if

除了DOM子树会被有条件地保护外，它类似 [w-preserve](#w-preserve)：

```xml
<div>
    <table w-preserve-if(data.tableData == null)>
        ...
    </table>
</div>
```

## w-preserve-body

除了子DOM节点会被保护外，它类似[w-preserve](#w-preserve)：

```xml
<div w-preserve-body> <!-- Don't ever rerender any nested DOM elements -->
    ...
</div>
```

## w-preserve-body-if

除了子DOM节点会被保护外，它类似[w-preserve-if](#w-preserve)：

```xml
<div>
    <table w-preserve-if(data.tableData == null)>
        ...
    </table>
</div>
```

## w-preserve-attrs

该自定义属性用来防止选中的DOM元素在重渲染时被更改：

```xml
<div w-preserve-attrs="class,style">
    ...
</div>
```

#### w-for

`w-for` 属性用来渲染可以引用作用域组件元素的 `for` 属性：

```xml
<form>
    <label w-for="yes">Yes</label>
    <input type="radio" w-id="yes" value="yes">

    <label w-for="no">No</label>
    <input type="radio" w-id="no" value="no">
</form>
```

它会生成类似下面的代码：

```html
<form>
    <label for="w0-yes">Yes</label>
    <input type="radio" w-id="w0-yes" value="yes">

    <label for="w0-no">No</label>
    <input type="radio" id="w0-no" value="no">
</form>
```

# 自定义标签

## `<init-widgets>`

生成必要的代码来初始化在 _服务器端_ 渲染的关联UI组件的小组件代码。

支持的属性：

- __`immediate`__ - 如果为true，那么会生成 `<script>` 标签，它会 _立即_ 初始化所有的组件，而不会等到DOM准备事件完成。对于异步片段，`<script>` 会插到每个异步片段的结尾。

### 例子

### 非即时组件初始化

如果没有提供 `immediate` 属性或者它的值设为 `false`，那么组件会在DOM准备好的时候初始。例如下面的Marko代码：

```xml
<init-widgets/>
```

这会生成如下类似的HTML代码：

```html
<noscript id="markoWidgets" data-ids="w0,w1,w2"></noscript>
```

`<noscript>` HTML标签是一个简单的用来保存所有根UI组件相关的IDs的容器，这个UI组件有需要初始化的一个小组件。当`marko-widgets` 模块在浏览中初始化后，它会查找  `#markoWidgets` ，以发现所有的组件IDs。

### 即时组件初始化

如果提供了 `immediate` 属性或者它的值设为  `true`，那么组件会通过添加在输出的HTML中的内联JavaScript代码初始化。例如下面的Marko代码：

```xml
<init-widgets immediate/>
```

这会生成如下类似的HTML代码：

```html
<script>
/* REMOVED: serialized widget state and config */
$markoWidgets("w0,w1,w2")
</script>
```

当即时组件初始化生效的时候，组件在DOM准备事件开始之前被初始化。此外，内联的组件初始化代码会在附加到每个异步碎片中。

## `<widget-types>`

使用条件绑定：

```xml
<widget-types default="./widget" mobile="./widget-mobile"/>

<div w-bind=(data.isMobile ? 'default' : 'mobile')>
    ...
</div>
```

`<widget-types>` 同样可以用来释放组件绑定：


```xml
<widget-types default="./"/>

<div w-bind=(data.includeWidget ? 'default' : null)>

</div>
```