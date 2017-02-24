Marko布局标签库
============

该模块为Marko提供了内置的 `layout` 标签库。`layout` 标签库提供从内容中分离出THML布局的支持。布局文件只是一个有占位符的普通Marko模版，这些占位符是允许提供额外的内容给另一个模版用。

# 样例

`layout` 标签库的使用方法如下样例所示：

_default-layout.marko:_

```xml
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8"/>
        <title>
            <layout-placeholder name="title"/>
        </title>
    </head>
    <body>
        <h1 if(data.showHeader !== false)>
            <layout-placeholder name="title"/>
        </h1>
        <p>
            <layout-placeholder name="body"/>
        </p>
        <div>
            <layout-placeholder name="footer">
                Default Footer
            </layout-placeholder>
        </div>
    </body>
</html>
```

_`default-layout.marko`的用法:_

```xml
<layout-use("./default-layout.marko") show-header=true>
    <layout-put into="title">My Page</layout-put>
    <layout-put into="body">BODY CONTENT</layout-put>
</layout-use>
```

# 定义布局

布局只是一个有着特殊 `<layout-placeholder>` 的标准Marko模版。

## `<layout-placeholder>`

支持的属性：

- **name** - 用来分配占位符的别名（必须的）。
- 每个 `<layout-placeholder>` 标签应该用 `name` 属性来分配一个name。

每个占位符应该拥有一些默认的内容，如果调用方没有需要呈现的内容传过来，默认的内容就会显示出来。默认的内容（若果有的话）应该像下面嵌套的内容一样：

```xml
<layout-placeholder name="footer">
    This is the default content for the "footer" placeholder. If
    no "footer" content is provided by the caller then this content will
    be rendered.
</layout-placeholder>
```

布局模版的使用者用 `<layout-put>` 标签给占位符提供内容（稍后详述）。

# 使用布局

## `<layout-use>`

`<layout-use(template[, templateData])>` 标签被用来给布局模版渲染内容，这些内容由调用者提供。

支持的属性：

- **template** - 布局布局模版的路径或者用于解析加载好的模版实例的JavaScript表达式。
- 任何其他的属性都可以用来提过额外的数据给布局模版（稍后详述）

样例：

```xml
<layout-use("./default-layout.marko") show-header=true>
    ...
</layout-use
```

## `<layout-put>`

`<layout-put>` 标签用来提供布局内容。

支持的属性：

- **into** (必须的) － 需要被替换的占位符的别名
- **value** (可选的) － 应该使用的布局内容。如果没有提供相应的布局内容给对应的占位符，那么就应该用嵌套内容作为替代。

如果提供了嵌套内容，那么它会被用来替换相应占位符内容。

使用方法：

```xml
<layout-put into="title">My Page</layout-put>
```

同样的，可以用如下的 `value` 属性来提供内容：

```xml
<layout-put into="title" value="My Page"/>
```

# 布局数据

额外的数据可以被调用者来提供给布局模版。数据独立于内容，可以用来控制如何渲染布局。布局数据会是在标准 `data` 变量里的可用的值。除了"template"属性，任何额外的属性都被用来提供给 `<layout-use>` 标签使用，用来传递额外的数据给布局模版。

下面是一个 `showHeader` 属性的例子，它可以传递给布局模版，用来控制如何渲染头部内容：

_default-layout.marko:_

```xml
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8"/>
        <title>
            <layout-placeholder name="title"/>
        </title>
    </head>
    <body>
        <h1 if(data.showHeader !== false)>
            <layout-placeholder name="title"/>
        </h1>
        ...
    </body>
</html>
```

_`default-layout.marko`的用法:_

```xml
<layout-use("./default-layout.marko") show-header=true>
    ...
</layout-use>
```

_NOTE: All data attributes will be de-hyphenated and converted to camel case (e.g., `show-header` will be accessible via the `showHeader` property on the `data` object)._
