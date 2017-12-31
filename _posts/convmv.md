---
title: convmv
categories:
  - 时光机
date: 2010-09-18 20:00:52
---

windows 下压缩的文件在linux下解压后，会由于编码原因出现文件名乱码，这时我们可以使用一个实用的小工具`convmv`来转换编码
ubuntu 下用 `sudo apt-get install convmv` 进行安装
使用也很简单

```bash
convmv -f 当前编码 -t 目标编码 -r --notest 要转换的文件或文件夹
```

其中-r表示递归转换子文件夹和其中的文件
不加--notest可以预览转换的结果但不进行实际转换
加上--notest才进行实际转换
