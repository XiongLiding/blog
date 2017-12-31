---
title: Eclipse 编码
categories:
  - 时光机
  - 快修
date: 2009-07-30
---

在新安装的 eclipse 中编辑已有的项目，而项目原来是在其他系统中做的，那么很容易出现乱码。

主要原因是系统默认的编码不同，而 eclipse 会将系统默认的编码作为自己的默认编码，最终出现乱码。

解决的方法是进入 project-properties-resource，在 text file encoding 下的 other 中选择相应的编码。

其他语言版本可以使用类似的方法解决。
