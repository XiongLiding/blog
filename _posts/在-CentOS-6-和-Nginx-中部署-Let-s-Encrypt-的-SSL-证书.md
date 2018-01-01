---
title: 在 CentOS 6 和 Nginx  中部署 Let's Encrypt 的 SSL 证书
categories:
  - DevOps
tags:
  - CentOS
  - Nginx
  - Https
date: 2016-12-29 23:47:01
---

上周写了一篇文章讲了讲为什么应该使用 HTTPS 加密你的网站，并笼统的介绍了如何申请和使用 Let's Encrypt 提供的 SSL 证书。这次我们说说如何在 CentOS 6 和 Nginx 中部署 Let's Encrypt 的 SSL 证书，过程中可能遇到的问题，以及如何解决。

整体上分成申请证书和使用证书两个部分。

## 申请证书

### 准备工作

在申请证书之前，你必须有一个域名，并配置好 DNS 解析服务，使其指向你将要使用的服务器。说白了，就是你要保证所有人都可以通过在浏览器中输入域名打开你的网站。

此时 Nginx 的配置文件如下（为了便于说明只包含了最基本的信息）:

```
[ngxin]
server {  
    listen 80;
    server_name www.bnlt.org;

    location / {
        root /usr/share/nginx/html;
    }
}
```

其中 80 是 http 协议默认端口，www.bnlt.org 是我的域名，/usr/share/nginx/html 是网站根目录。

### 安装 Certbot

Certbot 是 Let's Encrypt 官方推出的，用来获取 SSL 证书的工具。Certbot 的官网为各大主流操作系统和 Web 服务器提供了安装和使用的指南。下面首先介绍官方提供的方法。

> 下列操作均在 root 用户权限下执行。

#### 官方推荐

针对 CentOS 6 系统，官方推荐的是使用 certbot-auto 脚本。安装方法如下：

```bash
wget https://dl.eff.org/certbot-auto  
chmod a+x certbot-auto  
```

第一个命令将 certbot-auto 下载到当前目录下，第二个命令给予执行权限。

这是一个 shell 脚本，可以通过在当前目录下执行 `./certbot-auto` 来调用。

它首先检查你是否已经安装了最新版本的 certbot，如果尚未安装，则调用 yum 检查和安装相关的依赖，其中包括了 python、python-dev 和 python-pip，然后创建一个虚拟环境（相对独立的、运行 python 的虚拟环境，在里面安装的依赖和库独立于操作系统使用的 python 环境，可以避免影响主系统的运行），在这个虚拟环境里安装一个最新版本的 certbot。用户交给 certbot-auto 的参数都被委派给这个安装在虚拟环境中的 certbot 来执行。

由于这个 certbot 被安装在虚拟环境里，你不能直接在命令行里使用 certbot 命令，只能通过 certbot-auto 脚本间接使用它。

不过使用 certbot-auto 脚本有时会遇到问题，表现为虚拟环境创建完毕后，长时间卡在安装 Python 包的步骤：

```
Creating virtual environment...  
Installing Python packages...  
```

这通常被认为是 pip 的源在国内访问受限导致。

#### 直接使用 pip 安装 certbot

上面讲到了，certbot-auto 是先安装 python-pip，再通过 python-pip 来安装 certbot 的，所以我们也可以自己手动来完成这个过程。通过这个方法安装的 certbot 会成为一个全局的命令，具体安装在 /usr/bin/certbot 。

如果你想自己安装 certbot，我仍然建议你先执行一下 certbot-auto 命令，让它先帮你安装好相关的依赖。

默认情况下 certbot-auto 也会为你安装 python 和 python-pip，但是版本教旧的 python2.6 和 pip7.1，，虽然用来安装和使用 certbot 是完全没有问题的，但每次执行时都会提示你当前使用的版本较老，官方已不再支持，有安全隐患。 

我使用的是 python2.7 以及 pip9.0，可以通过添加 IUS 的源来安装。

添加 IUS 源

```bash
yum install https://centos6.iuscommunity.org/ius-release.rpm  
```

安装 python2.7 和 pip9.0+

```bash
yum install python27 python27-devel python-pip  
```

其中 python27-devel 是运行 certbot 必需的。

全局安装 certbot

```bash
pip install certbot  
```

如果 pip install 速度很慢或卡进度，可以尝试设置 pip 的源为阿里云的镜像，再尝试上面的命令。

在当前用户目录下建立 ~/.pip/pip.conf 文件。内容如下：

```
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com  
```

安装完毕后，你会得到一个新命令 certbot，用 which 命令可以查到其安装的位置。

```bash
which certbot  
/usr/bin/certbot
```

### 获取证书

现在可以使用 certbot 获取 SSL 证书了。

> 如果你使用官方脚本，将下列命令中的所有 certbot 替换为 ./certbot-auto 即可。

这里我们使用 `certbot certonly --webroot` 方式来获取证书，此命令借助已有的 Web 服务实现认证，并生成证书，命令执行过程中不会对网站的正常运行造成影响，以后给证书续期也更平滑。

完整的命令如下：

```bash
certbot certonly --webroot -w /usr/share/nginx/html -d www.bnlt.org  
```

`-w` 参数指定了网站的根目录，`-d` 参数指定了网站的域名。

背后的原理大致如下：执行命令的计算机会和 Let's Encrypt 的服务器进行通信，商定一个字符串，这个字符串以文件的形式保存在`-w` 参数指定的路径下的 `.well-known/acme-challenge/` 目录中，服务器再访问 `-d` 指定的网站目录下的相应文件来进行核实。核对成功就可以基本证明这个域名确实归你所有，Let's Encrypt 就会为你颁发相应的证书了。

> 实际上只能证明当前是你控制了这个域名的解析，如果域名解析被他人恶意控制或污染，他也能生成此域名的证书，因此 Let's Encrypt 提供的证书仅能用于加密，保证信息在传输过程中不被篡改，但不能用来证明域名的所有权。从这个角度来看，只要攻击者能控制或污染域名解析，即使你访问的某个网站显示了绿锁，仍有可能是个钓鱼网站。

执行此命令后，会弹出一个界面问你是否接受相关的协议，选择 OK 并确认即可。你也可以在执行上述命令时加上 --agree-tos 自动接受协议以跳过此步骤。

如果服务器配置正确，命令行参数也无误，那么就能成功完成，提示如下：

```
- Congratulations! Your certificate and chain have been saved at
  /etc/letsencrypt/live/www.bnlt.org/fullchain.pem. Your cert will
  expire on 2017-01-30\. To obtain a new or tweaked version of this
  certificate in the future, simply run certbot again. To
  non-interactively renew *all* of your certificates, run
  "certbot renew"
```

提示信息告诉你证书存放在 /etc/letsencrypt/live/www.bnlt.org 目录下，过期时间是 2017-01-30，最后还告诉你续期的方法是执行 `certbot renew`

而最常见的错误提示如下：

```
An unexpected error occurred:  
The request message was malformed :: No such challenge  
```

遇到这样的情况是因为 Let's Encrypt 的服务器无法在你的网站上找到验证用的文件。你首先要检查 `-w` 和 `-d` 参数是否有误，是否把验证文件放在了别的目录下或者填错了域名，另外还要确保 Let's Encrypt 的服务器能访问到你的网站，比如你是刚做的域名解析，可能对它所处的网络而言域名解析尚未生效。

### 使用证书

要在 Nginx 中用刚才得到的证书来启用 https ，需要将配置文件改为：

```
server {  
    listen 443 ssl;
    server_name www.bnlt.org;

    ssl_certificate /etc/letsencrypt/live/www.bnlt.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.bnlt.org/privkey.pem;

    location / {
        root /usr/share/nginx/html;
    }
}
```

和原来的配置文件相比主要有两处变化，一是将 listen 端口改为 https 协议的默认端口 443，并加上 ssl 说明是加密连接。

还有就是增加了 `ssl_certificate` 和 `ssl_certificate_key`，前者指定公钥，后者指定私钥。

保存配置文件，执行 `/etc/init.d/nginx reload` 使新配置生效

#### http 自动跳转 https

如果你想更进一步，对所有用户强制使用 https，可以在配置文件里增加如下内容：

```
server {  
    listen 80;
    server_name www.bnlt.org;
    return 301 https://$server_name$request_uri;
}
```

此配置会将所有通过 80 端口的 http 协议访问的用户，安全的跳转到对应的 https 版本页面。

同样，执行 `/etc/init.d/nginx reload` 使新配置生效。
