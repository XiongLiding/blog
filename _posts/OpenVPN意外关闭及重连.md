---
title: OpenVPN意外关闭及重连
date: 2010-01-10
categories:
  - 时光机
---

openvpn连接意外关闭时可能会导致无法再次连接，此时可以将系统中的openvpn进程彻底关闭后再试。

先用ps命令

```bash
ps -C openvpn
```

查看系统中的openvpn进程号——即PID命令下面对应的数字：

然后用kill命令杀死这个进程后再连接就可以了

此方法适用于Android手机和桌面Linux。
