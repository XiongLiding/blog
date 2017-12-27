---
title: CodeIgniter MY_Model
date: 2009-07-19
categories:
  - 程序设计
tags:
  - 时光机
  - PHP
  - CodeIgniter
---

–2009.10.12–
本文最早提到的添加
```php
$this->load->library(’model’);
$this->load->library(’my_model’);
```

的方法在忽略大小写的系统中可能会导致重复定义my_model的错误，可能会在换服务器时带来不必要的麻烦，请不要使用，正确的方法是将my_model.php文件重命名为MY_model.php。

–2009.8.19–
终于找到根本原因了。
根本的解决方法：看看是不是将文件命名为my_model.php了，将文件名改为MY_model.php就可以避免这个错误。

–2009.7.19–

MY_Model Can’t find
在载入模型的语句前加上
```php
$this->load->library(’model’);
$this->load->library(’my_model’);
```
这个不规范的方法可能带来其他问题，请不要使用，具体参看–2009.10.12–部分
