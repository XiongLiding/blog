---
title: MySQL LOAD DATA LOCAL INFILE Skipped
categories:
  - 时光机
tags:
  - MySQL
date: 2010-06-12 12:54:18
---

使用MySQL的LOAD DATA LOCAL INFILE载入文件时可能会遇到部分记录没有导入(在执行结果反馈的信息中显示为Skipped: n)的情况.
如果你想知道哪些记录被跳过了,请使用LOAD DATA INFILE,它不会跳过那些记录,而是提示错误和出错的原因.
以下是MySQL文档中关于这个问题的tips.
http://dev.mysql.com/doc/refman/5.1/en/load-data.html

> Posted by Clive le Roux on February 2 2009 12:20am	[Delete] [Edit]
> If you get “Skipped records” using “LOAD DATA LOCAL INFILE” copy the data file to the actual database server and do the load without the “LOCAL” keyword.
> This will then stop when an error occurs, 9 times out of 10 it will be index issues and you will know why there are skipped records.

e.g.
```
LOAD DATA LOCAL INFILE ‘myinfile.txt’;
Query OK, 288168 rows affected (1 min 44.49 sec)
Records: 494522 Deleted: 0 Skipped: 206354 Warnings: 0

LOAD DATA INFILE ‘/data/input/myinfile.txt’;
Query OK, 252243 rows affected (0.02 sec)
ERROR 1062 (23000): Duplicate entry ’5935009001-2008-08-03 04:19:18′ for key 1
```
