---
title: 'Kohana 3.2 in sina app engine'
categories:
  - 时光机
date: 2011-11-18
---

新浪是目前国内比较领先的云计算服务供应商。Sina App Engine选择了PHP做为开发环境，但和其他云计算平台一样，在使用上还是有一些限制的，由于禁用了对本地文件的读写权限，Kohana在默认设置下是不能在SAE上正常运行的。这里我们就来介绍一些方法，通过简单的修改，让Kohana3.2运行在SAE平台上。

## Quick-and-dirty
首先介绍这个快速而简单的解决方案，如果你不打算使用缓存和日志功能，这个方案足够的简单和方便，并能保证kohana运行起来。

请先上传一份原始的Kohana 3.2到你的sae上以便参照操作。

### 删除install.php
install.php本是用来检测服务器环境是否符合kohana运行要求的。经过检测，我们会发现sae的环境不能满足要求，原因是缓存目录和日志目录不能写入。

这里，我们直接删除这个文件。

### 修改application/bootstrap.php
文件application/bootstrap.php修改前后差异见下图（左边为修改前，右边为修改后）： bootstrap.php修改前后对比

我们可以看到有三处修改，分别对应行号49、84、90，具体操作解释如下：

####. 注释掉行49 ini_set(‘unserialize_callback_func’, ‘spl_autoload_call’);
注释掉行49，否则会看到下面的警告信息，提示没有权限设置unserialize_callback_func。

SAE_Warning: Set the ini directive ‘unserialize_callback_func’ without permission in application/bootstrap.php on line 49

原因是sae对ini_set的使用做了限制，目前不允许通过ini_set设置unserialize_callback_func。具体参见：http://sae.sina.com.cn/?m=feedback&a=view&id=4910

对大多数人而言注释掉这行不会有什么影响。除非满足以下情况：代码中使用了unserialize()，并且其解析的结果是一个对象，同时这个对象对应的类还没有定义。这种情况下就会导致一个错误。

#### 增加行 84 ‘cache_dir’ ⇒ SAE_TMP_PATH,
在添加这行前，会看到下面的错误信息，提示缓存目录必须为可写。

Kohana_Exception [ 0 ]: Directory APPPATH/cache must be writable

因为sae不允许对本地文件系统进行操作，我们自然没有该目录的写入权限。唯一可以直接使用并提供了写入权限的是一个虚拟目录SAE_TMP_PATH。但是这个目录会在php脚本执行完后销毁，所以实际上并不能真正起到缓存的作用。

这里，我们将cache_dir设置为SAE_TMP_PATH以通过Kohana的检测。

#### 注释掉行90 Kohana::$log→attach(new Log_File(APPPATH.’logs’));
注释掉行90，否则有错误信息如下，提示日志目录必须为可写。

Kohana_Exception [ 0 ]: Directory APPPATH/log must be writable

注释掉这一行会停用Kohana的日志功能，避免报错，但同时也禁用了日志功能。当然，这对Kohana的运行没有任何影响。

### 小结
通过修改上述两个文件，Kohana已经可以在sae上正常运行了，但同时也失去了缓存和日志功能。接下来的文章我们会介绍如何开启缓存和日志。
