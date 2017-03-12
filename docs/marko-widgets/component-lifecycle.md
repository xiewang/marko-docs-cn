组件生命周期
===================

<!--{TOC}-->

渲染一个UI组件会让顶层的UI组件和所有嵌套UI组件被渲染。渲染的输出内容是是一个HTML字符串，它包含所有渲染UI组件的输出。当这个HTML字符串添加到DOM的时候，任何和这个渲染好的UI组件相关的部件都会被初始化。嵌套的小组件会在它的父组件前初始化。

# 组件渲染

当一个组件被渲染，下面的可选函数会依次调用：

1. `getInitialProps(input)`
2. `getInitialState(input)`
3. `getTemplateData(state, input)`
4. `getInitialBody(input)`
5. `getWidgetConfig(input)`

每个渲染方法会在下面的章节中介绍。


## 渲染方法

`this` 不应在这些方法使用，因为一个组件实例在渲染的时候还没有被建立。

### getInitialProps(input, out)

这个可选方法是用来在UI组件渲染阶段规范化输入值。如果方法被执行，它会返回基于提供的`input` 和 `out` 参数的可用输入值。

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

### getInitialState(input, out)

这个可选的方法是用来为最新渲染的UI组件确定初始状态。

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

### getTemplateData(state, input, out)

这个可选方法是用来确定哪些数据会被传递到用来渲染UI组件的Marko模版中。

### getWidgetConfig(input, out)

这个可选方式是用来当组件初始化后，在浏览器中确定哪些数据传递到组件构造函数中。如果UI组件在服务器中渲染好，那么这个组件配置数据会序列化到类JSON的数据结构汇总，并且会存到DOM中一个特定的 `data-w-config` 属性中。

### getInitialBody(input, out)

这个可选的方法是用来确定这些嵌套的外部内容，这些内容会植入到UI组件的body里。实际如何植入，是由 `w-body` 属性来决定。

# 组件生命周期

UI组件的DOM节点添加到DOM后，一个组件实例会被创建，并绑定到相应的DOM节点。

## 组件生命周期方法

`this` 可以在这些组件实例中使用。组件生命周期方法是可选的方法，对应组件的各个生命周期事件，它的实现会被Marko组件运行时调用。例如：

```javascript
module.exports = require('marko-widgets').defineComponent({
	// ...
	//
	init: function() {
		console.log('The UI component has been mounted to the DOM.', this.id);
	},

	onBeforeUpdate: function() {
		console.log('The DOM is about to be updated...', this.id);
	},

	onUpdate: function() {
		console.log('The DOM has been updated.', this.id);
	},

	onDestroy: function() {
		console.log('The UI component is being removed from the DOM :(', this.id);
	}
});
```

### init(widgetConfig)

当组件被首次创建并在DOM中被挂载后，`init(widgetConfig)` 构造函数方法在浏览器中会被立即调用。`init(widgetConfig)` 方法只会被指定的组件调用。

### onBeforeUpdate()

当组件的视图由于新的值或者状态改变而将要被更新的时候，`onBeforeUpdate()` 方法会被调用。

### onUpdate()

当组件视图由于新的值或者状态改变而已经被更新的时候，`onUpdate()` 方法会被调用。这个方法调用以后，DOM节点就会被更新。

### onRender(event)

当组件已经被渲染时（或渲染成功后）会被调用，并挂载到DOM上。`event` 参数是一个对象。如果这事件间正被这第一个渲染器引发，那么  `event` 参数会将 `firstRender` 值设为 `true`。

### onBeforeDestroy()

当组件将要被销毁的时候，`onBeforeDestroy()` 会被调用。

### onDestroy()

在组件已经被销毁并从DOM中移除的时候，`onDestroy()` 方法会被调用。

### shouldUpdate(newProps, newState)

当一个组件视图将要被更新的时候，`shouldUpdate(newProps, newState)` 方法会被调用。返回的 `false` 会阻止组件视图的更新。
