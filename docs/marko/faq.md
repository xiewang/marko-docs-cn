常见问题
==========================

{TOC}

# Marko可以在生产环境中使用吗？

是的，Marko已经在[eBay](http://www.ebay.com/)和其他公司严格测试一年多，它是高性能、可扩展、安全稳定的设计。

# Marko模版可以在浏览器端编译吗？

可以，但是不推荐这么做，并且在比较老的浏览器可能吧不能工作。编译器被优化来生成精短、高性能的编译模版，但是编译器本身并不小，而且它绑定了一些很重的模块，比如[JavaScript HTML parser](https://github.com/philidem/htmljs-parser)。总的来说，在服务器端来编译模版才是推荐的方法。我们推荐使用[Lasso.js](https://github.com/lasso-js/lasso)来引用编译好的模版来作为网页的一部分。

# 支持哪些浏览器?

模版渲染的运行环境支持所有的浏览器。如果你发现了什么问题，请提出bug。

# Marko如何和Express一起使用？

Marko和Express一起使用的推荐方式可参照 [Express + Marko](http://markojs.com/docs/marko/express/) 说明。

# 模版以及局部模版最好放在什么目录下？

你的模版应该同其他JavaScript模块一样组织。你应该将你的模版直接放在引用他们的代码旁边。就是说，你不用新建一个独立的“templates”文件夹。使Marko编写一个简单的Express应用，你可以参考 [marko-express](https://github.com/marko-js-samples/marko-express)。