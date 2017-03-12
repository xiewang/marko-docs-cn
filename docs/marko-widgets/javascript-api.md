JavaScript API
==============

# marko-widgets exports

## defineComponent(def)

用来定义一个UI组件，UI组件包含渲染器和小组件（例如：客户端行为）。如果一个UI组件只会在服务器端渲染，那么你可能通过使用 `defineRenderer(def)` 和 `defineWidget(def)` 函数尝到组件独立案渲染所带来方便的甜头。

`defineComponent(def)` 的返回值是一个 `Widget` 构造函数，它带有 `renderer(input, out)` 和 `render(input)` 方法。

定义一个无状态UI组件的方法示例：

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getTemplateData: function(state, input) {
		return {
			name: input.name
		};
	},

	handleClick: function() {
		this.el.style.backgroundColor = 'yellow';
	}
});
```

## defineRenderer(def)

`defineRenderer(def)` 函数可以用来定义一个独立于关联组件的UI组件渲染器。当一个UI组件之需要在服务器端渲染的时候，这就很有好处，而且不给浏览器下发模版和渲染逻辑是可取的方法。对于之在服务器渲染的UI组件，只有客户端行为才需要发送到浏览器。

`defineRenderer(def)` 的返回值是一个 `renderer(input, out)` 函数和一个静态 `render(input)` 方法。

## defineWidget(def)

`defineWidget(def)` 函数用来定义UI组件在客户端的行为，渲染UI组件的独立代码。当一个UI组件之需要在服务器端渲染的时候，这就很有好处，而且不给浏览器下发模版和渲染逻辑是可取的方法。对于之在服务器渲染的UI组件，只有客户端行为才需要发送到浏览器。
The `defineWidget(def)` function can be used to define a UI component's client-side behavior independent of the code to render the UI component. This can be beneficial when a UI component needs to only be rendered on the server and it is desirable to avoid sending down the template and rendering logic to the browser. For UI components that are only rendered on the server, only the client-side behavior really needs to be be sent to the browser.

`defineRenderer(def)` 的返回值是一个小组件构造函数，它用来实例化新的组件实例。

## getRenderedWidgets(out)

_注：只在服务端可用_

用来支持在服务端渲染的UI组件初始化组件捆绑内容，通过呢AJAX请求发送到浏览器里。

```javascript
var markoWidgets = require('marko-widgets');
var template = require('./template.marko');

module.exports = function(req, res) {
	template.render(viewModel, function(err, html, out) {
		var renderedWidgets = markoWidgets.getRenderedWidgets(out);

		// Serialize the HTML and the widget IDs to the browser
		res.json({
	            html: html,
	            renderedWidgets: renderedWidgets
	        });
	});
}
```

然后，在浏览器，下面的的代码可以来初始化这个组件：

```javascript
var result = JSON.parse(response.body);
var html = result.html
var renderedWidgets = result.renderedWidgets;

document.body.innerHTML = html; // Add the HTML to the DOM

// Initialize the widgets to bind behavior!
require('marko-widgets').initWidgets(renderedWidgets);
```

## getWidgetForEl(el)

`getWidgetForEl(el)` 函数用来在组件嵌套内容之外取回一个组件对象。

```javascript
var myToggle = require('marko-widgets').getWidgetForEl('w0-myToggle');
myToggle.setSelected(true);
```

它也可通过使用组件的el来获得组件处理方法：

```javascript
var el = document.getElementById('w0-myToggle');
var myToggle = require('marko-widgets').getWidgetForEl(el);
myToggle.setSelected(true);
```

# Widget

## 方法

### $(querySelector)

当使用jQuery的时候，这个方法可以用来很方便的获得一个组件DOM元素。这个混合方法充当了jQuery的一个代理，用来轻松构建基于组件元素IDs的选择器。

在里面，jQuery代理方法会通过给组件元素IDs添加组件ID前缀来让组件元素ID指向真正的DOM元素ID：

```javascript
this.$() ➡ $("#w123")
this.$("#myEl") ➡ $("#w123-myEl")
```

下面介绍它的混合使用方法：

__`$()`__

封装成一个jQuery对象，获得根组件DOM元素的简易用法。下面的所有方法都是等价的：

```javascript
this.$()
$(this.el)
$("#" + this.id)
```

__`$('#<widget-el-id>')`__

封装成一个jQuery对象，获得嵌套组件DOM元素的简易用法。下面的所有方法都是等价的：

```javascript
this.$("#myEl")
$(this.getEl("myEl"))
$("#" + this.getElId("myEl"))
```

__`$('<selector>')`__

查询根组件DOM元素内的嵌套DOM元素的简易用法。下面的所有方法都是等价的：

```javascript
this.$("ul > li")
$("ul > li", this.el)
$("#" + this.id + " ul > li")
```

__`$('<selector>', '<widget-el-id>')`__

查询嵌套组件DOM元素内的嵌套DOM元素的简易用法。下面的所有方法都是等价的：

```javascript
this.$("li.color", "colorsUL")
this.$("#colorsUL li.color")
$("li.color", this.getEl("colorsUL"))
$("#" + this.getElId("colorsUL") + " li.color")
```

__`$('#<widget-el-id> <selector>')`__

查询根组件DOM元素内的嵌套DOM元素的简易用法。下面的所有方法都是等价的：

```javascript
this.$("#colorsUL li.color")
this.$("li.color", "colorsUL")
$("li.color", this.getEl("colorsUL"))
$("#" + this.getElId("colorsUL") + " li.color")
```

__`$(callbackFunction)`__

给在DOM准备好的事件添加一个监听器，以及让给提供的回调函数的对象成为当前组件的实例的简易用法。下面的所有方法都是等价的：

```javascript
this.$(function() { /*...*/ });
$(function() { /*...*/ }.bind(this));      // Using Function.prototype.bind
$($.proxy(function() { /*...*/ }, this));
```

### addEventListener(eventType, listener)

### appendTo(targetEl)

将组件的根DOM节点从当前的父元素移动到一个新的父元素中。例如：

```javascript
this.appendTo(document.body);
```

### destroy()

通过释放所有使用 `subscribeTo` 方法的监听器销毁组件，然后将这个组件根元素从DOM中分离。所有的嵌套的组件（通过搜索DOM来查找）都会被销毁。

Destroy有两个可选参数：

```javascript
widget.destroy({
	removeNode: true, //true by default
	recursive: true //true by default
})
```
将 `removeNode` 设置为 `false` 会保留这个组件在DOM中，同时仍然销毁它的所有事件。＝

将 `recursive` 设置为 `false` 会防止子组件被销毁。

### detach()

通过将节点从父节点中移除，来将组件更元素从DOM中分离。

### doUpdate(stateChanges, oldState)

### emit(eventType, arg1, arg2, ...)

发射一个事件。这个方法会从EventEmitter中插入（参阅[Node.js Events: EventsEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter)）

### getBodyEl()

### getEl(widgetElId)

通过给提供 `widgetElId` 添加组件ID前缀，返回一个嵌套的DOM元素，嵌套的组件元素应该用自定义属性 `w-id` 分配一个ID。如果 `widgetElId` 没有提供，会返回  `this.el`。

### getEls(id)

从指定的ID中返回 _重复的_ `DOM` 元素数组。重复的DOM元素必须有一个 `w-id` 属性的值，并以 `[]` 结尾（例如：`w-id="myDivs[]"`）。

### getElId(widgetElId)

类似 `getEl`，但是只会返回嵌套DOM元素的字符串ID，而不是真实的DOM元素。

### getWidget(id[, index])

返回用于指定ID嵌套  `Widget` 的相关内容。如果提供来一个 `index`，并且这个目标组件是一个重复的组件（例如：`w-id="myWidget[]"`），那么这个指定索引的组件会被返回。

### getWidgets(id)

为制定ID返回 _重复的_ `Widget` 实例数组。重复的组件必须有一个 `w-id` 属性的值，并以 `[]` 结尾（例如：`w-id="myWidget[]"`）。

### insertAfter(targetEl)

### insertBefore(targetEl)

### isDestroyed()

### isDirty()

### on(eventType, listener)

### prependTo(targetEl)

### ready(callback)

### replace(targetEl)

### replaceChildrenOf(targetEl)

### replaceState(newState)

给当前状态替换成一个完整的全新状态。如果这个状态值有任何改变，组件视图就会自动更新。

值得注意的：
虽然 `setState()` 是附加物，不会删除新状态中没有的旧状态的值，`replaceState()` 会添加新的状态，并删除新状态中没有的旧状态值。来自不属于新状态的状态属性的状态或者模版数据值会变成  `undefined`。因此，如果使用 `replaceState()`，必须考虑到它的副作用，那就是新的状态时候包含少于被替换状态的值或者其他的值。

### rerender(data)

使用 `renderer` 提供的 `data` 或者内部的 `state`重渲染组件。

### setState(name, value)

用来改变一个单独状态的值。例如：

```javascript
this.setState('disabled', true);
```

注意，`setState()` 只会为合适的渲染器指定组件。因此，如果至少一个组件状态值改变 (`oldValue !== newValue`)，这个组件才会被重渲染。如果没有任何值改变（因为前后只相同或者没有被一个简单的比较程序探测到），引用 `setState()` 不会有任何操作。


相比`setState()`， `setStateDirty()` 不会为渲染指定一个组件，但是相反，它永远从他的状态属性值中独立重新渲染组件（即便这些值没有改变）。

### setState(newState)

用来改变多个状态的值。例如：

```javascript
this.setState({
	disabled: true,
	size: 'large'
});
```

### setStateDirty(name, value)

强制更新一个状态值，即使这个值和旧的值一样。在遇到一个复杂的对象情况下，这个对象不会被简单的对比来检查，那么这个功能就很有用。引用这个方法会完全规避所有的相同值的坚持（简单比对），并永远重渲染这个组件。

额外信息：
第一个参数 `name` 是用来让更新处理器（例如：`update_foo(newValue)`）来给特定的标记为脏数据的状态处理状态转换。第二个参数 `value` 用来作为更新器的新值。因为 `setStateDirty()` 永远绕过所有值的相同检查，这个参数是可选的。如果没有指定，或者和旧的值相同，旧的值会给更新器使用。值得注意的是，指定的参数不会影响 setStateDirty()` 如何或者是否渲染组件；它们知识作为更新处理器的额外信息。

例子：

```javascript
// Add a new item to an array without going through `this.setState(...)` - because this
// does not create a new array, the change would not be detected by a shallow property comparison
this.state.colors.push('red');

// Force that particular state property to be considered dirty so
// that it will trigger the widget's view to be updated
this.setStateDirty('colors');
```

### setProps(newProps)

对于有状态组件来说，使用心得input设置组件值会让组件的重渲染。对于无状态组件，设置组件值会导让`getInitialState(newProps)` 被再次调用，`getInitialState(newProps)` 方法确定新的状态，使用新的状态，组件状态会被更新。

### subscribeTo(targetEventEmitter)

### update()

立刻强制更新DOM，而不使用下面的通常的更新机制来重渲染。

```js
this.setState('foo', 'bar');
this.update(); // Force the DOM to update
this.setState('hello', 'world');
this.update(); // Force the DOM to update
```

## Properties

### this.el

根[HTML 元素](https://developer.mozilla.org/en-US/docs/Web/API/element)，它和组件绑定。

### this.id

根[HTML 元素](https://developer.mozilla.org/en-US/docs/Web/API/element)的字符串ID，它和组件绑定。

### this.state

组件的当前状态。例如：

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getInitialState: function(input) {
		return {
			disabled: false
		}
	},

	init: function() {
		console.log(this.state.disabled); // Output: false
		this.setState('disabled', true);
		console.log(this.state.disabled); // Output: true
	}
});
```
