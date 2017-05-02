# 自定义标签

Marko 可以使用[核心标签](./core-tags.md)一样的API，这样的话你就可以用自定义标签和属性来扩展语言。

> **提示：** 我建议自定义标签和属性至少使用一个破折号，来表明它不是标准的HTHML语法的一部分。

## 写自定义标签

开始的时候，让我们来先看看基于模板的标签，它能让你通过一个指定的自动移标签引用另一个模版，而不是指定系统文件路径并使用 `<include>`。

### 搜索标签

当编译一个模版的时候，Marko会在模版的路径开始搜索，向上到目录名为 `components/` 的项目根目录。然后尝试加载这些目录的子目录作为自定义标签。这些子目录可能是Marko模版或者一个有 `index.marko` 模版的目录（和其他支持的文件）。

```dir
components/
    app-header/
        index.marko
        logo.png
        style.css
    app-footer.marko
pages/
    home/
        components/
            home-banner.marko
        index.marko
```

当在 `pages/home/index.marko` 这里编译模版的时候，会找到下面的标签：

- `<app-header>`
- `<app-footer>`
- `<home-banner>`

所以现在你不需要指定一个路径：

```xml
<include('../../components/app-header/index.marko')/>
```

只需要使用标签名：

```xml
<app-header/>
```

## 从 npm 中使用标签

通过  `npm` 使用自定义标签。确保包已安装并在列在你的 `package.json` 依赖里：

```
npm install --save some-third-party-package
```

然后就这样，Marko 编译你的模版时会搜索这些标签，你可以很简单的在你的模版中使用：

```xml
<div>
    <some-third-party-tag/>
</div>
```

## 更多说明

给定的模版文件，`marko` 模块会通过搜索相关的模版文件自动寻找所有的标签库。标签‘搜索器’会自动引入所有在容器包中的根文件 `package.json` 中的作为依赖的包。

举个例子，给定一个 `/my-project/src/pages/login/template.marko` 路径上的模版和一个类似下面的 `/my-project/package.json` 文件：

```json
{
    "name": "my-package",
    "dependencies": {
        "foo": "1.0.0"
    },
    "devDependencies": {
        "bar": "1.0.0"
    }
}
```

搜索路径会如下所示：

1. `/my-project/src/pages/login/marko.json`
2. `/my-project/src/pages/marko.json`
3. `/my-project/src/marko.json`
4. `/my-project/marko.json`
5. `/my-project/node_modules/foo/marko.json`
6. `/my-project/node_modules/bar/marko.json`

### 隐藏标签库

如果你希望从 marko.json 的搜索域中隐藏特定文件夹和（或者）node_module，你可以排除特定的目录或者包。这个主要会在测试中使用。

```javascript
    require('marko/compiler').taglibFinder.excludeDir(dirPath);
    // Where 'dirPath' is an absolute path to the folder containing marko.json

    require('marko/compiler').taglibFinder.excludePackage(packageName);
    // Where 'packageName' is the name of the node_module containing marko.json
```

这些声明应该在渲染进程开始前使用。


### marko.json 语法

```json
{
    "tags": {
        "my-hello": {
            "renderer": "./hello-renderer",
            "attributes": {
                "name": "string"
            }
        }
    }
}
```

Marko 同样支持声明标签和属性的速记写法。下面的 `marko.json` 和上面 `marko.json` 的相同：

```json
{
    "<my-hello>": {
        "renderer": "./hello-renderer",
        "@name": "string"
    }
}
```

### 定义标签

标签可以通过在你的 `marko.json` 中添加 `"<tag_name>": <tag_def>` 来定义：

```json
{
    "<my-hello>": {
        "renderer": "./hello-renderer",
        "@name": "string"
    },
    "<my-foo>": {
        "renderer": "./foo-renderer",
        "@*": "string"
    },
    "<my-bar>": "./path/to/my-bar/marko-tag.json",
    "<my-baz>": {
        "template": "./baz-template.marko"
    },
}
```

每个标签应该和一个渲染器或者一个模版关联。当一个标签在模版中使用，渲染器（或者模版）会
Every tag should be associated with a renderer or a template. When a custom tag is used in a template, the renderer (or template) will be invoked at render time to produce the HTML/output. If a `String` path to a `marko-tag.json` for a custom tag then the target `marko-tag.json` is loaded to define the tag.

### 定义属性

如果你提供属性，那么 Marko 编译器会作验证，确保只提供支持的属性。一个通配符 (`"@*"`) 允许任何属性传递。下面的是属性定义的例子：

_多属性：_

```javascript
{
    "@message": "string",     // String
    "@my-data": "expression", // JavaScript expression
    "@*": "string"            // Everything else will be added to a special "*" property
}
```


### 自定义文件夹扫描

你可以在你的 `marko.json` 中配置 `tags-dir` 值来配置文件夹的名称，marko 会在该文件夹下扫描自定义标签。正如上面的所描述的，默认会使用 `components/`。你可以在文件夹层次下重写该名称，给定另一个文件夹用来扫描：

```json
{
    "tags-dir": "./ui-components"
}
```

如果你有多个目录哟哦你过来组织标签库，`tags-dir` 也支持接受数组。

```json
{
    "tags-dir": ["./ui-components", "./components"]
}
```

## 输出标签

为了让你的工程可以被其他工程使用，在你的工程根目录下 `marko.json` 文件中定义公共标签。