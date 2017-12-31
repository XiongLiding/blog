---
title: input button submit
categories:
  - 时光机
tags:
  - HTML
date: 2012-04-19 12:32:24
updated: 2017-12-31 22:11
---



以前做表单的按钮使用这两个元素
普通按钮：`<input type="button">`
提交按钮：`<input type="submit">`

后来做表单的按钮使用这两个元素
普通按钮：`<button>`
提交按钮：`<button type="submit">`

今天发现 
`<button>` 和 `<button type="submit">` 是一样的，都是提交按钮，囧

和 `<input type="button">` 对应的：
不是 `<button>` 而是 `<button type="button">`
也就是说 `<button>` 元素的 type 默认是 submit 不是 button

-- 2017-12-31 --

事实上，不同浏览器的对 `<button>` 的默认行为有不同的定义，你应该总是显式的声明 `type`，又一个例子，证明写清楚点总是好的，不要去记浏览器们的差别在哪里。
