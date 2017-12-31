---
title: PHP中“数据库和页面编码相同仍乱码”的解决方法
categories:
  - 时光机
  - 快修
tags:
  - MySQL
  - PHP
date: 2010-03-31 11:19:22
---

这种情况通常出现在网页服务器和数据库服务器分别运行在两台不同的机器上时。
数据库中的数据读出来后传给网页服务器时也是按照一定的编码规则来的，把这个编码也设成一致的就能解决这个问题：
比如数据库和页面编码都是utf8，则在数据库连接后执行：

```sql
mysql_set_charset("utf-8");
```

或

```sql
mysql_query("set names 'utf-8'");
```

http://www.php.net/manual/en/function.mysql-set-charset.php
