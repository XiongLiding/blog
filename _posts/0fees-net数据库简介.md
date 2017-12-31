---
title: 0fees.net数据库简介
categories:
  - 时光机
  - 域名、空间
date: 2009-08-14
updated: 2017-12-28 08:43
---

之前写了一篇日志简单介绍了[0fees.net以及注册过程](https://bnlt.org/2009/07/07/0fees-net免费空间/)

今天来简单介绍下0fees的数据库，包括可能在使用过程中遇到的两个问题：

1. 0fees提供了5个数据库，共50M空间
2. 数据库管理界面是phpMyAdmin
3. 默认编码latin1，这会导致一些问题。比如用cpanel控制面板中的Fantastico type installer功能自动安装博客（比如wordpress）时，由于数据库默认编码导致中文乱码。
4. 最后，0fees的数据库不能从非0fees的空间连接，因此只想用这个做免费数据库的可以绕道了。
