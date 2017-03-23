# 入门

入门Marko的最简单的方式就是使用[在线使用](https://markojs.com/try-online) 功能。你只要在另一个标签中打开并依次尝试一下。如果你更喜欢在本地开发使用，你可以参照[安装说明](./installing.md)。

## Hello world

Marko让你很轻松使用类似HTML的语法来呈现你的UI：

_hello.marko_
```xml
<h1>Hello World</h1>
```

实际上，Marko十分像HTML，你可以用它来替换如handlebars、mustache或者pug这样的模版语言：

_template.marko_
```xml
<!doctype html>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```

然而，Marko不仅仅是一个模版语言。它也是一个UI库，能让你将你的应用分解成很多组件，这些组件相互独立、描述应用如何随时变化、响应用户的操作。

在浏览器，当数据反映出UI的改变，Marko会自动高效地根据这些改变来更新DOM。

## 一个简单的组件

比如说我们有个一个 `<button>`，我们想它被点击的时候，给它分配一些行为：

_button.marko_
```xml
<button>Click me!</button>
```

Marko做这个轻而易举，你可以直接在 `.marko` 视图中给组件定义一个 `class`，通过 `on-` 属性来调用这个类中的方法：

_button.marko_
```xml
class {
    sayHi() {
        alert(`Hi!`);
    }
}

<button on-click('sayHi')>Click me!</button>
```

### 添加状态

当点击按钮的时候，最好给出提醒，但是通过响应一个动作来更新你的UI会如何呢？Marko的有状态组件可以很轻松完成这个任务。你所要做的仅仅是在你的组件类中添加 `this.state`。这个会在你的视图中创建新的 `state`。当 `this.state` 改变后，视图会自动重新渲染，并且只会更新DOM改变的部分。

_counter.marko_
```xml
class {
    constructor() {
        this.state = {
            count:0
        };
    }
    increment() {
        this.state.count++;
    }
}

<div>The current count is ${state.count}</div>
<button on-click('increment')>Click me!</button>
```