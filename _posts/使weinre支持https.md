---
layout: post
title: 使weinre支持https
date: 2018-06-16 10:39:00
tags:
---

在远程调试手机网页的工具中，weinre算是元老级的了，但是作为一个独立工具已经很久没有更新（现在成了Cordova的一个子项目），易用性落后于Chrome内置的调试工具，也比不上微信web开发者工具。
但是纯粹在“远程”这一点上，还是无可替代的，不需要连接USB，不需要在同一个局域网，weinre可以调试任何一台连接到你服务器的手机。

weinre通过将一个JavaScript文件引入目标网页达到远程调试的目的，但是在https加密的网站上，会遇到浏览器
