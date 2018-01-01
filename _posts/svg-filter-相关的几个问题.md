---
title: Svg Filter 相关的几个问题
categories:
  - 其他
tags:
  - SVG
  - leaflet
date: 2017-06-28 22:52:45
---

在用 [Leaflet](http://leafletjs.com/) 处理 [GeoJSON](http://geojson.org/) 格式的地理信息时，解析结果被以内联的 SVG 格式嵌入网页中。为了使地图更有质感，希望通过添加阴影获得视觉上的高低效果，尝试过程记录如下。

# css

首先，`<svg>` 内的元素是支持 css 样式的，用法也和普通的 css 一样，可以通过 id 或 class 来套用样式，但是 svg 元素具体支持的规则和普通 DOM 元素相差很大

## box-shadow

`box-shadow` 不支持 `svg` 内部的元素

## filter

`filter` 的 `drop-shadow` 也可以制造阴影效果，但是目前只有 Edge 支持对 svg 内部元素使用

# <filter> 元素

```html
<defs>
    <filter id="shadow-water">
        <feGaussianBlur out="blurOut" in="SourceGraphic" stdDeviation="5"></feGaussianBlur>
        <feBlend in="SourceGraphic" in2="blurOut" mode="normal"></feBlend>
    </filter>
</defs>
```

`<filter>` 本身就是 svg 标准的一部分，也是目前兼容性最好的，为 svg 元素添加阴影的方法，但是其本身并没有提供一个直接的添加阴影的方法，上面的例子里是通过将模糊后的图像和原始图像叠加实现的

`<defs>` 需要放置在 `<svg>` 元素内

需要使用此效果的元素，需要设置属性 filter="url(#shadow-water)" 来应用

# svg 元素的操作

在我的环境里，svg 是由 leaflet 生成的，因此自定义的 `<defs>` 只能通过代码动态的插入到已有的 `<svg>` 标签内，但是这里不能使用 jQuery 来操作，类似

```JavaScript
$('svg').prepend('<defs> <filter id="shadow-water"> <feGaussianBlur out="blurOut" in="SourceGraphic" stdDeviation="5"></feGaussianBlur> <feBlend in="SourceGraphic" in2="blurOut" mode="normal"></feBlend> </filter> </defs>');
```

是行不通的，因为 `<svg>` 内的元素类型需要是 SVGElement 才能正常工作，而 jQuery 会将 defs 等元素当作普通 DOM 元素加入到 svg 内部，因此会出现从调试工具里看到代码和手动编写的 svg 文件一致，但实际上无法发挥作用的情况。

这里有一篇文章描述了这个问题，并提供了一个[动态创建 svg 元素的方法](http://http://chubao4ever.github.io/tech/2015/07/16/jquerys-append-not-working-with-svg-element.html)

# 两个 svg 标签

但是上面提到的方法写起来比较麻烦，尤其是需要动态插入的标签数量比较多，并且有嵌套的情况。

最后我尝试了将 defs 写在另一 svg 里来解决这个问题。一个 svg 里的元素是可以引用另一个 svg 里的 filter 的，我预先创建了一个只包含 defs 的 svg 标签，内部只包含了几个需要用到的 filter 效果，而没实质性的元素。这个 svg 默认也会占据一定的屏幕空间，150x300，因此需要加上样式将其隐藏。

# 隐藏提供特效的 svg 标签

首先尝试 style="display: none"，svg 标签被隐藏了，但是特效也一起消失；
然后尝试 style="width: 0; height: 0"，特效得以保留，但仍占据了屏幕大约一行文字的高度；
最终改成 style="position: absolute; width: 0; height: 0"，问题解决
