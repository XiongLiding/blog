---
title: Simple nginScript
date: 2018-04-25 21:47:06
categories:
  - 软件开发
tags: 
  - Nginx
---

![nginScript](https://cdn-1.wp.nginx.com/wp-content/uploads/2017/03/introduction-to-nginScript-1000x600-768x461.jpg)

某客户有定时关闭网站的需求，每天晚上10点后关闭，第二天6点开启，节假日全天关闭。

想到通过 nginx 的配置文件实现，但是自己对 nginx 的掌握一直停留在“够用”阶段，涉及逻辑判断的功能从来没用过，只是知道 nginx 一定具备这样的功能。

网上一搜，马上发现一篇关于[通过 lua 控制服务器开关的文章](https://blog.micblo.com/2017/03/12/nginx-Lua-设定网站定时关闭/)，与我的需求非常类似，只是其基于 lua 模块，而我服务器上用的是官方源，不带 lua 模块，而且跑了好多个网站，自己编译或者换用 openresty 有一定风险搞挂掉，慎重起见又是一番搜索，决定用 nginScript 来实现这个功能。

<!--more-->

## 安装
如果你像我一样用了 nginx 的官方源，安装就比较简单了，nginx 官方以动态库的形式提供了 nginScript 模块，在 CentOS 中只要执行 `yum install nginx-module-njs` 就可以自动安装这个模块。

其他系统可以根据实际情况调整包管理器的命令，[官方文档页面](http://nginx.org/en/linux_packages.html#dynmodules)。我博客中也有历史文章介绍如何安装 nginx 官方源。

## 载入模块
通过上述命令安装之后，nginx 并没有自动启用相关功能，需要我们修改配置文件以载入动态模块。

载入模块的命令是 `load_module file` ，其中 file 是模块所在路径，我们是通过包管理工具自动安装的模块，所以并不知道这个模块文件具体在什么位置。在 CentOS 中，我们可以通过 `rpm -ql` 命令找到我们所需的信息。

```
# rpm -ql nginx-module-njs
/usr/bin/njs
/usr/lib64/nginx/modules/ngx_http_js_module-debug.so
/usr/lib64/nginx/modules/ngx_http_js_module.so
/usr/lib64/nginx/modules/ngx_stream_js_module-debug.so
/usr/lib64/nginx/modules/ngx_stream_js_module.so
/usr/share/doc/nginx-module-njs
/usr/share/doc/nginx-module-njs/CHANGES
/usr/share/doc/nginx-module-njs/COPYRIGHT
```

可以看到 /usr/lib64/nginx/modules 下有 4 个 so 文件，这里我们只要用到 ngx_http_js_module.so 即可。因为我们需要用到两个指令 js_include 和 js_set 都包含在这个模块中。

因此最后的载入命令是：

```
load_module modules/ngx_http_js_module.so
```

## 使用 nginScript
这是[官方文档](http://nginx.org/en/docs/http/ngx_http_js_module.html#js_include)对 js_include 和 js_set 的说明：

```
Syntax:	js_include file;
Default:	—
Context:	http
Specifies a file that implements location and variable handlers in njs.

Syntax:	js_set $variable function;
Default:	—
Context:	http
Sets an njs function for the specified variable.
```

非常简洁，所以其实我还是靠网上搜到的内容才知道为何要使用他们，以及如何使用他们。

通过 js_include 引用一个 js 文件，该文件里定义的函数，可以在 js_set 指令中调用，并将执行结果赋值给一个变量

这两个命令的有效范围是 http ，也就是说应该放在 nginx 配置文件的这个位置

```
http {
    ...

    js_include /data/block.js
    js_set $nightOrHoliday nightOrHoliday

    ...
}
```

一开始我没去理解官方文档中 `Context: http` 的意思，将 js_include 和 js_set 放在了 server 中，毕竟只有部分网站需要这个规则，但是配置文件无法生效，通过 `nginx -t` 检测提示 'js_include' is not allowed here 。最后才意识到 Context 可能是这个意思，然后把这两行配置放到 http 下解决了问题。

上面引用了 /data/block.js 这个文件，这个 js 文件会在 nginx 启动或重载配置时被载入内存，而不是像 PHP 那样，每次有用户访问的时候从硬盘读取文件，这保证了 nginx 整体的速度不会受到大的影响。

我在这个文件里定义了两个函数，一个用来计算当前日期和时间，一个用来判断这个时间是否需要屏蔽。

```
function litdate() { // 一个用来计算日期和时间的函数
    ...
}

function nightOrHoliday() {
    var hour = litdate().G;
    if (hour < 6 || hour >= 22) {
        return true;
    }

    var date = litdate().format('md');
    if (date == '0425' ||
        date == '0430' ||
        date == '0501' ||
        date == '0618' ||
        date == '0924' ||
        date == '1001' ||
        date == '1002' ||
        date == '1003' ||
        date == '1004' ||
        date == '1005' ||
        date == '1006' ||
        date == '1007') {
        return true;
    }

    return false;
}
```

这个函数会在 `js_set $nightOrHoliday nightOrHoliday` 中被调用，其返回的 true 或 false 会被赋值给变量 $nightOrHoliday，最后在相关站点的 server 中通过 if 指令判断这个变量以决定是否返回网站内容。

```
server {
    ...
    
    location / {
        if ($nightOrHoliday = 'true') {
            return 444;
        }

        ...
    }

    ...
}
```

这里需要特别说明一下，nginx 配置文件中的变量只有一种类型，就是字符串，js 文件中返回的 true 或者 false ，到了这里就变成了相应的字符串，不能直接用 if 进行判断。

全部设置完毕后，重启或重载配置，使之生效。
