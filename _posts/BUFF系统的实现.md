---
title: 'BUFF系统的实现'
categories:
  - 时光机
date: 2011-10-17 07:33:16
---

version 0.1,2011-09

BUFF是很多游戏都在采用的一种临时增益机制。本文讲述如何在基于关系型数据库的网页游戏中实现这一系统；如何扩展该系统；以及如何提高该系统的性能。

## 引言

BUFF是很多游戏都在采用的一种临时增益机制；与之对应，还有用于减益的DEBUFF；部分游戏还引入了不限制持续时间的永久性BUFF。

对于游戏的开发人员来说，BUFF和DEBUFF的差别只是正数和负数的差别，永久性BUFF也只是持续时间很长（超出游戏寿命）的普通BUFF。所以，用统一的系统来实现所有这些机制是完全可行的。

## 1. 这个BUFF有什么作用

### 1.1. BUFF的效果

BUFF的效果是BUFF的本质，也是玩家和开发人员真正关心的内容。所以这里我们先来讨论BUFF的效果，并约定用BUFF_KEY来称呼它。

在demo中，我们只提供一种BUFF效果

 BUFF_KEY   描述
ATK

增加指定点数的攻击力

### 1.2. BUFF的强度

有了BUFF的效果，还需要一个数值来描述BUFF的强度，我们称其为BUFF_VALUE。
通过两者的组合，我们已经可以提供多个BUFF了

 BUFF   BUFF_KEY   BUFF_VALUE     描述
ATK1

ATK

1

增加1点攻击力

ATK2

ATK

2

增加2点攻击力

ATK3

ATK

-1

降低1点攻击力

这里我们提供了ATK1、ATK2、ATK3三个BUFF，ATK1、ATK2分别可以增加1、2点攻击力，ATK3则是一个DEBUFF，可以降低1点攻击力。

## 2. 赐予我力量吧——给某个单位加BUFF

BUFF只有加在特定的单位上才有意义，我们用UNIT来表示某个单位，用TIME表示BUFF的过期时间（UNIX时间戳）

 UNIT   BUFF_KEY   BUFF_VALUE     TIME           描述
U001

ATK

1

1356105599

单位U001拥有增加1点攻击力的BUFF，持续到1356105599

U001

ATK

2

1318774750

单位U001拥有增加2点攻击力的BUFF，持续到1318774750

U001

ATK

-1

1318774760

单位U001拥有降低1点攻击力的DEBUFF，持续到1318774760

如此一来，我们想要知道当前时刻单位U001上ATK类型BUFF的总值，只要找出TIME大于当前时间戳，且BUFF_KEY为ATK的所有记录，并对BUFF_VALUE求和便能得到想要的数值：

SELECT SUM(BUFF_KEY) WHERE UNIT = 'U001' AND TIME > NOW() AND BUFF_KEY = 'ATK'

在时刻1318774745，单位U001有3个有效的ATK类型BUFF，数值为1+2-1=2

在时刻1318774655，单位U001有2个有效的ATK类型BUFF，数值为1-1=0

在时刻1318774765，单位U001有1个有效的ATK类型BUFF，数值为1，事实上，这个BUFF会持续到世界末日。:)

---

Version 0.1

Last updated 2011-10-17 07:33:16 CST
