JavaScript API
==============

<!--{TOC}-->

# require('marko')

## 方法

### load(templatePath[, templateSrc][, options]) : template

为指定路径下的模版加载一个模版实例。
`templateSrc`和`options都是可选的。
Loads a template instance for the given template path.

浏览器端和服务器端都支持模版加载，但是有稍许的不同。

**在服务器端，**
如果没有提供`templateSrc`的话，`templatePath`就会成为Marko模版文件的路径。如果提供了`templateSrc`，那么它就会成为一个	`字符串`，它的值会成为原始的模版。如果提供了`templateSrc`，参数`templatePath`只会用来报告错误信息。


**在浏览器端**
`templatePath`会成为指向编译好模版的模块名的路径，这个模块会通过`require(templatePath)`加载。如果这个`加载`的函数在浏览器中引用，那么`templateSrc`会被忽略。

如果提供了`options`参数，那么最后一个参数应该是一个`对象`。

浏览器和服务器的使用样例：

```javascript
var templatePath = require.resolve('./template.marko');
var template = require('marko').load(templatePath);
template.render({ name: 'Frank' }, process.stdout);
```

在**浏览器端**模版中用`writeToDisk: false`加载的样例：

```javascript
var templatePath = './sample.marko';
var template = require('marko').load(templatePath, {writeToDisk: false});
template.render({ name: 'Frank' }, process.stdout);
```

在**服务器端**模版中用字符串编译的样例：

```javascript
var templatePath = 'sample.marko';
var templateSrc = '<div>Hello $!{data.name}</div>';
var template = require('marko').load(templatePath, templateSrc);
template.render({ name: 'Frank' }, process.stdout);
```

支持的`options`:

- `buffer` (`Boolean`) － 如果值为`true`（默认），那么渲染的输出直到`out.flush()`被调用才会被缓冲，或者到渲染已经结束后。否则，当输出一生成的时候就会被写到底层流中。
- `writeToDisk` (`Boolean`) － 这个可选项只应用在服务器端的模版加载中。如果值为`true`，编译好的模版会被写到瓷盘中。如果值为`false`，模版会被编译和加载，但是编译好的源文件不会被写到瓷盘中。


### createWriter([stream]) : AsyncWriter

创建一个[AsyncWriter](https://github.com/marko-js/async-writer)的实例，这个实例可被用来支持异步渲染。

使用方法:

```javascript
var out = require('async-writer').create(process.stdout);
out.write('Hello');

var asyncOut = out.beginAsync();
setTimeout(function() {
	asyncOut.write('World')
	asyncOut.end();
}, 100);

require('./template.marko').render({}, out);
```

## 属性值

### helpers

所有的模版都是通过全局的helpers。它会被当作`__helpers`变量应用在编译的模版中。不推荐使用这个全局helpers属性值。
Global helpers passed to all templates. Available in compiled templates as the `__helpers` variable. It is not recommended to use this property to introduce global helpers (globals are evil).


# 模版

## 方法

### renderToString(templateData) : String

同步把一个模版渲染成`字符串`。

_注：如果在同步渲染的时候`out.beginAsync()`被调用，会抛出一个错误。_

使用方法:

```javascript
var template = require('./template.marko');
var html = template.renderToString({ name: 'Frank' });
console.log(html);
```

### renderToString(templateData, callback)

异步渲染一个模版并且提供一个返回方法的输出。

```javascript
var template = require('./template.marko');
template.renderToString({ name: 'Frank' }, function(err, html, out) {
	if (err) {
		// Handle the error...
	}

	console.log(html);
});
```

### render(templateData, stream.Writable)

渲染一个模版到可写流中。

使用方法：

```javascript
var template = require('./template.marko');
template.render({ name: 'Frank' }, process.stdout);
```

### render(templateData, AsyncWriter)

渲染一个模版到[AsyncWriter](https://github.com/marko-js/async-writer)实例中，这个实例包裹了一个底层流。当渲染到一个AsyncWriter时，这个写入器不会被自动关闭。你必须亲自调用`out.end()`方法。

使用方法：

```javascript
var template = require('./template.marko');
var out = require('marko').createWriter(process.stdout);
template.render({ name: 'Frank' }, out);
out.end();
```

_注：这个`out`参数将会很少被用到，但是它是[AsyncWriter](https://github.com/marko-js/async-writer)实例的应用，它被用来加快模版的渲染。_

### stream(templateData) : stream.Readable

返回一个可读的流，这个流会被用来读取渲染模版是输出的内容。

使用方法：

```javascript
var template = require('./template.marko');
template.stream({ name: 'Frank' }).pipe(process.stdout);
```

### renderSync(templateData) : String
> v3中不再使用, 用 `renderToString(templateData)` 作为替代.

### render(templateData, callback)
> Dv3中不再使用, 用 `renderToString(templateData, callback)` 作为替代.

# require('marko/compiler')

## 方法

### createCompiler(path, options) : TemplateCompiler

新建一个用于指定模版路径和制定选项的编译器

### compile(src, path, options, callback)

编译一个模版，这个模版指定了加载好的模版源、源模版的系统文件路径和选项。
目前，编译都是同步，所以callback时可选的。将来，我们也许会允许异步渲染。

输出的结果是编译好的JavaScript源代码。

### compileFile(path, options, callback)

编译一个模版，这个模版指定了加载好的模版源、源模版的系统文件路径和选项。
目前，编译都是同步，所以callback时可选的。将来，我们也许会允许异步渲染。

输出的结果是编译好的JavaScript源代码。

### getLastModified(path, options, callback)

编译一个模版，这个模版指定了加载好的模版源、源模版的系统文件路径和选项。
目前，编译都是同步，所以callback时可选的。将来，我们也许会允许异步渲染。

返回类似数字的最后更改时间。

### clearCaches()

清除编译器使用的任何内部缓存。需要热加载。

## 属性值

### 默认选项

这些默认选项用给编译器用的。这些选项可以像下面的演示代码一样被更改：

```javascript
require('marko/compiler').defaultOptions.writeToDisk = false;
```

默认选项：

```javascript
{
    /**
     * 如果为true，编译器会检查磁盘，看是否有一个之前编译好的
     * 模版是同时生成或者更早。如果是这样，这个之前的编译的模版
     * 会被加载。否则，该模版会被编译并保存到磁盘中。  
     *
     * 如果为false，这个模版会永远被编译。如果 `writeToDisk` 
     * 为false，这个选项会被忽略。
     */
    checkUpToDate: true,
    /**
     * 如果为true（默认），那么编译好的模版会被重写到磁盘。
     * 如果为false，编译好的模版不会被写到磁盘（比如，不会生成
     * `.marko.js` 文件）
     */
    writeToDisk: true,

    /**
     * 如果为true，那么当磁盘上存在编译好的模版，它会被认为最新的。
     */
    assumeUpToDate: NODE_ENV == null ? false : (NODE_ENV !== 'development' && NODE_ENV !== 'dev')
};
```

# require('marko/defineRenderer')

用于绑定UI组件的默认模块，它同时带有a `renderer(input, out)` (为了用来作为Marko自定义标签的渲染器) a `render(input)` method (用来渲染UI组件和插入HTML到DOM中)两个方法：

_src/components/app-hello/index.js_

```javascript
var defineRenderer = require('marko/defineRenderer');

module.exports = defineRenderer({
	template: require('./template.marko'),
	getTemplateData: function(input) {
		var firstName = input.firstName;
		var lastName = input.lastName;

		return {
			fullName: firstName + ' ' + lastName
		};
	}
})
```

UI组件可以被当作自定义标签使用：

```xml
<app-hello first-name="John" last-name="Doe" />
```

并且它也可以呗渲染并插入到DOM中：

```javascript
require('./src/components/app-hello')
	.render({
		firstName: 'John',
		lastName: 'Doe'
	})
	.appendTo(document.body);
```

这个 `render()` 的返回值是一个[RenderResult](https://github.com/raptorjs/raptor-renderer#renderresult)的实例。
