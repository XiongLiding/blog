---
title: base 标签，所有链接在新页面中打开
categories:
  - 时光机
  - 页面设计
tags:
  - HTML
date: 2010-05-29 22:26:43
---

实用的html标签`<base>`

```html
<base href="..." target="...">
```

能定义超链接等的基本行为，有人喜欢所以链接都在新页面中打开，让我改……差点冲动去一个个改，结果发现了这个标签，只要

```html
<base target="_blank">
```

那么所有没有特别指定 `target` 属性的超链接都会在新页面中打开。

这个标签能做的不止这个，`href` 属性可以定义默认的路径，如果指定此属性则页面中所有相对路径(对图片路径也有效)都会以此为参考。

更多细节请参考：http://www.w3schools.com/tags/tag_base.asp
