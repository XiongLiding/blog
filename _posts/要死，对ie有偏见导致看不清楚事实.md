---
title: '要死，对ie有偏见导致看不清楚事实'
categories:
  - 时光机
date: 2010-02-23 11:20:04
---

今天的车票钱没白费……

有一页面，编码utf-8，设置了<meta charset=utf-8>，使用了iframe，内部的编码也是utf-8；在ie中，页面主体显示正常，iframe内乱码；FireFox等中全部正常；结果草率的下结论认为是ie的bug。

事实是ie虽然比较弱，但只要在iframe包含的页面中也添加<meta>标签设置编码，ie也是能正确显示的。
