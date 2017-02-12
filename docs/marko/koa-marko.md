Koa + Marko
=====================

# 安装

```
npm install koa --save
npm install marko --save
```

# 用法

```javascript
require('marko/node-require').install();

var koa = require('koa');

var app = koa();

app.use(function *() {
    this.body = template.stream({
            name: 'Frank',
            count: 30,
            colors: ['red', 'green', 'blue']
        });
});

app.listen(8080);
```