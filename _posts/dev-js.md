---
title: dev.js
date: 2018-02-13 13:01:04
categories:
  - DevOps
tags:
  - riot
  - prettier
  - eslint
  - gulp
  - webpack
  - npm scripts
---

写完这篇关于 [npm scripts](/2018/02/02/npm-scripts/) 的博客后，我又陆续花了两个半天，将这个定制脚本做到了能让自己满意的程度。功能和性能全面超越了原来的 gulp 脚本，投入的时间也很合理，因为之前无论 gulp 还是 webpack ，都至少花掉我一天时间用在配置上，却达不到理想的效果。

从功能上看，现在的脚本能监视目录、优化代码（主要痛点，市面上没有现成的方案）、检查代码、编译 riot 标签、打包文件，并为自动重载做好了准备。从性能上看，由于充分利用了内存来加速，每次文件变更后触发的流程（优化、检查、编译和打包）提速在十倍以上，从原来的平均超过 1000 ms 到现在的平均不到 100 ms。只是启动仍然需要 2 秒左右，但也比原来稍胜一筹，加上优化难度大、提升空间有限，且一般而言一天也就执行一次，就不强求了。

<!-- more -->

当前版本的完整代码如下：

```JavaScript
// 依赖引入
const fs = require('fs');
const path = require('path');
const http = require('http');
const crypto = require('crypto');

const prettier = require('prettier');
const Linter = require('eslint').Linter;
const eslintrc = require('./.eslintrc.json');
const chokidar = require('chokidar');
const riot = require('riot');

const cacheTags = {};

// 各关键函数
// - 拆分 Pug 文件
const split = (tag) => {
    var pug, script, style;
    [pug, script] = tag.split(/\n    script.*\.\n/);
    [pug, style] = pug.split(/\n    style.*\.\n/);
    if (style) {
        return [pug, script, style]
    }
    [script, style] = script.split(/\n    style.*\.\n/);
    return [pug, script || '', style || ''];
};

// - prettier 处理 
const pretty = (pug, script, style) => {
    script = prettier.format(script, {tabWidth: 4, singleQuote: true, trailingComma: 'all', arrowParens: 'always'});
    style = prettier.format(style, {parser: 'css', tabWidth: 4});
    return [pug, script, style];
};

// - eslint 检查
const lint = (pug, script, filename) => {
    const line = pug.split('\n').length;
    const linter = new Linter();
    console.log(linter.verify(script, eslintrc, {filename: filename}));
};

// - 重新组装
const join = (pug, script, style) => {
    script = script
        ? '\n    script.' + ('\n' + script.trim()).replace(/\n/g, '\n        ') + '\n'
        : '';
    script = script.replace(/\n\s+\n/g, '\n\n');
    style = style
        ? '\n    style.' + ('\n' + style.trim()).replace(/\n/g, '\n        ') + '\n'
        : '';
    style = style.replace(/\n\s+\n/g, '\n\n');
    return pug + script + style;
};

// 编译 Riot 标签
const compile = (tag) => {
    return riot.compile(tag, {
        template: 'pug',
        type: 'es6',
        style: 'css',
    });
};

// 前缀时间，用于输出
const log = (str) => {
    const time = new Date().toString().match(/\d+:\d+:\d+/)[0];
    console.log(`[${time}] ${str}`);
};

// 单次处理的完整流程
const job = (tag, filename) => {
    var pug, script, style;
    [pug, script, style] = split(tag);
    [pug, script, style] = pretty(pug, script, style);
    lint(pug, script, filename);
    const newTag = join(pug, script, style);
    cacheTags[filename] = compile(newTag);
    return newTag;
};

// 处理工序
const watch = (subapp) => {
    const cwd = path.join(__dirname, 'riot-tags', subapp);
    const cache = {};
    chokidar
        .watch('*.tag', {cwd: cwd})
        .on('add', (filename) => {
            const fullname = path.join(cwd, filename);
            fs.readFile(fullname, {encoding: 'utf8'}, (err, tag) => {
                cacheTags[filename] = compile(tag);
            });
        }).on('change', (filename) => {
            const fullname = path.join(cwd, filename);
            fs.readFile(fullname, {encoding: 'utf8'}, (err, tag) => {
                if (cache[fullname] == tag) return;
                try {
                    const result = job(tag, filename);
                    if (result == tag) return;
                    log('文件已更新，等待重新加载')
                    cache[fullname] = result;
                    fs.writeFile(fullname, result, {encoding: 'utf8'});
                } catch (e) {
                    log('语法错误');
                    console.log(e.codeFrame);
                    return;
                }
            });
        });
    log('程序已启动');
};

watch(process.argv[2]);

http.createServer((req, res) => {
    const tags = Object.values(cacheTags).join('\n');
    res.setHeader('Content-Type', 'text/javascript');
    res.setHeader('Accept-Charset', 'utf-8');
    const hash = crypto.createHash('md5').update(tags).digest("hex");
    res.setHeader('ETag', hash);
    res.end(tags);
}).listen(process.env.PORT);
```

代码总计 121 行，比我预想的要简单多了。

## 代码分析

由于内容比较多，只能针对部分重点细讲，其他就一笔带过，读者可以自行阅读源码。

1-11 行，引入依赖。

### 功能函数

这批函数，各自完成一个独立功能。

17-26 行，split 函数，通过正则表达式拆解 riot 标签，将其分解成 pug、JavaScript、CSS。

29-33 行，pretty 函数，使用 Prettier 格式化代码。

36-40 行，lint 函数，使用 Eslint 对 Prettier 格式化后的代码，进行进一步的检查。检查结果中的行号不是根据原文件而是根据 JavaScript 片段来的，还有改进空间。

43-53 行，join 函数，将 pug、JavaScript、CSS 重新拼接回 riot 标签。

56-62 行，compile 函数，编译 riot 标签。

65-68 行，log 函数，在输出的内容前加上当前时间。

### 串联

71-79 行，job 函数，串联上述功能，并将 compile 函数的结果缓存到 cacheTags 对象中用于后续打包。其中调用 lint 的步骤，不会对后面的 compile 产生影响，也不影响 job 函数的返回值，因此如果能改成开一个子进程来执行，就可以更有效地利用 CPU 来提高效率。

### 监视、初始化与渐进

82-110 行，watch 函数，完成了初始化并通过监视文件变化渐进式的改进结果。

这里使用 chokidar 来监视指定目录，on('add', ...) 在开启监视时被触发，针对每个被监视的文件执行一次，因此可以读取所有 riot 标签文件，并将编译结果保存到 cacheTagss对象中，on('change', ...) 在文件发生变化时触发，每当用户保存文件时，针对该文件调用 job 函数将分解、优化、检查、拼接、编译的流程走上一遍。优化的结果会写回原文件，用户在编辑器中重新加载该文件（可以通过配置编辑器实现自动重载），就会看到优化后的代码。编译的结果也会保存在 cacheTags 变量中。

由于回写文件会再次触发 change 事件，这里用 cache 变量保存了每个文件上次优化后的内容，通过比较文件内容来避免链式触发。

112 行，用命令行中指定的参数作为目录名，调用 watch 函数，因此，在命令行执行：

```bash
node dev.js app
```

即可调用此脚本监视项目目录下的 riot-tags/app 目录。

### 文件打包

原来的 gulp 版本会将最终结果打包成一个物理文件，作为静态文件供网页调用。而我在学习 webpack 的时候发现它能开启一个 web 服务来提供打包结果，以内存计算代替文件读写，相比 gulp 要高效得多，因此我采用了这种方案。

114-121 使用了 node 原生的 http 来启动一个 web 服务器，无论你向它发送什么请求，都会将打包结果返回给你。

这里首先通过 Object.values 将 cacheTags 中的内容全部取出来，再合并成一个整体。Object.values 函数可能比较新，node 6.x 中不能使用，因此我用了 8.x 来执行这个脚本。当然这个功能也可以通过循环实现。

后面设置了几个 http header 来确保浏览器能正确的识别返回内容。特别是 ETag 的设置，这里将打包结果的 md5 码作为 ETag，保证每次文件变化时，ETag 都能相应变化，这样就可以通过 [live.js](http://livejs.com/) 快速实现代码修改后，浏览器自动刷新的功能了。

最后 listen 中的参数，参考了 express 设置端口的方法。

### Npm Scripts
最后，在 package.json 中，添加下列内容：

```
"scripts": {
    "dev-app": "PORT=16664 node dev.js app"
},
```

就可以通过 `npm run dev-app` 调用上述脚本，其中目录参数是 app，端口参数是 16664。

## 总结
目前看来，选择通过自制脚本来解决这个问题无疑是正确的。开发成本只是略高于 gulp 或 webpack 的学习和配置成本，但是灵活性要好得多，对于特殊需求，不再需要技巧性 hack，而是直接从根源解决，得到一个更满意的结果。面对飞速发展的 web 前端和一个又一个新冒出来的工具，也能更坦然了。

另外，实现上述工作的代码量远小于我的预期，最初畏惧调用原生库转而寻求各种 gulp 和 webpack 插件是完全没有必要的。很多看似高大上的东西，花心思去了解琢磨了，也没之前想象的那么复杂。

世上无难事，只怕有心人。
