---
title: '_.compact(_.map(_.toArray(n.toString(2)).reverse(), (v, i) => 2 ** i * v))'
categories:
  - It's Magic
tags:
  - JavaScript
  - Underscore
date: 2016-10-13 20:33:32
---

包含 underscore 和 es2016 语法

# 功能

将一个数转换成多个2的N次方的数的和

| 输入 | 二进制 | 输出 | | - | - | - | | 1 | 0001 | [1] | | 6 | 0110 | [2, 4] | | 7 | 0111 | [1, 2, 4] |

# 用途

数据库中以整数形式存储，界面上以多选的形式展现

```html
<select multiple>
  <option value="1">a</option>
  <option value="2">b</option>
  <option value="4">c</option>
</select>

<script>
  $('select').val([2,4])
</script>
```

数据库存了6，取到前端，转换成 [2,4]，同时选中 b 和 c

# 分解动作

假设 n = 6

```JavaScript
n // 6
n.toString(2) // '110'
_.toArray(n.toString(2)) // ['1', '1', '0']
_.toArray(n.toString(2)).reverse() // ['0', '1', '1']
_.map(_.toArray(n.toString(2)).reverse(), (v, i) => 2 ** i * v) // [0, 2, 4]
_.compact(_.map(_.toArray(n.toString(2)).reverse(), (v, i) => 2 ** i * v)) // [2, 4]
```

# 其他写法

```JavaScript
_.chain(n.toString(2)).toArray().map((b, i, a) => 2 ** (a.length - i - 1) * b).compact().value()
_.compact(_.range(32).map((v) => 2 ** v & n)) // 最大处理32位整型
```    
