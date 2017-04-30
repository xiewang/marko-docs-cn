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

## Using tags from npm

Using custom tags from `npm` is easy.  Ensure that the package is installed and listed in your `package.json` dependencies:

```
npm install --save some-third-party-package
```

And that's it.  Marko will now discover these tags when compiling your templates and you can simply use them in your templates:

```xml
<div>
    <some-third-party-tag/>
</div>
```

## Advanced details

Given a template file, the `marko` module will automatically discover all taglibs by searching relative to the template file. The taglib discoverer will automatically import all taglibs associated with packages found as dependencies in the containing package's root `package.json` file.

As an example, given a template at path `/my-project/src/pages/login/template.marko` and given a `/my-project/package.json` similar to the following:

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

The search path will be the following:

1. `/my-project/src/pages/login/marko.json`
2. `/my-project/src/pages/marko.json`
3. `/my-project/src/marko.json`
4. `/my-project/marko.json`
5. `/my-project/node_modules/foo/marko.json`
6. `/my-project/node_modules/bar/marko.json`

### Hiding taglibs

If you wish to hide particular folder and/or node_module from discovery of marko.json, you can exclude certain directories or packages.  This is used primarily for testing.

```javascript
    require('marko/compiler').taglibFinder.excludeDir(dirPath);
    // Where 'dirPath' is an absolute path to the folder containing marko.json

    require('marko/compiler').taglibFinder.excludePackage(packageName);
    // Where 'packageName' is the name of the node_module containing marko.json
```

These statements should be used before any rendering begins in the process.


### marko.json syntax

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

Marko also supports a short-hand for declaring tags and attributes. The following `marko.json` is equivalent to the `marko.json` above:

```json
{
    "<my-hello>": {
        "renderer": "./hello-renderer",
        "@name": "string"
    }
}
```

### Defining Tags

Tags can be defined by adding `"<tag_name>": <tag_def>` properties to your `marko.json`:

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

Every tag should be associated with a renderer or a template. When a custom tag is used in a template, the renderer (or template) will be invoked at render time to produce the HTML/output. If a `String` path to a `marko-tag.json` for a custom tag then the target `marko-tag.json` is loaded to define the tag.

### Defining Attributes

If you provide attributes then the Marko compiler will do validation to make sure only the supported attributes are provided. A wildcard attribute (`"@*"`) allows any attribute to be passed in. Below are sample attribute definitions:

_Multiple attributes:_

```javascript
{
    "@message": "string",     // String
    "@my-data": "expression", // JavaScript expression
    "@*": "string"            // Everything else will be added to a special "*" property
}
```


### Custom directory scanning

You can configure the `tags-dir` value in your `marko.json` to configure the name of the directory that marko scans in for custom tags.  As described above, by default it uses the name `components/`.  You can override this at a directory level and give a path to another directory to scan:

```json
{
    "tags-dir": "./ui-components"
}
```

`tags-dir` also accepts an array if you have taglibs organized in multiple folders.

```json
{
    "tags-dir": ["./ui-components", "./components"]
}
```

## Exporting tags

To make tags from your project available to other projects, define the public tags in a `marko.json` at the root of your project.