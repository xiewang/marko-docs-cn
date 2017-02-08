编译器 API
================

---
# 1、require('marko/compiler')

## 方法

### buildTaglibLookup(dirname)

返回一个 [TaglibLookup](#TaglibLookup) 来寻找某个目录下模板有哪些可用的定制标签。

用法样例:

```javascript
var taglibLookup  =
    require('marko/compiler').buildTaglibLookup('some/dir');

taglibLookup.forEachTag((tag) => {
    console.log(tag.name);
});

taglibLookup.forEachAttribute('div', (attr) => {
    console.log(attr.name);
});
```

### compile(src, filename, options, callback)

### compileFile(filename, options, callback)

### createBuilder(options)

### createWalker(options)

### parseRaw(templateSrc, filename)

## 值

<a name="TaglibLookup"></a>
### taglibLookup

返回[taglib-lookup](#taglib-lookup)模块相关的东西

### taglibLoader

### taglibFinder

---
<a name="taglib-lookup"></a>
# taglib-lookup

## Methods

### registerTaglib = registerTaglib;

### buildLookup(dirname)

### clearCache();

---
# 2、AST

## 节点

这个 `Node` 是所有的AST节点的基类。所有加入到AST的AST节点都必须继承 `Node`


## 方法

### wrap(wrapperNode)

让当前的节点成为 `wrapperNode` 子节点。如下：


```javascript
this.container.replaceChild(wrapperNode, this);
wrapperNode.appendChild(this);
```

### makeContainer(array)

将提供的 `Array` 转换成 `ArrayContainer`。如果这个 `Array` 如果已经是一个 `Container` 的实例，那么他就会被直接返回。


### appendChild(node)

给某节点的相关容器里添加子节点。这个 `this,body` 的值会被用来作为默认容器，但是这个方法在衍生节点中会被重写

## 值

## 类型

节点类型为字符串。类型有：`"TemplateRoot"`, `"HtmlElement"`, `"Text"`, `"If"`, `"Else"`, `"ForEach"`, 等。

### 容器

如果一个 `Node` 是另一个 `Node` 的子节点，那么它会被关联到一个 `Container`。比如：

```javascript
if (this.container) {
    var parentNode = this.container.node;

    // NOTE: The following lines produce the same result:
    this.container.removeChild(this)
    this.detach()
} else {
    // Either the node is the root node or it is detached from the AST
}
```

---
# 3、Builder

## 方法

### arrayExpression(elements)

### assignment(left, right)

返回一个节点，这个节点会生成如下的代码：

```javascript
<left> = <right>;
```

例如：

```javascript
builder.assignment(
    builder.identifier('foo'),
    builder.literal('123'));

// Output code:
foo = '123';
```

### binaryExpression(left, operator, right)

返回一个节点，这个节点会生成如下的代码:

```javascript
<left> <operator> <right>
```

例如：

```javascript
builder.binaryExpression(
    builder.identifier('foo'),
    '<'
    builder.literal(99));

// Output code:
foo < 99;
```

### code(value)
返回一个可以写任意有指定值的JavaScript代码的节点。代码缩进被毁被正确格式化。

例如：

```javascript
builder.program([
    builder.code('var a = 1;\nvar b = 2;'),
    builder.assignment(builder.identifier('b'), builder.literal(3))
])

// Output code:
var a = 1;
var b = 2;

b = 3;
```

### conditionalExpression(test, consequent, alternate)

返回一个节点，这个节点会生成如下的代码:

```javascript
<test> ? <consequent> : <alternate>
```

例如：

```javascript
builder.conditionalExpression(
    builder.identifier('isHidden'),
    builder.literal('hidden'),
    builder.literal('visible'));

// Output code:
isHidden ? "hidden" : "visible"
```

### elseStatement(body)

返回一个节点，这个节点会生成如下的代码：

```javascript
else {
    <body>
}
```

例如：

```javascript
builder.elseStatement([
    builder.functionCall('console.log', ['hello']),
    builder.functionCall('console.log', ['world'])
]);

// Output code:
else {
    console.log('hello');
    console.log('world');
}
```

### elseIfStatement(test, body, elseStatement)

返回一个节点，这个节点会生成如下的代码：

```javascript
else if (<test>) {
    <body>
}[ <elseStatement>]
```

例如：

```javascript
builder.elseIfStatement(
    builder.literal(true),
    [
        builder.functionCall('console.log', ['hello']),
        builder.functionCall('console.log', ['world'])
    ]);

// Output code:
else if (true) {
    console.log('hello');
    console.log('world');
}
```

### forEach(def)

生成一个可以遍历数组，、对象或者一个返回的代码的节点。

___array:___

```javascript
builder.forEach({
    varName: 'color',
    target: 'colors'
    body: [
        builder.functionCall('console.log', [
            builder.identifier('color')
        ])
    ]
})


// Output code:
var forEach = __helpers.f; // Static variable

// ...

forEach(data.colors, function(color) {
  out.w(escapeXml(color));
});
```

### forEach(varName, target, body)

返回一个可以生成简单 `forEach` 方法的节点。 参照 `forEach(def)`

### forRange(def)

返回一个可以生成遍历数字范围的代码的节点。

___array:___

```javascript
builder.forRange({
    varName: 'i',
    from: 0,
    to: 'myArray.length',
    step: 2,
    body: [
        builder.functionCall('console.log', ['hello'])
    ]
})


// Output code:
(function() {
  for (var i = 0; i<=myArray.length; i+=2) {
    console.log(i);
  }
}());
```

### forRange(varName, from, to, step, body)

返回一个可以生成简单 `forRange` 方法的节点。参照  `forRange(def)`。

### forStatement(init, test, update, body)

返回一个节点，这个节点会生成如下的代码：

```javascript
for (<init>; <test>; <update>) {
    <body>
}
```

例如：

```javascript
builder.forStatement(
    builder.vars([
        {
            id: 'i',
            init: builder.literal(0)
        }
    ]),
    builder.binaryExpression('i', '<', builder.literal(0)),
    builder.updateExpression('i', '++'),
    [
        builder.functionCall('console.log', [
            builder.identifier('i')
        ])
    ]
)

// Output:
for (var i = 0; i < 0; i++) {
  console.log(i);
}
```

### functionCall(callee, args)

返回一个节点，这个节点会生成如下的代码：

```javascript
<callee>(<arg1, arg2, ..., argN>)
```

```javascript
builder.functionCall(
    'console.log',
    [
        builder.literal('Hello'),
        builder.identifier('name')
    ]);

// Output:
console.log('Hello', name);
```

### functionDeclaration(name, params, body)

返回一个节点，这个节点会生成如下的代码：

```javascript
function [name](<param1, param2, ..., paramN>) {
    <body>
}
```

_具名函数声明：_

```javascript
builder.functionDeclaration(
    'foo',
    [
        'num1',
        'num2'
    ],
    [
        builder.returnStatement(builder.binaryExpression('num1', '+', 'num2'))
    ]);

// Output:
function add(num1, num2) {
  return num1 + num2;
}
```

_匿名方法声明：_

```javascript
builder.functionDeclaration(
    null,
    [
        'num1',
        'num2'
    ],
    [
        builder.returnStatement(builder.binaryExpression('num1', '+', 'num2'))
    ]);

// Output:
function(num1, num2) {
  return num1 + num2;
}
```

### html(argument)

返回一个可以渲染HTML的节点（支持HTML的特殊字符）

Returns a node that renders a fragment of HTML (special HTML characters will not be escaped):

```javascript
builder.html(
    builder.literal('<div>Hello World</div>')
);

// Output:
out.w("<div>Hello World</div>");
```

### htmlComment(comment)

```javascript
builder.htmlComment(
    builder.literal('This is an HTML comment'))

// Output:
out.w("<--This is an HTML comment-->");
```

### htmlElement(tagName, attributes, body, argument, openTagOnly, selfClosed)

```javascript
builder.htmlElement(
    'div',
    [
        {
            name: 'class',
            value: builder.literal('greeting')
        }
    ],
    [
            builder.text(builder.literal('Hello World'))
    ])

// Output:
out.w("<div class=\"greeting\">Hello World</div>");
```

### identifier(name)

返回一个可以生成JavaScript标志符代码的节点（例如：变量，参数，属性，等。）

例如：

```javascript
builder.assignment(
    builder.identifier('foo'),
    builder.literal('abc'))

// Output code:
foo = "abc"
```

### ifStatement(test, body, elseStatement)

返回一个节点，这个节点会生成如下的代码：

```javascript
if (<test>) {
    <body>
}[ <elseStatement>]
```

例如：

```javascript
builder.ifStatement(
    builder.literal(true),
    [
        builder.functionCall('console.log', ['hello']),
        builder.functionCall('console.log', ['world'])
    ]);

// Output code:
if (true) {
    console.log('hello');
    console.log('world');
}
```

### invokeMacro(name, args, body)

返回一个可以生成利用制定名称、参数、内容来引用宏指令代码的节点。

例如：

```javascript
builder.invokeMacro(
    'greeting',
    [
        builder.literal('Frank'),
        builder.literal(10),
    ],
    [
        builder.text(builder.literal('This is the body passed to the macro'))
    ])
```

### invokeMacroFromEl(el)

返回一个可以生成基于指定 `HtmlElement` 节点的引用宏指令代码的节点。

例如：

```javascript
var el = builder.htmlElement('greeting', {
        name: builder.literal('Frank'),
        age: builder.literal(10)
    });
builder.invokeMacroFromEl(el)
```

### literal(value)

返回一个可以生成文本字符串、数字、布尔、对象和数组的节点。

例如：

```javascript
builder.literal('abc');

// Output code:
"abc"
```

或者，如下更复杂的例子:

```javascript
builder.vars({
    'aString': builder.literal('abc'),
    'aNumber': builder.literal(123),
    'aBoolean': builder.literal(false),
    'anObject': builder.literal({
        foo: 'bar',
        dynamic: builder.expression('data.name'),
    }),
    'anArray': builder.literal([
        'foo',
        builder.expression('data.name')
    ])
})

// Output code:
var aString = "abc",
    aNumber = 123,
    aBoolean = false,
    anObject = {
      "foo": "bar",
      "dynamic": data.name
    },
    anArray = [
      "foo",
      data.name
    ]
```

### logicalExpression(left, operator, right)

### macro(name, params, body)

返回一个可以生成有指定名称、参数和具体内容的宏方法的节点。这个 `InvokeMacro` 节点应该用来生成用来引用这个宏的代码。

### negate(argument)

返回一个节点，这个节点会生成如下的代码：

```javascript
!<argument>
```

例如：

```javascript
builder.negate(builder.identifier('foo'))

// Output:
!foo
```

### newExpression(callee, args)

### node([type, ]generatCode)

返回一个普通 `Node` 实例，这个实例有既定的节点类型（可选）和一个用来给节点生成代码的方法`generateCode(node, generator)`。如果这个 `generateCode(node, generator)` 方法提供，这个节点会添加一个补丁方法`generateCode(generator)`。

例如：

```javascript
builder.node(function(node, generator) {
    var builder = generator.builder;
    return builder.text(builder.literal('Hello World!'));
})

// Output code:
out.w("Hello World!");
```

### objectExpression(properties)

### program(body)

返回一个可以生成JavaScript的根声明代码的节点。

例如：

```javascript
builder.program([
    builder.vars({
        name: builder.literal('Frank')
    }),
    builder.functionCall('console.log', [
        builder.literal('Hello'),
        builder.identifier('name')
    ])
])

// Output code:
var name = "Frank";

console.log("Hello", name);
```

### property(key, value)

### require(path)

返回一个节点，这个节点会生成如下的代码：

```javascript
require(<path>)
```

短写法如下：

```javascript
builder.functionCall('require', [path])
```

### returnStatement(argument)

返回一个节点，这个节点会生成如下的代码：

```javascript
return[ <argument>]
```

例如：

```javascript
builder.functionDeclaration(
    'upperCase',
    [
        'str'
    ],
    [
        builder.returnStatement(builder.functionCall('str.toUpperCase'))
    ]
)

// Output code:
function upperCase(str) {
  return str.toUpperCase();
}
```

### selfInvokingFunction(params, args, body)

返回一个节点，这个节点会生成如下的代码：

```javascript
(function(<params>) {
    <body>
}(<args>))
```

例如：

```javascript
builder.selfInvokingFunction(
    [
        'win'
    ],
    [
        'window'
    ],
    [
        builder.assignment('win.foo', builder.literal('bar'))
    ])

// Output code:
(function(win) {
  win.foo = "bar";
}(window))
```

或者是传入空参数：

```javascript
builder.selfInvokingFunction(null, null, [
        builder.vars(['foo']),
        builder.assignment('foo', builder.literal('bar'))
    ])

// Output code:
(function() {
  var foo;

  foo = "bar";
}())
```

### selfInvokingFunction(body)

等价于 `selfInvokingFunction(null, null, body)`.

### slot(onDone)

直到所有代码都执行完成后才返回一个节点。这个对于一个未知的碎片工具在其他代码已经生成后才执行的情况是很有帮助的。

举个例子：[TemplateRoot](../compiler/ast/TemplateRoot.js) 节点用slot方法来延迟生成这个已经编译的模版的静态变量部分。到所有的节点生成的代码都已知前，静态的变量需要添加到编译的模版顶部。

```javascript
builder.program([
    builder.slot((slot, generator) => {
        slot.setContent(generator.builder.vars(vars));
    }),
    builder.node(function(node, generator) {
        vars.push({
            id: 'foo',
            init: generator.builder.literal('abc')
        });
    }),
    builder.node(function(node, generator) {
        vars.push({
            id: 'bar',
            init: generator.builder.literal(123)
        });
    })
])

// Output code:
var foo = "abc",
    bar = 123;
```

### strictEquality(left, right)

返回一个节点，这个节点会生成如下的代码：

```javascript
<left> === <right>
```

例如：

```javascript
builder.strictEquality('a', 'b')

// Output code:
a === b
```

### templateRoot(body)

### text(argument, escape)

### thisExpression()

### unaryExpression(argument, operator, prefix)

### updateExpression(argument, operator, prefix)

### variableDeclarator(id, init)

### vars(declarations, kind)

