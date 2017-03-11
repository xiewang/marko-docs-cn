概述
========

Marko Widgets继承了 [Marko templating engine](https://github.com/marko-js/marko)，它提供了一个简单且高效的给UI组件绑定行为的机制，它既可以在服务端渲染，也可以在浏览器端渲染。此外，改变组件的状态或者值可以不改变额外代码的情况下，让DOM自动更新。Marko组件吸收了很多 [React](https://facebook.github.io/react/index.html) 的设计思想，但它力求更轻更快（特别是在服务端）。当为组件更新视图时，Marko Widgets使用DOM对比让DOM的改变最小化，这个DOM对比是使用 [morphdom](https://github.com/patrick-steele-idem/morphdom) 模块来完成的。

<a href="http://markojs.com/marko-widgets/try-online/" target="_blank">在线使用 Marko Widgets！</a>

# 功能

- 简单
	- 简易的JavaScript语法用来定义组件
	- 利用[Marko templates](https://github.com/marko-js/marko) (一个基于HTML的模版语言)来展示视图
	- 支持有状态和无状态组件
	- 没有复杂的类层次
	- 简单、声明式的事件绑定，即可用于原生DOM事件，也可用于自定义事件
	- 支持组件生命周期管理（销毁和新建组件轻而易举）	- 支持事件冒泡和视图状态改变分发
	- 只需要理解一些简单的概览就能上手
- 高性能
	- 在服务端和浏览器端，快如闪电般的性能（参阅[Marko vs React: Performance Benchmark](https://github.com/patrick-steele-idem/marko-vs-react)）
	- 支持流河异步渲染
	- 在服务端和浏览器端渲染的UI组件的高效行为绑定
	- 通过下面的一些方法，完成高效的DOM更新：		- DOM对比是使用[morphdom](https://github.com/patrick-steele-idem/morphdom)模块来完成最小化的DOM改变。
		- 批量更新
		- 当重渲染一个组件时，嵌套组件会被重用。
		- 只有状态怪变的组件才会被重新渲染。
		- 如果提供类状态过渡处理器，完全的重渲染可被短路。
		- 对于容器组件，嵌套的主体DOM节点会被自动保护。
		- 整个DOM子树在渲染期间会被保护。
		- 智能的模版编译器尽可能在编译时作更多的工作	- 分成高效的事件委托
	- 使用[warp10](https://github.com/patrick-steele-idem/warp10)完成从服务器到浏览器的状态快速序列化
- 轻量化
	- 运行时的JavaScript及其小(压缩到大约 6.3 KB)
	- 不依赖于类似JQuery等任何其他的JavaScript库
	- 专注在UI视图层面（和其他的库／框架很容易混合匹配使用）F

# 设计哲学

- 一个UI组件应该封装好视图，行为和样式
- 一个复杂的页应该分解成模块化的UI组件
- UI组件应该被用来构建模块
- 一个UI组件视图应该由一个接受输入状态和输出HTML的干净的函数驱动
- 一个UI组件应该可被独立测试
- 一个UI组件不应该暴露它内部的引用
- 一个UI组件应该可依通过npm安装
- 一个UI组件应该和其他框架和库很好的融合
- 所有UI组件应该可以很轻易的被组合
- 开发者应该不需要手动去操作

# 示例代码

Marko Widgets允许你在Marko模版个红枣南瓜给HTML元素声明绑定行为。这个组件为你UI组件提供客户端行为。

## 无状态组件

__src/components/app-hello/template.marko__

```xml
<div w-bind>
	Hello ${data.name}!
</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getTemplateData: function(state, input) {
		return {
			name: input.name
		};
	},

	init: function() {
		var el = this.el; // The root DOM element that the widget is bound to
		console.log('Initializing widget: ' + el.id);
	}
});
```

恭喜你，你刚完成来一个可重用的UI组件！你的UI组件可以嵌入到其他Marko模版文件中：

```xml
<div>
	<app-hello name="Frank"/>
</div>
```

此外，你的UI组件可以重渲染，并可以使用JavaScriptAPI被添加到DOM中：

```javascript
var widget = require('./app-hello')
	.render({
		name: 'John'
	})
	.appendTo(document.body)
	.getWidget();

// Changing the props will trigger the widget to re-render
// with the new props and for the DOM to be updated:
widget.setProps({
	name: 'Jane'
});
```

## 有行为的无状态组件

__src/components/app-hello/template.marko__

```xml
<div w-bind
	 w-onClick="handleClick">

	Hello ${data.name}!

</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getTemplateData: function(state, input) {
		return {
			name: input.name
		};
	},

	handleClick: function() {
		this.setSelected(true);
	},

	setSelected: function(selected) {
		if (selected) {
			this.el.style.backgroundColor = 'yellow';
		} else {
			this.el.style.backgroundColor = null;
		}
	}
});
```

## 有状态组件

让我们创建一个有状态的组件，当你点击的时候，它可以变成黄色：

__src/components/app-hello/template.marko__

```xml
<div w-bind
	 w-onClick="handleClick"
	 style="background-color: ${data.color}">

	Hello ${data.name}!

</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getInitialState: function(input) {
		return {
			name: input.name,
			selected: input.selected || false;
		}
	},

	getTemplateData: function(state, input) {
		var style = ;

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

## 有更新处理的有状态组件

如果你想为一个特定的状态值改变而阻止组件的重新渲染，那么你可以简单地提供你自己的方法来处理这个状态改变，如下：

__src/components/app-hello/template.marko__

```xml
<div w-bind
	 w-onClick="handleClick"
	 style="background-color: ${data.color}">

	Hello ${data.name}!

</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	getInitialState: function(input) {
		return {
			name: input.name,
			selected: input.selected || false;
		}
	},

	getTemplateData: function(state, input) {
		var style = ;

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
	},

	update_selected: function(newSelected) {
		// Manually update the DOM to reflect the new "selected"
		// state" to avoid re-rendering the entire widget.
		if (newSelected) {
			this.el.style.backgroundColor = 'yellow';
		} else {
			this.el.style.backgroundColor = null;
		}
	}
});
```

## 复杂组件

```xml
<div w-bind>
	<app-overlay title="My Overlay"
		w-id="overlay"
		w-onBeforeHide="handleOverlayBeforeHide">
		Body content for overlay.
	</app-overlay>

	<button type="button"
		w-onClick="handleShowButtonClick">
		Show Overlay
	</button>

	<button type="button"
		w-onClick="handleHideButtonClick">
		Hide Overlay
	</button>
</div>
```

下面是定义组件类型的 `index.js` 文件的内容：

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	init: function() {
		// this.el will be the raw DOM element the widget instance
		// is bound to:
		var el = this.el;
	},

	handleShowButtonClick: function(event) {
		console.log('Showing overlay...');
        this.getWidget('overlay').show();
    },

    handleHideButtonClick: function(event) {
		console.log('Hiding overlay...');
        this.getWidget('overlay').hide();
    },

    handleOverlayBeforeHide: function(event) {
        console.log('The overlay is about to be hidden!');
    }
})
```

## 容器组件

一个容器组件支持嵌套内容。当容器组件重渲染时，嵌套组件会自动被保护起来。

__src/components/app-alert/template.marko__

```xml
<div class="alert alert-${data.type}" w-bind>
	<i class="alert-icon"/>
	<span w-body></span>
</div>
```

__src/components/app-alert/index.js__

```javascript
module.exports = require('marko-widgets').defineComponent({
	template: require('./template.marko'),

	init: function() {
		// this.el will be the raw DOM element the widget instance
		// is bound to:
		var el = this.el;
	},

	getInitialState: function(input) {
		return {
			type: input.type || 'success'
		}
	},

	getTemplateData: function(state, input) {
		return {
			type: state.type
		};
	},

	getInitialBody: function(input) {
		return input.message || input.renderBody;
    },

	setType: function(type) {
		this.setState('type', type);
	}
})
```

这个组件可以想如下一样被使用：

```xml
<app-alert message="This is a success alert"/>

<app-alert>
	This is a success alert
</app-alert>

<app-alert message="This is a failure alert" type="failure"/>

<app-alert type="failure">
	This is a failure alert
</app-alert>
```

## 重渲染时的DOM节点保护

有时 _不去_ 重渲染DOM子树是很重要的。为什么这样的原因可能有以下几点：

- 提高性能
- DOM节点包含额外提供的内容
- DOM节点有需要维护的内部状态

Marko Widgets通过添加一个特殊的`w-preserve`、 `w-preserve-if(<condition>)`、 `w-preserve-body` 或者 `w-preserve-body-if(<condition>)`属性在需要被保护的HTML标签中，允许DOM节点被保护。保护的DOM节点会被自动重用和重新插入组件刚渲染的DOM中。

```xml
<div w-bind>

	<span w-preserve>
		<p>
			The root span and all its children will never
			be re-rendered.
		</p>
		<p>
			Rendered at ${Date.now()}.
		</p>
	</span>
	<div w-preserve-body>
		Only the children of the div will preserved and
		the outer HTML div tag will be re-rendered.
	</div>

	Don't rerender the search results if no search results
	are provided.
	<app-search-results items="data.searchResults"
		w-preserve-if(data.searchResults == null)/>
</div>
```

## 重渲染时的DOM属性保护

类似于保护DOM节点，Marko Widgets同样让保护DOM节点上的特定属性变得可能。如果一个分离的库正在更改DOM属性，并且这些更改应该在重渲染的时候被保护，这个保护功能就变得很有用。当使用一个动画引擎的时候（如[Velocity.js](http://julian.com/research/velocity/) 或者 [GSAP](http://greensock.com/gsap)），通常情况下会使用 `class` 和 `style` 属性。

`w-preserve-attrs` 属性可以应用到任意DOM元素中，并且支持用逗号分开的属性名序列，如下：

```xml
<div w-preserve-attrs="class,style">
	...
</div>
```