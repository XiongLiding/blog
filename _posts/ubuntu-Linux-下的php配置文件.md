---
title: ubuntu Linux 下的 php 配置文件
categories:
  - 时光机
tags:
  - PHP
date: 2010-06-24 13:43:34
---

ubuntu下, 用apt-get安装的php5默认情况下将配置文件保存在/etc/php5/

以下是我的机器上的情况(加粗的是输入的命令):

```bash
cd /etc/php5/
ls -l */
```
```
apache2/:
总计 68
-rw-r–r– 1 root root 67459 2010-04-09 16:35 php.ini
cgi/:
总计 156
lrwxrwxrwx 1 root root     9 2010-02-21 22:00 conf.d -> ../conf.d
-rw-r–r– 1 root root 44789 2010-06-22 13:12 php.ini
-rw-r–r– 1 root root 44789 2010-05-24 12:01 php.ini~
-rw-r–r– 1 root root 67459 2010-04-09 16:35 php.ini.ucf-dist
cli/:
总计 68
lrwxrwxrwx 1 root root     9 2010-03-19 15:17 conf.d -> ../conf.d
-rw-r–r– 1 root root 67457 2010-04-09 16:35 php.ini
conf.d/:
总计 28
-rw-r–r– 1 root root 229 2010-04-30 12:30 memcache.ini
-rw-r–r– 1 root root  57 2010-04-09 16:35 mysqli.ini
-rw-r–r– 1 root root  56 2010-04-09 16:35 mysql.ini
-rw-r–r– 1 root root  52 2010-04-09 16:35 pdo.ini
-rw-r–r– 1 root root  60 2010-04-09 16:35 pdo_mysql.ini
-rw-r–r– 1 root root 122 2010-06-24 11:55 xdebug.ini
-rw-r–r– 1 root root 122 2010-06-24 11:47 xdebug.ini~
```

如上所示,/etc/php5下有4个文件夹:apache2 cgi cli conf.d
其中:
apache2 cgi cli下都有php.ini文件,且彼此独立;
cgi cli下有conf.d,且均是指向../conf.d(即/etc/php5/conf.d)的符号链接.
再来看phpinfo的部分输出:
```
Server API	CGI/FastCGI
Configuration File (php.ini) Path	/etc/php5/cgi
Loaded Configuration File	/etc/php5/cgi/php.ini
Scan this dir for additional .ini files	/etc/php5/cgi/conf.d
Additional .ini files parsed	/etc/php5/cgi/conf.d/memcache.ini,
/etc/php5/cgi/conf.d/mysql.ini,
/etc/php5/cgi/conf.d/mysqli.ini,
/etc/php5/cgi/conf.d/pdo.ini,
/etc/php5/cgi/conf.d/pdo_mysql.ini,
/etc/php5/cgi/conf.d/xdebug.ini
```
如上所示,以CGI/FastCGI形式执行php时,会使用相应的cgi文件夹下的php.ini文件,并扫描cgi/conf.d获取附加的配置文件;

同理,如果将php作为shell使用就会使用cli文件下的php.ini和cli/conf.d下的配置文件.

conf.d下的文件,相信大家根据文件名也能看出这些都是另外安装的php模块和附加组件的配置,并且在默认情况下这些配置文件是共用的.

因此,该修改哪个配置文件要视具体情况而定,比如作为apache的模块运行web服务就要修改apache2下的php.ini,以FastCGI的形式在lighttpd和nginx下运行就要修改cgi下的,作为shell脚本运行则修改cli下的.

附加组件的配置默认是3种(数量和具体安装的软件有关)方式共用的,如果想独立设置,也可以将它们写到各自的php.ini中.
