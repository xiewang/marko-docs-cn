常见问题
===================

<!--{TOC}-->

# 构建一个应用的时候，建立可重用的组件的难点在哪？

作为一个开发者，要在构建“可重用”的UI组件和将应用和组件紧密绑定在一起之间做出选择－就是可重用和简单化之间的权衡。

可重用的UI组件需要发出普通的事件，而紧密绑定的组件会直接和应用实例对话，来控制应用。因此，可重用的组件会很复杂，因为它会依赖父组件来控制事件。相对而言，紧密绑定的组件不需要中间件就可以直接控制应用。

根据每个应用情况各自特性做决定，应用开发者需要考虑组件可重用是否会带来好处。

# 可重用和保护Marko Widgets上下文有什么区别？

DOM节点在重渲染的时候可被保护，组件实例在重渲染的时候可被重用（为了让组件在重渲染的之后可以拥有一样的组件）。被保存的DOM节点会完全留在它之前的状态中（而不会用一个新的父节点插入DOM中）。

保护下来的DOM节点会从DOM中分离，并插入到更新好的DOM中的合适位置。

重用相同的组件实例可以确保和旧组件实例相关的引用仍然正确。

# UI应该“组件化”到什么程度？

页面分解成独立的组件，到底需要分解到什么程度。我们的目标应该是让每个UI组件专注（Focused）、独立（Independent）、可重用（Reusable）、小巧（Small）、易测试（Testable）（FIRST](http://addyosmani.com/first/)）。如果你感觉UI组件不满足这些要求，那就分解成更小的UI组件。

# 什么时候应该使用自定义组件的主体？

有很多方法使用HTML写一个东西。例如，HTML按钮可以用下面两种方法来定义：

```html
<input type="button" value="My Button">

<button type="button">
    My Button
</button>
```

上面的例子中，会生成同一个按钮，但是当使用 `<input>` 标签，按钮的值会在 `value` 属性中，当使用 `<button>` 标签，按钮的值在嵌套主体内容中。这一点很重要，HTML属性 _不允许_ 有HTML内容，但HTML内容可以在HTML元素主体中。当设计一个UI组件时，如果很有必要将输出内容提供给组件，其中包含了标记，那么就可以将标记作为主体内容的一部分。

# 运行时会引用哪些组件函数，引用的顺序是什么？

请参阅 [Component Lifecycle](./component-lifecycle.md).

# Marko Widgets支持批量DOM更新，但是这对开发者来说有什么用？

DOM的批量更新可以阻止每次状态改变都触发DOM的更新。如果组件由 `setState()` 或者 `setProps()` 导致DOM的更新，组件会加入到队列中，在下一批处理中批量更新（组件只会在排到队列中一次）。

例如，下面的代码会给同一个状态重复设新的值：

```javascript
this.setState('name', 'Frank');
this.setState('name', 'Jane');
this.setState('name', 'John');
```

DOM只会为组件更新一次，这取决于 `name` 状态属性的最后值。

当处理从事件循环中脱离出来的冒泡DOM时间，Marko Widgets会开始批量处理。就是说，当所有代码有机会回应DOM时间后，DOM就会更新。如果组件排成队列来更新并且批处理还没有开始，那么新的批处理会自动开始，并用 `process.nextTick()` 来安排更新。

# 组件之间如何通信？

每个组件都是一个[EventEmitter](https://nodejs.org/api/events.html#events_class_events_eventemitter)实例，典型的组件通信方式是通过释放自定义事件，这样的话这些事件就会被组件父级直接处理。父组件可以选择处理事件或者释放另一个可以冒泡到它父级的自定义事件。一个组件应该只和它自己的嵌套组件（例如，嵌套组件会被引入到容器组件的模版中）直接通信。组件可以通过使用 `this.getWidget(nestedWidgetId)` 方法（`nestedWidgetId` 是使用 `w-id` 属性分配的ID）获得嵌套组件的直接引用。

在某些情况下，在一个全局的 发布/订阅 通道中和事件通信是很有帮助的。对于那种一个组件需要按层级冒泡处理一个非常复杂的组件的情况，是很难的，那么 发布/订阅 就很有用。[raptor-pubsub](https://github.com/raptorjs/raptor-pubsub)模块提供了一个非常简单的发布/订阅实施方案，它是基于[EventEmitter](https://nodejs.org/api/events.html#events_class_events_eventemitter) API。

# 为什么Marko Widgets的状态很重要？

评估了[React](https://facebook.github.io/react/)的一些很棒的概念后，我们引入了有状态组件的概念后到Marko Widgets中。通过让组件有状态并追踪状态的改变，Marko Widgets运行时可以最小化DOM的更新。Marko Widgets和React都希望让开发者通过推广写代码来手动更新DOM的渲染方式，建立更多简单可维护的应用。如果出于心能考虑，Marko Widgets仍然可以让开发者更新DOM，但这不应该一直这么用。

一个有状态的组件视图只会由于它的状态值改变才会更新。此外，当渲染一个组件树的时候，只有特定的需要更新的组件才会被更新，其他的组件仍会继续使用。当用嵌套组件来渲染组件的时候，如果Marko Widgets遇到了一个先前已经用相同ID在DOM中渲染的组件，那么这个先前的组件会被重用，以防止重渲染整个组件子树。Marko Widgets也有容器组件的概念，它会接受外部的嵌套内容。如果容器组件的状态改变，只有外面的壳会被重新渲染，而不会影响到嵌套的内容。

由于新能的原因，当判断状态值改变的时候，只作一个简单的对比。这就意味着存在状态中的复杂的对象会被视为不变的，或者Marko Widgets需要被告知什么时候一个状态值使用 `setStateDirty(name)` 已经被改变。两种结局方案如下：

___不可变对象和写时拷贝：___

```javascript
function addColor(newColor) {
    // Create a new Array with the new color added:
    var newColors = this.state.colors.concat([newColor]);

    // Set the colors state with the new Array:
    this.setState('colors', newColors);
}
```

___使用 setStateDirty：___

```javascript
function addColor(newColor) {
    // Modify the existing colors array
    this.state.colors.push(newColor);

    // Let Marko Widgets know that the colors Array was modified:
    this.setStateDirty('colors');
}
```

不像React，Marko Widgets 不会试图维护一个虚拟DOM树。这会让Marko Widgets运行时更加轻小。Marko Widgets假设直接渲染DOM通常情况下已经很快。在特别关心性能的情况下，开发者可以选择提供自定义状态更新处理器来手动更新DOM。此外，Marko Widgets允许开发者标记整个DOM子树为“保存的”，那么DOM每部分就永远不会被渲染，却在整个渲染过程中仍可继续。
