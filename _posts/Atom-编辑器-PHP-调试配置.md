---
title: Atom 编辑器 PHP 调试配置
categories:
  - DevOps
tags:
  - PHP
  - XDebug
date: 2017-06-26 13:24:12
---

# 环境

* macOS 10.12
* php 7.1
* atom 1.16
* brew 1.2 

# 安装和配置

## xdebug

打开命令行终端，输入以下命令

```bash
brew install php71 php71-xdebug
```
brew 是 macOS 下的包管理器，php71 是 php7.1 的包名，如果你使用其他系统，请自行替换成相应的命令

### 配置

xdebug 的配置在 /usr/local/etc/php/7.1/conf.d/ext-xdebug.ini 中[注1]，增加下列配置

```
xdebug.remote_enable=1
xdebug.remote_host=127.0.0.1
xdebug.remote_connect_back=1    # Not safe for production servers
xdebug.remote_port=9332
xdebug.remote_handler=dbgp
xdebug.remote_mode=req
xdebug.remote_autostart=false
```

这里将 remote_port 设置成 9332 是为了避免和 php-fpm 的默认端口 9000 冲突，你也可以自行选择一个端口，但是必须和下面的 php-debug 插件中指定的端口一致

remote_autostart 设置成 false 可以配合浏览器插件，按需开启调试功能。

#### [注1]
如果你的配置文件不在这个位置，可以在命令行终端中输入

```bash
php -i | grep with-config-file-scan-dir
```

找到配置文件所在目录

## php-debug

打开插件安装界面，搜索 php-debug

![php-debug](https://ws1.sinaimg.cn/large/006tNc79gy1fgyjnyo9lej30lt09swfo.jpg "php-debug")

点击 install 安装

### 配置
安装完毕，点击 Settings，打开 php-debug 的配置界面
找到 Server Port 选项，默认是 9000，改成 9332，和 xdebug 的 remote_port 一致。
![server port](https://ws2.sinaimg.cn/large/006tNc79gy1fgyjqmwc0qj30gp07fjrl.jpg "server port")

php-debug 安装完毕后，atom 的左下角会有一个带虫子图表的 PHP Debug 按钮，点击展开，在五个调试按钮的右边可以看到 Listening on port 9332...，如果你看到的还是 9000，说明 Server Port 的修改尚未生效，可以尝试重启 atom 编辑器。

![](https://ws4.sinaimg.cn/large/006tNc79gy1fgyjryascuj30nw07u0t2.jpg "在这里输入图片标题")

## xdebug-helper

xdebug helper 是一个 chrome 插件，让我们可以选择性的开启 xdebug 的代码调试和性能调优工具，可以在 php-debug 的配置页面找到它的链接，也可以直接在 chrome 商城查找。

### 配置

安装完毕后，chrome 地址栏右侧会多出一个虫子图标，右键点击，进入选项，在 IDE key 选择 other，右侧输入框填写你的 key，然后点击 save 保存。

![输入图片说明](https://ws2.sinaimg.cn/large/006tNc79gy1fgyjvls8u4j30f6038gls.jpg "在这里输入图片标题")

key 是怎么来的呢？在命令行终端输入：

```bash
php -i | grep IDE
```

会得到类似：

```
IDE Key => xiongliding
```

右边的值就是你的 key 了。

php -i 可以理解为命令行版的 phpinfo 页面。

对于 macOS 和 brew 用户，安装 xdebug 时会自动以你的用户名为 key。

# 测试

新建一个目录，创建一个简单的 hello.php 文件：

```php
<?php
echo "Hello, World";
````

将命令行终端切换到该目录下，输入

```bash
php -S 0.0.0.0:8888
```

这时用浏览器访问 [http://127.0.0.1:8888/hello.php](http://127.0.0.1:8888/hello.php) 可以看到 Hello, World

现在，点击编辑器行号右边的位置设置断点，可以看到一个小圆点，行号变绿。

![输入图片说明](https://ws2.sinaimg.cn/large/006tNc79gy1fgyjzs918nj307e02vdft.jpg "在这里输入图片标题")

切换到浏览器，左键点击 xdebug helper 的虫子图标，在弹出菜单选择 debug 。

![输入图片说明](https://ws3.sinaimg.cn/large/006tNc79gy1fgyjxqnmxjj303k04z74a.jpg "在这里输入图片标题")

刷新浏览器，浏览器会保持在加载状态，回到编辑器，可以看到断点所在行已经高亮显示，下方 PHP Debug 的五个按钮也都变成可用状态，原来显示 Listening on port 9332... 的位置变为 Connected

![输入图片说明](https://ws2.sinaimg.cn/large/006tNc79gy1fgyk7fdab3j30ku0bedgo.jpg "在这里输入图片标题")

现在，你可以进入自己的项目目录，通过 php -S 建立临时的 php 服务来调试自己的代码了。如果原来使用的是 php-fpm ，那么重启 php-fpm 服务，也可以使调试在自己的项目中生效[注2]。

#### [注2]

默认前提是你的 php-fpm 和 php-xdebug 是用相同的包管理器安装的对应版本。
