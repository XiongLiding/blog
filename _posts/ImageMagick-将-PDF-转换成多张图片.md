---
title: ImageMagick 将 PDF 转换成多张图片
categories:
  - 其他
tags:
  - ImageMagick
  - Fix
date: 2016-12-20 08:44:02
---

![ImageMagick 将 PDF 转换成多张图片](https://www.imagemagick.org/image/wizard.png)

今天遇到一个需求，客户发来几份PDF，想放到他们的微网站（专门给微信用的网站）上，由于是给手机看的，加上文件页数不多，就想到转换成图片再放上去。

第一时间想到用 ImageMagick 的 convert 命令，网上一搜，命令如下<sup id="fnref:1">[1](https://bnlt.org/imagemagick-jiang-pdf-zhuan-huan-cheng-duo-zhang-tu-pian/#fn:1)</sup>：

```bash 
convert -density 600 foo.pdf foo-%02d.jpg  
```

`-density` 设置了生成的图片的精度，数值越大，图片越清晰（分辨率高），转换也越慢。如果给手机用，普通 A4 大小的 PDF 设置在 200 左右比较合适。

`foo-%02d.jpg` 是希望生成的文件名，`%02d` 部分会替换成页码（从 0 开始），用过 `printf` 函数的应该对这个规则会比较熟悉。

然而在实际使用时遇到了一个意外情况：

```bash
convert: no images defined `foo-%02d.jpg' @ error/convert.c/ConvertImageCommand/3258\.  
```

继续求助搜索引擎，找到解决方案，缺少 gs <sup id="fnref:2">[2](https://bnlt.org/imagemagick-jiang-pdf-zhuan-huan-cheng-duo-zhang-tu-pian/#fn:2)</sup>。gs 即 GhostScript，ImageMagick 用它来解析 PDF 文件。

```bash
brew install gs  
```

安装完 gs ，再执行一次 convert 命令，问题解决。

<div class="footnotes">
1. [http://superuser.com/questions/633698/convert-pdf-to-jpg-images-with-imagemagick-how-to-0-pad-file-names](http://superuser.com/questions/633698/convert-pdf-to-jpg-images-with-imagemagick-how-to-0-pad-file-names) [↩](https://bnlt.org/imagemagick-jiang-pdf-zhuan-huan-cheng-duo-zhang-tu-pian/#fnref:1 "return to article")
2. [http://superuser.com/questions/819277/cant-convert-pdf-into-image-because-of-no-images-defined-error](http://superuser.com/questions/819277/cant-convert-pdf-into-image-because-of-no-images-defined-error) [↩](https://bnlt.org/imagemagick-jiang-pdf-zhuan-huan-cheng-duo-zhang-tu-pian/#fnref:2 "return to article")</div>
