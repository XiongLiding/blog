---
title: 简单粗暴也是一种美——用 Prettier 处理 Pug 文件
date: 2018-02-11 09:32:17
categories:
  - DevOps
tags:
  - Prettier
  - Pug
---

![Prettier](http://p1kp5xfj8.bkt.clouddn.com/2018-2-11-上午11:57:28.png)

[Prettier](https://prettier.io) 是一个新兴的代码格式化程序，但目前支持的语言相对有限，特别对于混合文件的处理仍在比较初级的阶段。

因此，针对 Riot 这种 HTML/Pug 中混合了 JavaScript 和 CSS 的文件，要对其中的代码进行格式化，必须先将文件分解，分别用 Prettier 处理后，再重新合并。

相比 HTML，处理 Pug 这种由缩进决定包含关系的文件倒是更简单，很容易通过一些简单粗暴的方法进行分解。

Prettier 本身也是一种广受欢迎的粗暴解决方案。它的主要动机就是通过将一种固定格式强加给代码，消灭不同风格间的优劣之辫，节约程序员在争辩中浪费的时间，集中精力办实事。因为大多数时候，主流风格间并没有明显的好坏，只是个人喜好不同。

<!-- more -->

## 文件分解

```javascript
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
```

最初的版本是将文件切分成行后，找出 script 和 style 所在的行，然后进行分割。相对现在的版本，理论上更容易理解，但由于实际代码量大很多，在没有注解的情况下反而没个这个版本思路清晰了。

由于约定了缩进是 4 个空格，并要求显式包含 script 标签，riot 文件基础格式相对固定。

```
tagname
    // 标签

    script.
        // 脚本
    style.
        // 样式
```
另有几个变种，比如忽略 script 部分、忽略 style 部分、script 和 style 上下互换等，用上面这个函数都能处理，同时里面用到的正则表达式也能处理 `script.` 或 `script(type="text/es6").` 等指定或不指定类型的情况。

## 格式化

```javascript
[pug, script, style] = split(tag);
script = prettier.format(script, {tabWidth: 4, singleQuote: true, trailingComma: 'all', arrowParens: 'always'});
style = prettier.format(style, {parser: 'css', tabWidth: 4});
```

Prettier 具备对 JavaScript 文件和 CSS 文件的格式化能力，并允许少量参数调整，此处只要根据官方接口文档提供的信息，按照自己的项目需要进行配置即可。最主要是明确按照什么文件类型进行处理，并指定缩进宽度。

## 合并

```javascript
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

const result = join(pug, script, style);
```

在合并的时候，我也采用了粗暴的方案，无论原来代码中脚本和样式哪个在前，我总是将脚本放在前面，并且统一丢弃了类型说明。在拼接前，先将每行代码缩进 8 个空格，再去除对空行的缩进和多余的换行。

## 简单的性能测试和后续工作

格式化执行的频率越高，每次代码的变化就越少，也就越不容易影响编码者的思路。为了能使格式化的频率提高，单次的开销就必须尽可能降低，否则会影响编码者的体验。

Node 中有一个简单的性能测试方法，就是使用 `console.time` 和 `console.timeEnd`，这两个方法能方便的计算出两次调用间耗费的时间。

经过简单测试后，发现即便是固态硬盘，加载文件（主要是 node_module 中的文件）的时间仍是明显大于实际的格式化处理，可见将 watch 内置到脚本中是非常必要的，使格式化服务保持在线，相比“冷启动”可以起到明显的提速效果。

## 相关文章
[npm script](https://bnlt.org/2018/02/02/npm-scripts/)
