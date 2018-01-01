---
title: CentOS 6 安装 gearman 和它的 php 扩展
categories:
  - DevOps
tags:
  - CentOS
  - PHP
  - Gearman
date: 2016-03-04 17:58:47
---

### 0\. [PHP 中的 gearman 扩展](http://php.net/manual/en/gearman.requirements.php)

我的服务器使用的是 ius 的 php5.5 ，如果你使用其他源和版本，请自行替换部分包名

### 1\. 安装 epel 和 ius 源

```bash
yum install http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/epel-release-6-5.noarch.rpm
yum install http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/ius-release-1.0-11.ius.centos6.noarch.rpm
```

### 2\. 安装 gearman ，用于运行 gearman 服务

```bash
yum install gearmand
```

### 3\. 安装 libgearman 和 libgearman-devel ，用于编译 php 扩展

```bash
yum install libgearman libgearman-devel
```

### 4\. 安装 pecl 和其他依赖

```bash
yum install php-pear php55u-devel
```

### 5\. 安装 php 扩展

```bash
pecl install gearman
```

### 6\. 修改配置文件 /etc/php.ini 增加 extension=gearman.so

### 7.验证安装结果

```
$ php -m | grep gearman
gearman
```
