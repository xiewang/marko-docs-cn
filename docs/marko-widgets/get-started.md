入门
===========

<!--{TOC}-->

# 安装

```bash
npm install marko-widgets --save
```

# 术语表

在你开始前，先来熟悉一些定义：

* "widget"是UI组件的"客户端行"
* 一个组件实例有如下一些特性
	 * 所有的组件实例和DOM元素绑定
	 * 所有的组件是 [事件发射器](http://nodejs.org/api/events.html)
* 客户端行为包含下面几点：
    * 添加DOM事件监听器（鼠标点击事件，键盘点击事件等）
    * 给其他组件添加监听器
    * 操作DOM
    * 发布客户端事件
    * 等。

# 使用方法

## 绑定行为

给Marko使用绑定，通过使用自定义  `w-bind` 属性，你可以绑定一个组件到渲染好的DOM上，如下样例模版所示：

```xml
<div class="my-component" w-bind="./widget">
    <div>Click Me</div>
</div>
```

你也可以选择让 `w-bind` 属性值为空。如果值为空，`marko-widgets` 会通过先检查是否有 `widget.js` 存在，再在检查 `index.js` 是否存在来寻找一个组件模块。例如：

```xml
<div class="my-component" w-bind>
    <div>Click Me</div>
</div>
```

这个绑定到 `<div>` 的组件应该作为一个CommonJS模块来实现，这个模块暴露一个组件类型，如下面的JavaScript代码所示：

__src/pages/index/widget.js:__

```javascript

module.exports = require('marko-widgets').defineComponent({
	init: function() {
		var rootEl = this.el; // this.el returns the root element that the widget is bound to
	    var self = this;

	    rootEl.addEventListener('click', function() {
	        self.addText('You clicked on the root element!');
	    });
	},

	addText: function(text) {
        this.el.appendChild(document.createTextNode(text));
    }
})
```

## 组件Props

当一个组件被首次渲染时，它会接受一个初始的一组值。例如：

```javascript
require('fancy-checkbox').render({
		checked: true,
		label: 'Foo'
	});
```

如果组件时有状态的，那么这个状态值会来自从input值，模版数据会来自这个状态值。如果组件是无状态的，那么模版数据应该直接来自input值。如果你需要规范化input值，你可以实现 `getInitialProps(input, out)` 方法，具体如下：


```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getInitialProps: function(input, out) {
		return {
			size: input.size ? input.size.toLowerCase() : 'normal'
		};
	},

	getTemplateData: function(state, input) {
		// input will be the value returned by getInitialProps()
		// ...
	}

	// ...
});
```

## 组件模版

每个组件应该喝一个Marko模版关联在一起，这个模版会用来渲染组件。一个用 `template` 属性关联模版的组件如下所示：

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getTemplateData: function(state, input, out) {
		return {
			name: input.name
		};
	},

	...
});
```

`getTemplateData(state, input, out)` 方法用来构建试图模型，它会根据state 和／或 input传递到模版中。如果组件是有状态的，那么模版数据应该来自 `state`。如果组件是无状态的，那么模版数据来自 `input`。如果一个有状态的组件被重新渲染，那么 `input` 会永远是 `null`。对于无状态的组件来说，`state` 参数会是 `null`。

## 组件状态

一个有状态的组件会维护一个状态，来作为组件实例的一部分。如果这个组件的状态改变了，那么组件就会加入到对立中，在后面被更新。初始状态会使用 `getInitialState(input)` 方法来提供。所有状态的改变都应该通过 `setState(name, value)` 或者 `setState(newState)` 方法。例如：

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getInitialState: function(input, out) {
		return {
			name: input.name,
			selected: input.selected || false;
		}
	},

	getTemplateData: function(state, input) {
		return {
			name: state.name,
			color: state.selected ? 'yellow' : 'transparent'
		};
	},

	handleClick: function() {
		this.setState('selected', true);
	},

	isSelected: function() {
		return this.state.selected;
	}
});
```

当前组件的状态可以永远通过使用  `this.state` 属性来读取。例如：

```javascript
var isSelected = this.state.selected === true;
```

当状态被 `setState(name, value)` 或者 `setState(newState)` 方法改变时，只能简单对比下来看状态是否改变。因此，如果状态时一个复杂的对象，那么它应该视为不可变的。

## 组件配置

组件配置的数据在渲染的时候决定，它会通过实现 `getWidgetConfig(input, out)` 来提供给组件的构造函数使用，如下：

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getWidgetConfig: function(input, out) {
		return {
			foo: 'bar'
		}
	},

	init: function(widgetConfig) {
		var foo = widgetConfig.foo; // foo === 'bar'
	},

	...
});

```

## 引用嵌套组件

`marko-widgets` 标签库同样提供了对组件直接和嵌套组件通行的支持。一个嵌套组件会被分配一个组件ID（只需要和所有容器组件保持唯一就行），并且这个容器组件可以通过这个分配的组件ID引用嵌套组件，引用方法是使用 `this.getWidget(id)`。

下面的HTML模版片段包含一个组件，这个组件有三个嵌套的组件[sample-button](https://github.com/marko-js-samples/marko-sample-components/tree/master/components/sample-button)。每个嵌套的[sample-button](https://github.com/marko-js-samples/marko-sample-components/tree/master/components/sample-button)被分配一个ID（例如：`primaryButton`、 `successButton` 和 `dangerButton`）／

```xml
<div class="my-component" w-bind="./widget">
    <div class="btn-group">
        <sample-button label="Click Me" variant="primary" w-id="primaryButton"/>
        <sample-button label="Click Me" variant="success" w-id="successButton"/>
        <sample-button label="Click Me" variant="danger" w-id="dangerButton"/>
    </div>
    ...
</div>
```

容器组件可以像下面的的示例JavaScript代码一样引用一个特定的嵌套组件：

```javascript
this.getWidget('dangerButton').on('click', function() {
    alert('You clicked on the danger button!');
});
```

Marko组件同样支持引用 _重复的_ 嵌套组件，如下：

```xml
<div class="my-component" w-bind="./widget">
    <ul>
		<li for="todoItem in data.todoItems">
			<app-todo-item w-id="todoItems[]" todo-item="todoItem"/>
		</li>
	</ul>
</div>
```

容器组件可通过使用 `this.getWidgets(id)`  引用这些嵌套的todo item，如下：

```javascript
var todoItemWidgets = this.getWidgets('todoItems');
// todoItemWidgets will be an Array of todo item widgets
```

想动手尝试使用这些代码，请参照[widget-communication](https://github.com/marko-js-samples/widget-communication) 示例应用的文档和源文件。

## 引用嵌套DOM元素

和组件嵌套在一起的DOM元素可以基于容器组件的ID，给定唯一的ID。这些DOM元素可以通用容器组件提供的方法被高效的查找到。`w-id` 自定义属性可以用于分配DOM元素ID给HTML元素，HTML元素以组件的ID为前缀。例如，下面给定的HTML模版片段：

```xml
<form w-bind="./widget">
    ...
    <button type="submit" w-id="submitButton">Submit</button>
    <button type="button" w-id="cancelButton">Cancel</button>
</form>
```

假定这个分配给组件的唯一的ID是 `w123`，会输出如下的HTML：

```xml
<form id="w123">
    ...
    <button type="submit" id="w123-submitButton">Submit</button>
    <button type="button" id="w123-cancelButton">Cancel</button>
</form>
```

最终，引用一个组件的嵌套DOM元素的代码如下，它可用于容易组件：

```javascript
var submitButton = this.getEl('submitButton'); // submitButton.id === 'w123-submitButton'
var cancelButton = this.getEl('cancelButton'); // cancelButton.id === 'w123-cancelButton'

submitButton.style.border = '1px solid red';
```

这个对象通过 `this.getEl(id)` 返回一个元素[HTML element](https://developer.mozilla.org/en-US/docs/Web/API/element)。如果你想获得jQuery封装的元素，你可以用下面的任意一种方法：


法 1) 直接使用 jQuery:

```javascript
var $submitButton = $(this.getEl('submitButton'));
```

法 2) 使用 `this.$()` 方法:

```javascript
var $submitButton = this.$('#submitButton');
```

Marko组件同样支持引用 _重复的_  嵌套DOM元素，如下：

```xml
<ul>
	<li for="color in ['red', 'green', 'blue']"
		w-id="colorListItems[]">
		$color
	</li>
</ul>
```

容易组件可以通过使用 `this.getEls(id)` 来引用这些这些重复的DOM元素，如下：

```javascript
var colorListItems = this.getEls('colorListItems');
// colorListItems will be an Array of raw DOM <li> elements
```

## 添加事件监听器

Marko组件支持给嵌套DOM元素和嵌套组件添加事件监听器。事件监听器可以声明的注册在Marko模版中，或JavaScript代码中。

### 添加DOM事件监听器

一个组件可以在嵌套DOM元素中来订阅一些事件。

监听器可以用如下的示例代码被声明：

```xml
<div w-bind>
	<form w-onsubmit="handleFormSubmit">
		<input type="text" value="email" w-onchange="handleEmailChange">
		<button>Submit</button>
	</form>
</div>
```

在组件中：

```javascript

module.exports = require('marko-widgets').defineComponent({
	// ...

	handleFormSubmit: function(event, el) {
		event.preventDefault();
		// ...
	},

	handleEmailChange: function(event, el) {
		var email = el.value;
		this.validateEmail(email);
		// ...
	},

	validateEmail: function(email) {
		// ...
	}
});
```

注：事件处理方法可以通过 `this` 被引用，`this`是组件实例，而且下面的的两个参数被用来提供处理方法：

1. `event` - 原生的DOM事件对象（例如： `event.target`, `event.clientX`等）。
2. `el` - 监听器依赖的元素（根据冒泡的不同，会有别于 `event.target`）。

For performance reasons, Marko Widgets only adds one event listener to the root `document.body` element for each event type that bubbles. When Marko Widgets captures an event on `document.body` it will internally delegate the event to the appropriate widgets. For DOM events that do not bubble, Marko Widgets will automatically add DOM event listeners to each of the DOM nodes. If a widget is destroyed, Marko Widgets will automatically do the appropriate cleanup to remove DOM event listeners.

You can also choose to add listeners in JavaScript code by assigning an "element id" to the nested DOM element (only needs to be unique within the scope of the containing widget) so that the nested DOM element can be referenced by the containing widget. The scoped widget element ID should be assigned using the `w-id="<id>"` attribute. For example, in the template:

```xml
<div w-bind>
	<form w-id="form">
		<input type="text" value="email" w-id="email">
		<button>Submit</button>
	</form>
</div>
```

And then in the widget:

```javascript

module.exports = require('marko-widgets').defineComponent({
	// ...

	init: function() {
		var self = this;

		var formEl = this.getEl('form');
		formEl.addEventListener('submit', function(event) {
			self.handleFormSubmit(event, formEl)
		});

		// Or use jQuery if that is loaded on your page:
		var emailEl = this.getEl('email');
		$(emailEl).on('change', function(event) {
			self.handleEmailChange(event, emailEl)
		});
	},

	handleFormSubmit: function(event, el) {
		event.preventDefault();
		// ...
	},

	handleEmailChange: function(event, el) {
		var email = el.value;
		this.validateEmail(email);
		// ...
	},

	validateEmail: function(email) {
		// ...
	}
});
```

### Adding Custom Event Listeners

A widget can subscribe to events on nested widgets. Every widget extends [EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter) and this allows each widget to emit events.

Listeners can be attached declaratively as shown in the following sample code:

```xml
<div w-bind="./widget">
	<app-overlay title="My Overlay"
		w-onBeforeHide="handleOverlayBeforeHide">

		Content for overlay

	</app-overlay>
</div>
```

And then in the widget:

```javascript
module.exports = require('marko-widgets').defineComponent({
	// ...

	handleOverlayBeforeHide: function(event) {
        console.log('The overlay is about to be hidden!');
    }
});
```

You can also choose to add listeners in JavaScript code by assigning an "id" to the nested widget (only needs to be unique within the scope of the containing widget) so that the nested widget can be referenced by the containing widget. The scoped widget ID should be assigned using the `w-id="<id>"` attribute. For example, in the template:

```xml
<div w-bind="./widget">
	<app-overlay title="My Overlay"
		w-id="myOverlay">

		Content for overlay

	</app-overlay>
</div>
```

And then in the widget:

```javascript
module.exports = require('marko-widgets').defineComponent({
	// ...

	init: function() {
		var self = this;

		var myOverlay = this.getWidget('myOverlay');

		this.subscribeTo(myOverlay)
			.on('beforeHide', function(event) {
				self.handleOverlayBeforeHide(event);
			});
	},

	handleOverlayBeforeHide: function(event) {
        console.log('The overlay is about to be hidden!');
    }
});
```

NOTE: `subscribeTo(eventEmitter)` is used to ensure proper cleanup if the subscribing widget is destroyed.

## Lifecycle Methods

### Rendering Methods

`this` should _not_ be used in these methods because a widget instance has not yet been created during rendering.

#### getInitialProps(input, out)

This optional method is used to normalize the input properties during the rendering of a UI component. If implemented, this method should return the input properties to use based on the provided `input` and `out` arguments.

```javascript
{
	getInitialProps: function(input, out) {
		return {
			name: input.name.toUpperCase()
		}
	},
	...
}
```

#### getInitialState(input, out)

This optional method is used to determine the initial state for a newly rendered UI component.

```javascript
{
	getInitialState: function(input, out) {
		return {
			counter: input.counter == null ? 0 : input.counter
		}
	},
	...
}
```

#### getTemplateData(state, input, out)

This optional method is used to determine what data will be passed to the Marko template that is used to render the UI component.

#### getWidgetConfig(input, out)

This optional method is used to determine is passed to the widget constructor when the widget is initialized in the browser. If the UI component is rendered on the server then the widget config data will be serialized to a JSON-like data structure and stored in a special `data-w-config` attribute in the DOM.

#### getInitialBody(input, out)

This optional method is used to determine the nested external content that is to be injected into the body of the UI component (to support transclusion). The actual injection point is determined by the `w-body` attribute.

### Widget Methods

`this` can be used in these methods as the widget instance.

#### init(widgetConfig)

The `init(widgetConfig)` constructor method is called once in the browser when the widget is first created and after the widget has been mounted in the DOM. The `init(widgetConfig)` method is only called once for a given widget.

#### onBeforeUpdate()

The `onBeforeUpdate()` method is called when a widget's view is about to be updated due to either new properties or a state change.

#### onUpdate()

The `onUpdate()` method is called when a widget's view has been updated due to either new properties or a state change. The DOM nodes have been updated accordingly by time this method has been called.

#### onBeforeDestroy()

The `onBeforeDestroy()` method is called when a widget is about to be destroyed due to it being fromed from the DOM.

#### onDestroy()

The `onDestroy()` method is called after a widget has been destroyed and removed from the DOM.

#### shouldUpdate(newProps, newState)

The `shouldUpdate(newProps, newState)` method is called when a widget's view is about to be updated. Returning `false` will prevent the widget's view from being updated.

## Client-side Rendering

Every widget defined using `defineComponent(...)` exports a `render(input)` method that can be used to render the widget in the browser as shown below:

```javascript
var widget = require('fancy-checkbox').render({
		checked: true,
		label: 'Foo'
	})
	.appendTo(document.body)
	.getWidget();

widget.setChecked(false);
widget.setLabel('Bar');
```

The `appendTo(targetEl)` method is only one of the methods that can be used to insert the widget into the DOM. All of the methods are listed below:

- `appendTo(targetEl)`
- `insertAfter(targetEl)`
- `insertBefore(targetEl)`
- `prependTo(targetEl)`
- `replace(targetEl)`

## Server-side Rendering

In order for everything to work on the client-side we need to include the code for the `marko-widgets` module and the `./widget.js` module as part of the client bundle and we also need to use the custom `<init-widgets>` tag to let the client know which widgets rendered on the server need to be initialized on the client. To include the client-side dependencies will be using the [lasso](https://github.com/lasso-js/lasso) module and the taglib that it provides. Our final page template is shown below:

__src/pages/index/template.marko:__

```xml
<lasso-page name="index" package-path="./browser.json" />

<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Marko Widgets: Bind</title>
    <lasso-head/>
</head>
<body>
    <div>Marko Widgets: Bind</div>

    <div class="my-component" w-bind="./widget">
        <div>Click Me</div>
    </div>

    <lasso-body/>
    <init-widgets/>
</body>
</html>
```

The `browser.json` that includes the required client-side code is shown below:

__src/pages/index/browser.json:__

```javascript
{
    "dependencies": [
        "require: marko-widgets",
        "require: ./widget"
    ]
}
```

In the above example, the final HTML will be similar to the following:

```xml
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Widgets Demo</title>
    </head>
    <body>
		<div>Marko Widgets: Bind</div>
		<div class="my-component" id="w0" data-widget="/src/pages/index/widget">
			<div>Click Me</div>
		</div>
        <script src="static/index-8947595a.js" type="text/javascript"></script>
        <span style="display:none;" data-ids="w0" id="rwidgets"></span>
    </body>
</html>
```

To try out and experiment with this code please see the documentation and source code for the [widget-bind](https://github.com/marko-js-samples/widget-bind) sample app.

### Manually Initializing Server-side Rendered Widgets

It's also possible to manually initialize rendered widgets as shown in the following code:

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

And then, in the browser, the following code can be used to initialize the widgets:

```javascript
var result = JSON.parse(response.body);
var html = result.html
var renderedWidgets = result.renderedWidgets;

document.body.innerHTML = html; // Add the HTML to the DOM

// Initialize the widgets to bind behavior!
require('marko-widgets').initWidgets(renderedWidgets);
```

NOTE: the server side example above renders the template directly and therefore circumvents the
`index.js` file (neither `getInitialState()` nor `getTemplateData()` are executed).

To render the complete widget, use the code below instead
(the browser side is not affected; the same code snipped can be used):

```javascript
var markoWidgets = require('marko-widgets');
var helloComponent = require('src/components/app-hello');

module.exports = function(req, res) {
    var renderResult = helloComponent.render(viewModel);
    var renderedWidgets = markoWidgets.getRenderedWidgets(renderResult.out);

    // Serialize the HTML and the widget IDs to the browser
    res.json({
        html: renderResult.html,
        renderedWidgets: renderedWidgets
    });
}
```

## Split Renderer and Widget

For UI components that will only be rendered on the server it may be desirable to split the renderer (i.e. rendering logic and template) from the client-side behavior (i.e. widget). This can be done by using `defineRenderer(def)` and `defineWidget(def)` instead of `defineComponent(def)`. An example of a combined and split UI component is shown below.

### Combined Renderer and Widget

```
src/components/app-hello/
├── index.js
└── template.marko
```

___src/components/app-hello/template.marko:___

```xml
<div w-bind
	w-on-click="handleClick">
	Hello ${data.name}!
</div>
```

___src/components/app-hello/index.js:___

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

### Split Renderer and Widget

```
src/components/app-hello/
├── index.js
├── renderer.js
├── template.marko
└── widget.js
```

___src/components/app-hello/template.marko:___

```xml
<div w-bind="./widget"
	w-on-click="handleClick">
	Hello ${data.name}!
</div>
```

___src/components/app-hello/renderer.js:___

```javascript
module.exports = require('marko-widgets').defineRenderer({
	template: require('./template.marko'),

	getTemplateData: function(state, input) {
		return {
			name: input.name
		};
	}
});
```

___src/components/app-hello/widget.js:___

```javascript
module.exports = require('marko-widgets').defineWidget({
	handleClick: function() {
		this.el.style.backgroundColor = 'yellow';
	}
});
```

___src/components/app-hello/index.js:___

```javascript
exports.render = require('./renderer').render;
```
