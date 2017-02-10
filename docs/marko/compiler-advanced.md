编译器高级功能
====================

Marko编译器是为了能够输入Marko模版，然后输出JavaScript程序。Marko编译设计得非常灵活，可以让开发者控制如何生成JavaScript代码。

# 编译器过程

这里有三个主要的Marko编译器过程，分别是：解析、转换和生成：

- __解析__ - 解析模版源来生成一个 [抽象树（AST）](https//en.wikipedia.org/wiki/Abstract_syntax_tree)。
- __转换__ - 转换AST (add/remove/modify/rearrange nodes)
- __生成__ - 通过最终的AST生成编译好的JavaScript代码。

## 解析阶段

Marko编译器的第一阶段是将源模版字符串解析成一个初始AST。

比如输入下面的模版：

```xml
<div if(data.name)>
    Hello ${data.name}!
</div>
```

将会生成如下的AST：

```json
{
  "type": "TemplateRoot",
  "body": [
    {
      "type": "HtmlElement",
      "tagName": "div",
      "attributes": [
        {
          "name": "if",
          "argument": "data.name"
        }
      ],
      "body": [
        {
          "type": "Text",
          "argument": {
            "type": "Literal",
            "value": "\n    Hello "
          }
        },
        {
          "type": "Text",
          "argument": "data.name"
        },
        {
          "type": "Text",
          "argument": {
            "type": "Literal",
            "value": "!\n"
          }
        }
      ]
    }
  ]
}
```

----------

_提示:_ 如果你想要用这么一串AST树来debugging，你可以用下面的代码：

```javascript
console.log(JSON.stringify(astNode, null, 2));
```

----------

## 转换阶段

在转换阶段，AST会被用来生成正确的编译的代码。例如，AST转换器被用来生成如下的特殊的属性值：

- `if()`
- `else-if()`
- `else`
- `for()`
- etc.

在这个例子中的`if()` 属性，这个节点是由内置的Marko转换器（特别的，[taglibs/core/core-transformer.js](../taglibs/core/core-transformer.js)）转换而来，这个转换器时用来包裹真实的 `If` 节点，代码和下面的类似：

```javascript
var ifAttr = elNode.getAttribute('if');
if (ifAttr && ifAttr.argument) {
    // Remove the if() attribute from the HTML element node
    elNode.removeAttribute('if');

    // Create a new "If" node using the provided "builder"
    // (described later)
    var ifNode = builder.ifStatement(ifAttr.argument);

    //Surround the existing node with an "If" node
    node.wrap(ifNode);
}
```

继续用之前的例子，转换阶段过后，AST会变成如下的样子：

```json
{
  "type": "TemplateRoot",
  "body": [
    {
      "type": "If",
      "test": "data.name",
      "body": [
        {
          "type": "HtmlElement",
          "tagName": "div",
          "attributes": [],
          "body": [
            {
              "type": "Text",
              "argument": {
                "type": "Literal",
                "value": "\n    Hello "
              }
            },
            {
              "type": "Text",
              "argument": "data.name"
            },
            {
              "type": "Text",
              "argument": {
                "type": "Literal",
                "value": "!\n"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

你会注意到转换完的AST，`<div>` 标签被新的 `If` 节点包裹。等到AST转换结束，就可以生成编译好的JavaScript代码。

在转换阶段，整个AST会被多次翻译。直到没有更多的没有更多节点转换位置，整个转换阶段才结束。

## 生成阶段

生成阶段时Marko编译器的最后阶段。在生成阶段，Marko编译器会遍历AST树来生成最后的JavaScript代码。树中的每个节点都有可能生成JavaScript代码。Marko编译器提供了一个 [`CodeGenerator`](../compiler/CodeGenerator.js) 类有一个API来JavaScript代码片段，这样最后就很容易生成结构良好并且易读的JavaScript代码输出。

每一个节点必须实现下面的一个方法：

- `generateCode(generator)`
- `generate<OUTPUT_TYPE>Code(generator)` (e.g. `generateHtmlCode(generator)`)

`generator`属性会成为[`CodeGenerator`](../compiler/CodeGenerator.js)的一个实例。

Marko 编译器会根据“输出类型”的不同提供不同的编译模版。目前还只支持“Html”。用“Html”的输出类型，编译好的模版会成为执行后生成HTML字符串输出的这样一个程序。未来，我们会支持其他的输出类型，比如DOM, 虚拟 DOM, 增量（incremental） DOM，等。例如，用"DOM"输出类型，编译程序会用浏览器的DOM API来生成DOM树作为输出（而不是一个HTML字符串）。

下面的代码片段是 `If` 节点用来生成JavaScript代码的：

```javascript
generator.write('if (');
generator.generateCode(test);
generator.write(') ');
generator.generateBlock(body);
if (elseStatement) {
    generator.write(' ');
    generator.generateCode(elseStatement);
} else {
    generator.write('\n');
}
```

