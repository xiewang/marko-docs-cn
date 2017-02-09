编译时标签
=================

Marko允许开发者在编译时控制一个自定义标签如何生成JavaScript代码。开发者可以给自定义标签提供一个“代码生成器”的方法。

# 自定义标签代码生成器

让我们来看看如何通过提供我们自己的代码生成器来构建一个编译时标签。第一步时在 `marko-taglib.json` 文件中注册自定义标签。我们会用 `code-generator` 这个值来结合自定义标签，这个自定义标签有个功能，它会用来在编译时为元素生成代码：

```json
{
    "<greeting>": {
        "code-generator": "./greeting-tag"
    }
}
```
一个代码生成器模块应该暴露如下的一个方法：

```javascript
module.exports =  function generateCode(elNode, generator) {
    // ...
}
```

 `elNode` 变量会成为 [`HtmlElement`](../compiler/ast/HtmlElement.js) 的一个实例。`generator`变量会成为 [`CodeGenerator`](../compiler/CodeGenerator.js) 的一个实例。

继续用 `<greeting>` 作为例子，我们假设有如下的一个模版：

```xml
<greeting name="Frank"/>
```

让我们通过生成代码来实施这个greeting标签，最终会通过 `console.log` 输出 `Hello Frank`：

```javascript
module.exports = function generateCode(elNode, generator) {
    var builder = generator.builder;

    return builder.functionCall('console.log', [
        builder.literal('Hello'),
        elNode.getAttributeValue('name')
    ]);
};
```

上面的代码会生成下面的编译好的代码：

```javascript
console.log("Hello", data.name);
```

在上面的例子中，`generateCode(elNode, generator)`方法返回一个新的 [`FunctionCall`](../compiler/ast/FunctionCall.js) 节点，这个节点使用提供的 [`Builder`](../compiler/Builder.js)  实例。这个构建为生成的结合JavaScript原始变量的节点提供方法。这是被推荐的生成JavaScript代码的方法，应为它能够确保正确生成代码并且能合适的格式化。然而，出于演示的目的，下面的代码同样 _可行_ （但绝不推荐）： 

```javascript
module.exports = function generateCode(elNode, generator) {
    var nameValue = elNode.getAttributeValue('name');
    generator.write('console.log(');
    generator.write('"Hello"');
    generator.write(', ');
    generator.generateCode(nameValue);
    generator.write(')');
};
```

用 `console.log()` 编写标准的输出可能 _不是_ 一个自定标签开发者想要做的。你通常会写HTML的输出。继续用上面同样的例子，让我们更新接口，来生成能够产出如下HTML输出的代码：

```html
<div class="greeting">Hello Frank</div>
```

为了这个，我们会利用构建API来生成合适的节点树：

```javascript
module.exports = function generateCode(elNode, generator) {
    var builder = generator.builder;

    return builder.htmlElement(
        'div',
        {
            'class': builder.literal('greeting')
        },
        [
            builder.text(builder.literal('Hello ')),
            builder.text(elNode.getAttributeValue('name'))
        ]);
};
```

如下的模版：

```xml
<greeting name="Frank"/>
```

通过这个自定义标签生成的代码如下：

```javascript
out.w("<div class=\"greeting\">Hello Frank</div>");
```

如下的模版，它的name是一个non-literal JavaScript expression：

```xml
<greeting name=data.name/>
```

通过这个自定义标签生成的代码如下：

```javascript
out.w("<div class=\"greeting\">Hello " +
  escapeXml(data.name) +
  "</div>");
```

# 构建器 API

构建器API在为标签生成代码时起着至关重要的作用。构建器提供构建块来生成潜在的复杂JavaScript代码。
