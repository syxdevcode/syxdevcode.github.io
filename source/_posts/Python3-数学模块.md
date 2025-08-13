---
title: Python3-数学模块
date: 2021-04-26 09:09:55
tags:
- Python3
- Python
categories: Python3
---

## Python 数学模块

* **math**	提供了对 C 标准定义的数学函数的访问（不适用于复数）
* **cmath**	提供了一些关于复数的数学函数
* **decimal** 为快速正确舍入的十进制浮点运算提供支持
* **fractions**	为分数运算提供支持
* **random** 实现各种分布的伪随机数生成器
* **statistics** 提供了用于计算数字数据的数理统计量的函数

比较常用的模块：`math`、`decimal` 和 `random`。

<!--more-->

## math 模块

参考：[math --- 数学函数](https://docs.python.org/zh-cn/3/library/math.html)

* **math.fabs(x)** 返回 x 的绝对值。
* **pow(x, y)** 返回 x 的 y 次幂。
* **fsum(iterable)** 返回迭代器中所有元素的和。
* **sqrt(x)** 返回 x 的平方根。
* **trunc(x)** 返回 x 的整数部分。

常量

```py
import math

# 常量 e
print(math.e)
# 常量 π
print(math.pi)
```

## decimal 模块

参考：[decimal --- 十进制定点和浮点运算](https://docs.python.org/zh-cn/3/library/decimal.html)

decimal 模块为正确舍入十进制浮点运算提供了支持，相比内置的浮点类型 float，它能更加精确的控制精度，能够为精度要求较高的金融等领域提供支持。

decimal 在一个独立的 context 下工作，可以使用 getcontext() 查看当前上下文

```py
from decimal import *

print(getcontext())
# Context(prec=28, rounding=ROUND_HALF_EVEN, Emin=-999999, 
# Emax=999999, capitals=1, clamp=0, flags=[], traps=[InvalidOperation, 
# DivisionByZero, Overflow])
```

`prec` 为一个 [1, MAX_PREC] 范围内的整数，用于设置该上下文中算术运算的精度。

`getcontext().prec = 6` 来重新设置精度:

```py
from decimal import *

getcontext().prec = 6
print(Decimal(1) / Decimal(7))
# 0.142857
```

## random 模块

各种分布的伪随机数生成器。

* **choice(seq)** 从非空序列 seq 返回一个随机元素。
* **random()** 返回 [0.0, 1.0) 范围内的一个随机浮点数
* **uniform(a, b)** 返回 [a, b) 范围内的一个随机浮点数。
* **randint(a, b)** 返回 [a, b] 范围内的一个随机整数。
* **randrange(start, stop[, step])** 返回 [start, stop) 范围内步长为 step 的一个随机整数。
* **shuffle(x[, random])** 将序列 x 随机打乱位置。
**sample(population, k)** 返回从总体序列或集合中选择的唯一元素的 k 长度列表，用于无重复的随机抽样。

```py
import random

print(random.random())
print(random.uniform(1.1, 9.9))
print(random.randint(1, 10))
print(random.randrange(1, 10))
print(random.randrange(1, 10, 2))
print(random.choice('123456'))
print(random.choice('abcdef'))

l = [1, 2, 3, 4, 5, 6]
random.shuffle(l)
print(l)

l = [1, 2, 3, 4, 5, 6]
print(random.sample(l, 3))
```

参考：

[数字和数学模块](https://docs.python.org/zh-cn/3/library/numeric.html)

[Python 基础（十九）：数学相关模块](https://ityard.blog.csdn.net/article/details/104085908)