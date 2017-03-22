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

However, Marko is much more than a templating language.  It's a UI library that allows you to break your application into components that are self-contained and describe how the application view changes over time and in response to user actions.

In the browser, when the data representing your UI changes, Marko will automatically and efficiently update the DOM to reflect the changes.

## A simple component

Let's say we have a `<button>` that we want to assign some behavior to when it is clicked:

_button.marko_
```xml
<button>Click me!</button>
```

Marko makes this really easy, allowing you to define a `class` for a component right in the `.marko` view and call methods of that class with `on-` attributes:

_button.marko_
```xml
class {
    sayHi() {
        alert(`Hi!`);
    }
}

<button on-click('sayHi')>Click me!</button>
```

### Adding state

Alerting when a button is clicked is great, but what about updating your UI in response to an action?  Marko's stateful components make this easy.  All you need to do is set `this.state` from inside your component's class. This makes a new `state` variable available to your view.  When a value in `this.state` is changed, the view will automatically re-render and only update the part of the DOM that changed.

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