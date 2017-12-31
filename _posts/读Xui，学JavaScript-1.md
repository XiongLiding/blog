---
title: 读Xui，学JavaScript (1)
categories:
  - 时光机
tags:
  - JavaScript
date: 2011-12-03 11:33:09
---

初篇，hearder & footer —— 匿名函数
Xui的代码以 src/header.js 开头
```javascript
(function () {
```
以 src/footer.js 结尾。
```javascript
})();
```
所有其他代码都包含在两者之间——现在流行的JavaScript框架大多采用了这种形式，让这种写法几乎成了不成文的标准。
我们把上面两段代码连在一起并加以整理：

```javascript
( function () { //<1>
} )();
```

<1> 如果你没有接触过匿名函数，看到这一行就会难以理解。为什么定义函数时可以不指定函数名呢？
我们熟悉的函数定义通常是这样的：

```javascript
function f() { //<2>
}
```
<2>此处定义了一个名为f的函数，虽然这个函数什么也不做，但它是合法的，并且可以在之后以f()的形式调用它。
JavaScript则要灵活的多，允许用下面这种形式定义函数：

```javascript
f = function () {}
```
