---
title: MySQL ; SUBSTR()与CHAR_LENGTH()
categories:
  - 时光机
tags:
  - MySQL
date: 2010-06-03 17:44:54
---

SUBSTR()是SUBSTRING()的别名；最常用的形式和PHP等语言很类似，不过pos是从1开始的，len不能为负，如果负则返回空字符串；multi-byte safe（多字节安全）也就是不会在取子字符串时把汉字等截断，造成乱码；由于长度不能使用负值来表示倒数的位置，所以有时需要用CHAR_LENGTH()来计算字符串的总长度，CHAR_LENGTH()把多字节字符也看成基本单位，一个汉字也是按1计算的。

更多信息http://dev.mysql.com/doc/refman/5.0/en/string-functions.html#function_substr
