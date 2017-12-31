---
title: Android G1 OpenVPN 系列教程之──OpenVPN的安装与使用
categories:
  - 时光机
  - Android
tags:
  - Android
  - OpenVPN
date: 2009-11-22 17:50:35
updated: 2017-12-31 18:54
---

<span style="color: red;">*本文配图已遗失*</span>

### **声明**

本文所有内容在安装了HIAPK3.01ROM的G1手机中测试通过，对于其他机型和ROM，本人不保证其完全可行。

<div id="attachment_64" class="wp-caption alignnone" style="width: 330px">![我的手机相关参数](http://bnlt.org/wp-content/uploads/2009/11/CAP200911221108.jpg "环境")

测试机的相关参数
</div>

### **前言**

首先，我是一个有CMWAP包月无限流量卡的G1手机用户。

此前，我有一个诺基亚3230，我通过诺基亚的PC套件使电脑通过手机上网，并且使用OpenVPN成功地突破了移动对CMWAP限制，获得了类似CMNET的网络访问权限──不过我想更多的人用OpenVPN是因为对自由的向往──于是再后来我就将两者结合了。

那么，当我拥有了一个功能强大的并且非常依赖网络的Android手机时……

### **OpenVPN的安装与使用**

#### 参考资料：

[http://android.modaco.com/content/software/291919/openvpn-on-android/](http://android.modaco.com/content/software/291919/openvpn-on-android/)

这个帖子解决了我不少问题，本文的OpenVPN安装方法就是根据该帖2楼的方法并结合自己的实际情况进行调整后得出的，修改后的方法对PC端的要求更低，不需要安装adb等工具。

#### 准备工作：

1.超级终端
2.ROOT权限
3.读卡器或数据线

──我用的是HIAPK3.01ROM，系统自带了超级终端并默认具有ROOT权限，其他机型和ROM可能需要先行安装超级终端并获取ROOT权限。

#### 安装OpenVPN：

那么我们就开始吧，首先下载用于Android的OpenVPN软件包：[http://cloud.github.com/downloads/fries/android-external-openvpn/openvpn-android-2.1.tar.bz2](http://cloud.github.com/downloads/fries/android-external-openvpn/openvpn-android-2.1.tar.bz2)

解压缩，得到openvpn文件夹，结构如下：

```
-openvpn
-system
-bin
    openssl
    openvpn
-lib
    libcrypto.so
    liblzo.so
    libssl.so
```

将system文件夹复制到sd卡根目录下──读卡器和数据线随你喜欢；

将卡放回手机，打开超级终端；

输入下面这行命令将系统挂载为可写：

```bash
mount -o rw,remount -t ext2 /dev/block/mtdblock3 /system
```

输入下面这行命令进入/system/lib/目录：

```bash
cd /system/lib/
```

输入下面两行命令为系统文件libcrypto.so和libssl.so做备份(它们将被新的文件替换，在修改系统文件前做备份是个好习惯)：

```bash
cp libcrypto.so libcrypto-orig.so
cp libssl.so libssl-orig.so
```

输入下面三行命令将我们下载的三个.so文件复制到/system/lib/目录下：

```bash
cp /sdcard/system/lib/libcrypto.so libcrypto.so
cp /sdcard/system/lib/liblzo.so liblzo.so
cp /sdcard/system/lib/libssl.so libssl.so
```

输入下面这行命令进入/system/bin/目录：

```bash
cd /system/bin/
```

输入下面两行命令将我们下载的openssl和openvpn文件复制到/system/bin目录下：

```bash
cp /sdcard/system/bin/openssl openssl
cp /sdcard/system/bin/openvpn openvpn
```

<div id="attachment_66" class="wp-caption alignnone" style="width: 490px">![安装OpenVPN](http://bnlt.org/wp-content/uploads/2009/11/CAP2009112216441.jpg "安装OpenVPN")

在超级终端中安装OpenVPN
</div>

在超级终端输入下面的命令：

```bash
openvpn
```

如果出现大量openvpn参数的说明，就表明安装成功了。

<div id="attachment_67" class="wp-caption alignnone" style="width: 490px">![测试OpenVPN的安装](http://bnlt.org/wp-content/uploads/2009/11/CAP200911221634.jpg "测试OpenVPN的安装")

测试OpenVPN的安装
</div>

#### 连接OpenVPN

首先将你要使用的OpenVPN的证书和配置文件复制到sd卡中──一般包括一个crt文件，一个key文件，一个或多个ovpn文件──你的OpenVPN服务提供者可能会单独给出这些文件，也可能只给你一个定制好的OpenVPN安装程序，这种情况下，你通常可以在OpenVPN的安装目录的config目录下获取这些文件。在这里，我们要新建一个名为auth的文件，在这个文件的第一行输入你的OpenVPN用户名，第二行输入密码，然后将这个文件连同前面提到的那些证书和配置文件一起放到一个名为openvpn的文件夹中（自己新建一个），最后将这个文件夹复制到sd卡的根目录下。

将卡放回手机，打开超级终端，

输入下面这行命令进入到/sdcard/openvpn/目录下：

```bash
cd /sdcard/openvpn/
```

输入下面这行命令连接OpenVPN：

```bash
openvpn --config xxx.ovpn --auth-user-pass auth
```

xxx.ovpn是你要使用的配置文件的名称，请根据实际情况调整

auth文件也一样，你可以根据自己的需要改成其他名字，注意如果你是在windows下创建了这个文件，它可能有一个隐藏的.txt后缀，在使用命令行时别忘了加上后缀

如果屏幕上最终出现Initialization Sequence Completed，则表式已经连接成功。

<div id="attachment_68" class="wp-caption alignnone" style="width: 490px">![OpenVPN连接成功](http://bnlt.org/wp-content/uploads/2009/11/CAP200911202239.jpg "OpenVPN连接成功")

OpenVPN连接成功
</div>

#### **小结**

到这里为止，我们已经介绍了Android系统中OpenVPN基本的安装和使用方法，相信大多数人已经能够成功的连上自己OpenVPN服务器，享受自由了。部分玩家甚至已经用OpenVPN突破了CMWAP的限制，不会？也没关系，我会尽快奉上本系列的第二部分──在Android中用OpenVPN突破CMWAP限制。
