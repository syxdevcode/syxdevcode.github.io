---
title: Python3学习-迭代器与生成器
date: 2021-04-13 13:58:45
tags:
- Python3
- Python
categories: Python3
---

## 迭代器

迭代器是一个可以记住遍历的位置的对象。

迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。

迭代器有两个基本的方法：`iter()` 和 `next()`。

字符串，列表或元组对象都可用于创建迭代器：
<!--more-->
```py
>>> list=[1,2,3,4]
>>> it = iter(list)    # 创建迭代器对象
>>> print (next(it))   # 输出迭代器的下一个元素
1
>>> print (next(it))
2
```

迭代器对象可以使用常规for语句进行遍历：

```py
#!/usr/bin/python3
 
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象
for x in it:
    print (x, end=" ")
```

输出结果：

```py
1 2 3 4
```

next() 函数:

```py
#!/usr/bin/python3
 
import sys         # 引入 sys 模块
 
list=[1,2,3,4]
it = iter(list)    # 创建迭代器对象
 
while True:
    try:
        print (next(it))
    except StopIteration:
        sys.exit()

# 输出
1
2
3
4        
```




### 迭代

Python 中有一些对象可以通过 for 来循环遍历，比如：列表、元组、字符等，遍历的过程就是迭代。

以字符串为例，如下所示：

```py
for i in 'Hello':
    print(i)
```

### 可迭代对象

可迭代对象需具有 `__iter__()` 方法，它们均可使用 `for` 循环遍历，我们可以使用 `isinstance()` 方法来判断一个对象是否为可迭代对象：

```py
from collections import Iterable

print(isinstance('abcd', Iterable))
print(isinstance({1, 2, 3}, Iterable))
print(isinstance(1024, Iterable))
```

结果：

```py
True
True
False
```

### 迭代器

迭代器需要具有 `__iter__()` 和 `__next__()` 两个方法，这两个方法共同组成了迭代器协议，通俗来讲迭代器就是一个可以记住遍历位置的对象，迭代器一定是可迭代的，反之不成立。

* `__iter__()`：返回迭代器对象本身
* `__next__()`：返回下一个迭代器对象

迭代器对象本质是一个数据流，它通过不断调用 `__next__()` 方法或被内置的 `next()` 方法调用返回下一项数据，当没有下一项数据时抛出 `StopIteration` 异常迭代结束。

**实例**

创建一个返回数字的迭代器，初始值为 1，逐步递增 1：

```py
class MyNumbers:
  def __iter__(self):
    self.a = 1
    return self
 
  def __next__(self):
    x = self.a
    self.a += 1
    return x
 
myclass = MyNumbers()
myiter = iter(myclass)
 
print(next(myiter))
print(next(myiter))
print(next(myiter))
print(next(myiter))
print(next(myiter))
```

执行输出结果为：

```py
1
2
.
5
```

StopIteration
StopIteration 异常用于标识迭代的完成，防止出现无限循环的情况，在 `__next__()` 方法中我们可以设置在完成指定循环次数后触发 `StopIteration` 异常来结束迭代。`raise` 关键字用于引发一个异常。

在 20 次迭代后停止执行：

```py
class MyNumbers:
  def __iter__(self):
    self.a = 1
    return self
 
  def __next__(self):
    if self.a <= 20:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration
 
myclass = MyNumbers()
myiter = iter(myclass)
 
for x in myiter:
    print(x)
```

输出结果:

```sh
1
2
...
19
20
```

## 生成器

使用了 `yield` 的函数被称为生成器（`generator`）。

yield 是一个关键字，yield 返回的是一个生成器（在 Python 中，一边循环一边计算的机制，称为生成器）。

作用：有利于减小服务器资源，在列表中所有数据存入内存，而生成器相当于一种方法而不是具体的信息，用多少取多少，占用内存小。

跟普通函数不同的是，生成器是一个返回迭代器的函数，只能用于迭代操作，可以理解为生成器就是一个迭代器。

在调用生成器运行的过程中，每次遇到 `yield` 时函数会暂停并保存当前所有的运行信息，返回 `yield` 的值, 并在下一次执行 `next()` 方法时从当前位置继续运行。

调用一个生成器函数，返回的是一个迭代器对象。

使用 `yield` 实现斐波那契数列

```py
#!/usr/bin/python3
 
import sys
 
def fibonacci(n): # 生成器函数 - 斐波那契
    a, b, counter = 0, 1, 0
    while True:
        if (counter > n): 
            return
        yield a
        a, b = b, a + b
        counter += 1
f = fibonacci(10) # f 是一个迭代器，由生成器返回生成
 
while True:
    try:
        print (next(f), end=" ")
    except StopIteration:
        sys.exit()
```

输出：

```
0 1 1 2 3 5 8 13 21 34 55
```

参考：

[Python 基础（十六）：迭代器与生成器](https://ityard.blog.csdn.net/article/details/103897131)

[https://docs.python.org/zh-cn/3/tutorial/classes.html#iterators](https://docs.python.org/zh-cn/3/tutorial/classes.html#iterators)