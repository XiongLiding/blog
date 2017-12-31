---
title: IE的奇妙问题
categories:
  - 时光机
date: 2009-08-02 12:52:01
---

```html
name=$(xml).find("name").text();
sex=$(xml).find("sex").text();
phone=$(xml).find("phone").text();
address=$(xml).find("address").text();
```

IE提示对象不支持此属性或方法，且脚本无法运行
其他浏览器正常
把phone改名或在前面加var即在IE中也正常……

不解！

--2009.8.20--

虽然探究事物运作原理是搞技术的人应该具备的精神，但实际上很多问题并不需要知道答案 ，因为这些问题本就不该存在。
严格地遵守语法规则是减少遇到这类问题的最好方法。
