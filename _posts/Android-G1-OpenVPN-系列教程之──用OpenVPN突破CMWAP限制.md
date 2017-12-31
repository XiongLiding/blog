---
title: Android G1 OpenVPN 系列教程之──用OpenVPN突破CMWAP限制
categories:
  - 时光机
  - Android
tags:
  - Android
  - APN
  - CMWAP
  - DNS
  - OpenVPN
date: 2009-11-23 20:13:56
---

<span style="color: red;">*本文配图已遗失*</span>

### 前言

在上一次的教程里，我们已经讲解了如何[在Android中安装和使用OpenVPN](/2009/11/22/Android-G1-OpenVPN-系列教程之──OpenVPN的安装与使用/)。这次，我们要讲的是如何利用OpenVPN来突破CMWAP限制。

说到CMWAP，我想很多拥有CMWAP包月卡的Android用户(包括我)都使用过cmwrap──它让CMWAP用户能够轻松的使用Android上的大多数网络应用程序，但仍有部分自带的程序和大多数自行安装的程序让它也无能为力。而正确使用OpenVPN则能够让你的手机完全突破CMWAP的限制──但不是说OpenVPN能够取代cmwrap──OpenVPN也有自己的缺点，本文当然只讨论它的优点，我们将在后面的教程中讨论它的缺点。

本教程的所有内容建立在你已经正确安装了OpenVPN的前提下，如果没有，清先参考[本教程的第一部分](/2009/11/22/Android-G1-OpenVPN-系列教程之──OpenVPN的安装与使用/)安装OpenVPN

### OpenVPN配置文件

如果你已经安装好了OpenVPN并且能在普通的网络环境下(例如WIFI或CMNET)成功地连接OpenVPN服务器，那么只需对原有的配置文件进行小小的更改。

打开你要使用的配置文件(如xxx.ovpn)

找到proto开头的一行，这一行一般会是

```
proto tcp
```

或

```
proto udp
```

将这一行改为proto tcp，如果没有这一行就自己加上。

我们要通过CMWAP的HTTP代理来连接OpenVPN服务器，而HTTP代理必须在TCP协议下使用。

接下来设置具体的代理，在配置文件中加入

```
http-proxy 10.0.0.172 80
http-proxy-timeout 20
http-proxy-retry
http-proxy-option AGENT "NokiaN90"
```

第一行设置了代理的IP和端口，正是我们熟悉的CMWAP网关
第二行设置了尝试的时间，超过20秒未响应就认为没有连上
第三行表示连接失败就重试
第四行是将客户端标志伪装成NokiaN90,这是为了防止移动屏蔽你使用的客户端(我不确定Android是否需要伪装，不过IE之类的确实会被移动屏蔽)

保存这个配置文件。

### 用OpenVPN突破CMWAP限制

#### 连接OpenVPN

好了，你现在可以尝试用这个配置文件来连接OpenVPN了

进入超级终端，输入(如果你是根据我的教程来安装和命名就直接按照下面的内容输入，否则请自行调整)：

```bash
cd /sdcard/openvpn/
openvpn --config xxx.ovpn --auth-user-pass auth
```

如果修改前的配置文件能够正常使用，并且移动没有将你的OpenVPN服务提供商列入屏蔽的名单，那么你已经能够看到标志着连接成功的Initialization Sequence Completed了，那么好的，问题来了……

#### 问题来了(APN与DNS)

请先关闭cmwrap等软件，避免在测试过程中产生干扰以致于无法正确判断真实的情况并采取相应的措施──能重启或者使用工具将其他软件都关闭更好。

打开系统自带的浏览器，访问www.ip.cn查看自己的IP

这时会出现三种情况(请确认超级终端中显示的还是Initialization Sequence Completed)：

第一种，显示的IP仍然是自己所在地的移动的IP(我想绝大多数人是这一种)，别怕，这不代表OpenVPN出了问题，这其实是Android系统的问题──也许是HIAPK的ROM中默认的APN的功劳。

<div id="attachment_79" class="wp-caption alignnone" style="width: 490px">![显示的是移动的IP](http://bnlt.org/wp-content/uploads/2009/11/CAP200911231812.jpg "显示的是移动的IP")

显示的是移动的IP
</div>

第二种，显示的IP是OpenVPN服务器的IP(我想只有很小一部分人是这一种)，我想你应该成功了，不过别急，喂，还没下课呢……我不能保证你每次连接都是这个结果，还是虚心看看遇到另两种情况时该怎么对付吧。

<div id="attachment_80" class="wp-caption alignnone" style="width: 490px">![显示的是OpenVPN服务器的IP](http://bnlt.org/wp-content/uploads/2009/11/CAP2009112318411.jpg "显示的是OpenVPN服务器的IP")

显示的是OpenVPN服务器的IP
</div>

第三种，直接提示找不到网页，尝试访问google却能连上。

<div id="attachment_85" class="wp-caption alignnone" style="width: 490px">![由于DNS问题找不到网页](http://bnlt.org/wp-content/uploads/2009/11/CAP200911232208.jpg "由于DNS问题找不到网页")

由于DNS问题找不到网页
</div>

下面根据这三种情况分别进行分析：

**\*请先断开OpenVPN的连接**(方法是在超级终端按返回键)，如果你不怕折腾可以试试不断开……

对于出现第一种情况的朋友，请进入你的APN设置界面(我的是“设置>无线控件>移动网络设置>接入点名称”)，查看你所使用的CMWAP接入点，

我想是和下图第一张类似，是的话，请改成第二张这样──将代理和端口清空，名称可以用自己喜欢的，改完别忘记保存。

<div id="attachment_81" class="wp-caption alignnone" style="width: 330px">![设置了代理(这不是我们想要的)](http://bnlt.org/wp-content/uploads/2009/11/CAP200911231759.jpg "设置了代理")

设置了代理(这不是我们想要的)
</div>
<div id="attachment_82" class="wp-caption alignnone" style="width: 330px">![没有代理(这才是我们要的)](http://bnlt.org/wp-content/uploads/2009/11/CAP200911231844.jpg "没有代理")

没有代理(这才是我们要的)
</div>

Android系统中接入点的代理和端口设置只对浏览器有效，有人觉得这是bug，不过我认为这其实是设计者没有放对地方而对大家产生了误导──我觉得这个本来就是浏览器的代理设置(而不是APN的)，因为即使连接了OpenVPN，这里的代理设置仍然影响浏览器，其作用方法和电脑上的浏览器的代理设置完全一样──这个代理和端口的设置就应该被放到浏览器的设置里去。

回到我们的教程……

再次进入超级终端，连接OpenVPN──刚才没有断开的朋友可能已经发现OpenVPN正在自动尝试重新连接──而且一直连不上(连上也有可能，不过几率很小，一直连不上的话请重启手机再进行这一步)。

好，等连接好了可以再次打开浏览器，访问www.ip.cn查看自己的IP(请刷新一下，防止缓存搞鬼)。

一般情况下，你会遇到上面所说的第三种情况，这是因为没有可用的DNS，解决方法如下：

呃，请进入超级终端，然后退出，再进入(其实是为了退出OpenVPN……)，输入下面这行来设置一个DNS

```bash
setprop net.dns1 208.67.222.222
```

这里设置的正是大名鼎鼎的OpenDNS，大家闲着没事可以把自己的电脑的DNS也设成这个

再次连接OpenVPN，然后查看自己的IP。

看来应该能够出现前面所讲的第二种情况了。

这种情况下，你的手机已经突破了CMWAP的限制，就像在用CMNET或者WIFI上网。对，访问网站不会再出现乱码了，数据同步中的日历和联系人也能成功同步了(cmwrap说它不行)，其他联网的程序也不在话下。

不过不要开心的太早，由于是通过超级终端来连接OpenVPN，一个不留神这个进程就可能被挤到后台运行──如果你因为太开心而迫不及待地打开很多程序去试着联网就更容易把超级终端挤跑──再过一段时节，可能就被系统的内存回收机制给“终结”了，OpenVPN连接也就中断了。

非正常的退出OpenVPN会使再次连接变得很困难──在电脑上也有这个问题──你可以选择重启后再连，也许你会觉得这很麻烦，但这已经是我所知道的最简单可靠的方法了。

### 小结

本文中我们讲解了如何设置.ovpn配置文件以使OpenVPN在CMWAP下工作并提升CMWAP的价值，也讲解了APN和DNS的设置，同时提到了使用OpenVPN时可能遇到的其他问题和简单的解决方法。下一次，我们将讨论Android中OpenVPN所表现出来的缺点──更重要的是如何来避免和解决这些问题。
