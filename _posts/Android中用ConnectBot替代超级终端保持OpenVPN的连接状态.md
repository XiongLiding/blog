---
title: Android中用ConnectBot替代超级终端保持OpenVPN的连接状态
date: 2017-02-10
categories:
  - 时光机
  - Android
tags:
  - Android
  - OpenVPN
---

我之所以要用ConnectBot来替代系统自带的超级终端，还是和OpenVPN有关：我是用命令行连接OpenVPN的（找一个好的GUI实在不容易，TunnelDroid算是一个，可惜灵活性不够），用命令行连接时遇到的最大麻烦就是Android的内存回收机制——Android在内存不足时会优先停止在后台运行的程序，用来连接OpenVPN的超级终端很容易被关掉，虽然OpenVPN连接不会因此断开，但是要查看OpenVPN的状态或者做一些后续操作就变得复杂了。所以我需要一个不那么容易被关掉的终端……

我最初想到的是带通知功能的超级终端，后来意外发现自己已经安装的ConnectBot正是这么个东西——ConnectBot一般被用作连接远程服务器的SSH客户端，但也能通过连接local来做为一般的命令行终端使用，更重要的是它提供了通知栏功能，让程序在后台运行时也能保持连接。

东西找到了，接下来的操作就比较简单了。

1. 打开ConnectBot
2. 在下拉菜单的ssh，telnet，local中选择local
3. 文本框中提示输入Nickname，会做为这个连接的名称，方便下次再用
4. 按回车连接

后面的操作就和在系统自带的超级终端里一样了。第一次使用ConnectBot的话可能要适应下它的按键，特别是组合键，和超级终端不太一样。

p.s. ConnectBot可以在官方的Market里下载。
