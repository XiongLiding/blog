---
title: Org-mode 导出 html 模版设置
date: 2019-01-16 10:17:17
tags: Org-mode
---

用 Org-mode 写了一篇接口说明，要发给合作单位，所以得先导出成一个通用的文档格式。内置的几种格式看下来，只有 html 的模版最好看，但是内容上与设想的有所出入，主要在于以下几点：
    - 没有标题
    - 目录的标题是英文
    - 底部的签名是个人的

<!--more-->

# 解法
## 没有标题
在文档头部添加 `#+TITLE: 文章标题`，文章标题会出现在 Table Of Contents 之前，居中显示。

## 目录的标题是英文
在文档头部设置语言 `#+LANGUAGE: zh-CN`，“Table of Contents”会变成中文的“目录”。

## 底部的签名
默认情况下，底部会有文档创建时间、作者和w3c校验链接，通过设置 `#+OPTIONS: html-postamble:nil` 加以隐藏

# 其他
Org-mode 默认没有导出 PDF 的选项，需要配合 Latex 才能使用，一个比较简单的方法就是导出成 html ，然后用浏览器的打印功能保存成 PDF。
Org-mode 不能直接导出 Word ，插件的效果也不是很理想，ODT 可以用 Word 查看，但是格式并不是很好看，导出 html 再全文粘贴到 Word 会稍微好一点，可惜代码块内容的行距明显过大。

# 参考
[Export settings 导出设置的选项](https://orgmode.org/manual/Export-settings.html)
[ISO Language Code Table 语言代号](http://www.lingoes.net/en/translator/langcode.htm)
