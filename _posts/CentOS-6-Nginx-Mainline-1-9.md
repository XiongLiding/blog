---
title: CentOS 6 Nginx Mainline 1.9
categories:
  - DevOps
tags:
  - CentOS
  - Nginx
date: 2016-03-27 20:01:07
---

## 准备

*   版本：CentOS 6.7, Nginx 1.9
*   下列所有操作都在命令行终端中进行
*   权限：root

以下操作需要 root 权限，如果当前登录的用户不是 root，请输入 `su` 命令，并在提示中输入正确的密码切换为 root 用户

```bash
su
```


## 实现

###### 1.添加源

在 /etc/yum.repo.d/ 下新建 nginx.repo 文件，文件内容如下：

```nginx
name=nginx repo  
baseurl=http://nginx.org/packages/mainline/centos/6/$basearch/  
gpgcheck=0  
enabled=1  
```

###### 2.安装

通过 yum 安装 nginx

```bash
yum install nginx -y  
```

###### 3.启动

启动 Nginx 服务

```bash
/etc/init.d/nginx start
```

###### 4.验证

用 curl 访问本机地址 127.0.0.1

```bash
curl 127.0.0.1  
```

出现如下结果表明安装成功

```txt
<!DOCTYPE html>  
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p>

<p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at  
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

你也可通过浏览器访问 127.0.0.1 来查看结果

## 说明

###### 源

目录 /etc/yum.repos.d 下存放的是 yum 安装和更新软件时用到的源的配置文件。所谓源，就是软件仓库，里面存放了各种各样的软件，我们通过 yum 安装的软件，都是从这些源中下载过来的。

我们在这个目录下添加了一个新文件 nginx.repo ，这样当我们不再需要这个源的时候，只要将对应的文件删除即可。通过增加和删除配置文件的方式，我们可以实现向 yum 中添加新的源或去除不再使用的源的目的，这比修改原有的配置文件要更方便和安全。

配置文件中，关键的一行是，它指明了源的地址

```
baseurl=http://nginx.org/packages/mainline/centos/6/$basearch/  
```

nginx 官方针对不同的 Linux 发行版提供了不同的源地址，这里的 `centos` 和 `6` 对应了我们使用的 CentOS 6 操作系统。

其中的 `mainline` 表示这是 mainline 分支，这是相对 stable 分支而言的，mainline 会随着开发进度的推进加入一些新的特性，而 stable 的更新仅限于 bug 修复。mainline 是 nginx 官方推荐使用的分支。当然，如果 stable 分支的功能已经能够满足你的需求，并且你担心新特性可能带来 bug，那么可以考虑选择 stable 分支。

###### 安装

`yum install` 是最常用的 yum 命令之一，其作用是安装指定的软件，这里我们指定了 nginx ，参数 `-y` 的作用是当遇到提示询问是否(y/N)时，自动确认并继续。输入这个命令，yum 就会在我们刚刚添加的源中找到 nginx，并自动完成下载和安装的过程。

###### 启动

目录 /etc/init.d 下存放的是初始化脚本(init scripts)，通过这些脚本，我们可以针对很多软件实现启动、停止、重启、重新加载配置文件、查看软件运行状态等操作。

当我们通过 yum 安装 nginx 的时候，也生成了一个对应的初始化脚本 /etc/init.d/nginx 。

执行 `/etc/init.d/nginx start` 即可启动 nginx 服务，相应的：

* /etc/init.d/nginx start 启动
* /etc/init.d/nginx stop 停止
* /etc/init.d/nginx restart 重启
* /etc/init.d/nginx status 显示状态
* /etc/init.d/nginx reload 重新加载配置

还有另一种形式来调用这些命令：

* service nginx start 启动
* service nginx stop 停止
* service nginx restart 重启
* service nginx status 显示状态
* service nginx reload 重新加载配置

###### 验证

curl 是一个用途非常广泛的软件，这里我们使用它来快速的验证 nginx 服务是否已经开启，curl 后面跟上服务器地址，是一种便捷的检测网络服务是否可以正常访问的方法，它会向指定地址发送一个 GET 请求，并将收到的响应主体输出到终端的标准输出当中。

这里，我们收到的内容是 nginx 服务器的欢迎页面，这个页面的本体是 /usr/share/nginx/html/index.html 文件，刚才执行的 curl 命令的返回就是这个文件的内容。

## 其他

###### 远程访问失败

如果本机访问成功，而其他机器（局域网中的其他机器或外网的机器）无法访问，一般是防火墙导致的，可以执行下面的命令关闭防火墙后再试：

```bash
/etc/init.d/iptables stop
```

iptables 是 CentOS 默认安装并启用的防火墙软件，其默认配置是阻止本机以外的其他机器访问本机端口的。在掌握足够多的知识来管理 iptables 配置前，最简单的解决方式就是关闭它。

###### 开机启动

在完成上述操作后，nginx 和 iptables 都是会在开机时自动启动的，如果你想改变这些设置，可以安装 ntsysv 来管理服务器中的所有开机启动项

```bash
yum install -y ntsysv  
```

安装完毕后，启动它

```bash
ntsysv  
```

界面如图：
![ntsysv](https://bnlt.org/content/images/2016/03/N9-WK-7--N-W-5AN-A-S-AK.jpg)

可以通过上下键移动光标，空格键切换启动或不启动，选完之后用 tab 切换至 OK，最后按回车保存。

## 参考

* [http://nginx.org/en/linux_packages.html](http://nginx.org/en/linux_packages.html)
