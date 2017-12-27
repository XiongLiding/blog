---
title: 归来的技术栈——正确模块化，express 的 app.use
tags:
  - 归来的技术栈
date: 2017-10-17 00:01:41
---

在 express 4 中，app.use 有如下用法：

```javascript
const express = require('express');
const app = express();
const subapp = express();

app.use('/subpath', subapp);

app.listen(3000);
```

subapp 做为 express 的一个实例，本身也是 middleware ，可以被 app.use “挂载”到指定路径。

这种用法给我们项目中功能模块的可移植性进一步增加了保证，以之前用到的目录结构为例：

```
.
├── app.js
└── src
     ├── admin
     │    ├── stylesheets
     │    ├── javascripts
     │    ├── index.html
     │    ├── client.js
     │    ├── riot-tags
     │    └── server.js
     └── app
          ├── stylesheets
          ├── javascripts
          ├── index.html
          ├── client.js
          ├── riot-tags
          └── server.js
```

两个 server.js 文件分别是 admin 和 app 的入口文件。这时我们可以这样做：

/app.js

```javascript
const express = require('express');
const app = express();

const admin = require('./src/admin/server.js');

app->use('/admin', admin);

app->listen(3000);
```

/src/admin/server.js

```javascript
const express = require('express');
const app = express();

app->get('/login', ...)
app->post('/api/login', ...)
...

module.exports = app;
```

这样，我们可以通过`GET /admin/login` 和 `POST /admin/api/login` 来访问 admin 模块中的方法了。 在这种设计下，admin 目录中的内容自成一体，不依赖 /app.js 向它传入参数。可移植性就更好了。

由于 /src/admin/server.js 中的 app 是一个完整的 express 实例，因此它本身也可以通过 use 方法来使用 middleware。并且这个 middleware 的有效范围也会被控制在 admin 中，而不会干扰其他模块。所以这也是一种控制 middleware 有效范围的简便方法。

这个方法将模块和主程序完全解偶，如果你再进一步，将 admin 模块做成无状态的，那么就可以安全的实现热更新了，这将大大提高开发效率。
