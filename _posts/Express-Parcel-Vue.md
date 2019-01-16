---
title: 集成 Express、Parcel 和 Vue
date: 2018-11-26 22:01:52
categories:
  - DevOps
tags:
  - node.js
  - express
  - vue
  - parcel
---

最近终于动手整理项目的框架，总体思路和去年基本一致，“无状态”和“按功能模块分目录而不是按前后端”仍然是核心的思想，不过具体的工具发生了变化。
- 数据存取这边用 GraphQL 取代 PouchDB ，在灵活性和可预测间做一个平衡；
- 前端用 Vue 替换 Riot ，主要还是出于生态方面的考虑，虽然复杂度提升了一点，但是可用的第三方库丰富很多；
- 最后用 Parcel 替换 Webpack 和 dev.js，Webpack4 据说简洁了不少，但还是有阴影，Vue 如果自己写 dev.js 也要比 Riot 的难一点，先用 Parcel 偷个懒把。

当然首要目标还是简化日常开发，于是花了两个小时把 Express、Parcel 和 Vue 集成到一起。做到每次启动后端进程的时候，Parcel 也会同时开始工作，不需要执行另外的命令，也不会占用额外的端口。Parcel 如果不单独占用端口，在项目多的时候，配置起来会清爽很多。

<!-- more -->

## 目录与文件

```
.
├── index.js
├── dist
└── src
    ├── admin
    │   ├── index.js
    │   ├── index.html
    │   ├── admin.js
    │   └── components
    └── app
        ├── index.js
        ├── index.html
        ├── app.js
        └── components
```

目录和去年的思路一致，admin 和 app 都是功能模块，且结构相似，每个都包含自己的服务端文件和客户端文件。下面的内容以 admin 为例。

### /index.js （根目录下的）
```JavaScript
const express = require('express');
const app = express();

const admin = require('./src/admin');
app.use('/admin', admin);

app.use(express.static('dist'));

app.get('/', (req, res) => {
  res.end('Hello, World!');
});

app.listen('3000');
```

这里启动了一个 express 实例监听 3000 端口，重点在于第 4-5 行，把 /src/admin/index.js 做为 middleware 挂载到 app 上，原理我在这里讲过[正确模块化，express 的 app.use](https://bnlt.org/2017/10/17/%E5%BD%92%E6%9D%A5%E7%9A%84%E6%8A%80%E6%9C%AF%E6%A0%88%E2%80%94%E2%80%94%E6%AD%A3%E7%A1%AE%E6%A8%A1%E5%9D%97%E5%8C%96%EF%BC%8Cexpress-%E7%9A%84-app-use/)

第 7 行把 dist 做为静态文件目录，因为 Parcel 编译的结果都会放到这个这个目录里来，**没有这一行的话，后面访问 Parcel middleware 提供的地址时，会出现找不到 JavaScript 文件的情况**。

### /src/admin/index.js
```JavaScript
const path = require('path');

const express = require('express');
const app = express();

const Bundler = require('parcel-bundler');

const entry = path.join(__dirname, 'index.html');
const bundler = new Bundler(entry);
app.use('/index', bundler.middleware());

... 其他 admin 模块的接口

module.exports = app;
```

引入 parcel-bundler ，以同目录下的 index.html 为入口进行打包，并以 middleware 的形式集成到 express 实例中，也就是将新的入口文件集成到 /admin/index 这个地址

### /src/admin/index.html
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <div id="app">
      {{ message }}
    </div>
    <div id="app-2">
      <span v-bind:title="message">
        鼠标悬停几秒钟查看此处动态绑定的提示信息！
      </span>
    </div>
    <div id="app-3">
      <span v-if="seen">Now you see me</span>
    </div>
    <div id="app-4">
      <ol>
        <li v-for="todo in todos">
          {{ todo.text }}
        </li>
    </div>
    <div id="app-5">
      <p>{{ message }}</p>
      <button v-on:click="reverseMessage">逆转消息</button>
    </div>
    <div id="app-6">
      <p>{{ message }}</p>
      <input v-model="message">
    </div>
    <div id="app-7">
      <ol>
        <todo-item
          v-for="item in groceryList"
          v-bind:todo="item"
          v-bind:key="item.id">
        </todo-item>
      </ol>
    </div>
  </body>
  <script src="./admin.js"></script>
</html>
```
基于 vue 模版的 html 文件，内容是 vue 官方文档中的几个例子，
倒数第二行的 script 引用了同目录下的 admin.js，Parcel 编译后，这个 html 文件大体上保持原来的样子，只是这一行变成了引用编译后生成的文件名

```html
  <script src="./admin.20f23d0e.js"></script>
```
其中引用的 admin.20f23d0e.js 就是 Parcel 打包后的 JavaScript 文件。具体文件名根据文件内容会有所不同。

### /src/admin/admin.js
```JavaScript
import Vue from 'vue/dist/vue.esm.js';

var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
});
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date().toLocaleString()
  }
});
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
});
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: '学习 JavaScript' },
      { text: '学习 Vue' },
      { text: '整个牛项目' },
    ]
  }
});
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('');
    }
  }
});
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
});
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
});
var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      {id: 0, text: '蔬菜' },
      {id: 1, text: '奶酪' },
      {id: 2, text: '随便其它什么人吃的东西' },
      ]
  }
});
```
这个文件的内容是 vue 官网文档例子的 JavaScript 部分，此文件会被打包成 admin.20f23d0e.js ，需要注意的是第一行引用的是 vue/dist/vue.esm.js 而不是单纯的 vue ，不然会提示 You are using the runtime-only build。Vue 提供了多个版本，这里需要使用包含编译器的版本，具体可以参考官方文档中[对不同文件名的说明](https://cn.vuejs.org/v2/guide/installation.html#%E5%AF%B9%E4%B8%8D%E5%90%8C%E6%9E%84%E5%BB%BA%E7%89%88%E6%9C%AC%E7%9A%84%E8%A7%A3%E9%87%8A)

最后打包完的内容被放在了根目录下的 dist 目录中，而不是和 admin.js 同目录。配合上文提到的 /index.js 中的第 7 行的配置，就可以得到正确的结果了。

使用 node index.js 启动服务，parcel 也会被通过 api 调用的形式拉起来，访问 http://127.0.0.1:3000/admin/index 即可看到打包后的文件的运行效果。
