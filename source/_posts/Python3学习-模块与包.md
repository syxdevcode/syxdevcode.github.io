---
title: Python3学习-模块与包
date: 2021-04-15 09:16:41
tags:
- Python3
- Python
categories: Python3
---

## 介绍

### 模块

模块是一个包含所有你定义的函数和变量的文件，其后缀名是 `.py`。模块可以被别的程序引入，以使用该模块中的函数等功能。Python 有很多自带的模块（标准库）和第三方模块，一个模块可以被其他模块引用，实现了代码的复用性。

### 包

包是存放模块的文件夹，包中包含 `__init__.py` 和其他模块，`__init__.py` 可为空也可定义属性和方法，在 Python3.3 之前的版本，一个文件夹中只有包含 `__init__.py`，其他程序才能从该文件夹引入相应的模块、函数等，之后的版本没有 `__init__.py` 也能正常导入，简单来说就是 Python3.3 之前的版本，`__init__.py` 是包的标识，是必须要有的，之后的版本可以没有。

## 引用包与模块

### import 语句

想使用 Python 源文件，只需在另一个源文件里执行 import 语句，语法如下：

```py
import module1[, module2[,... moduleN]
```

当解释器遇到 import 语句，如果模块在当前的搜索路径就会被导入。

搜索路径是一个解释器会先进行搜索的所有目录的列表

一个模块只会被导入一次，不管你执行了多少次import。这样可以防止导入模块被一遍又一遍地执行。

### from … import 语句

Python 的 from 语句让你从模块中导入一个指定的部分到当前命名空间中，语法如下：

```py
from modname import name1[, name2[, ... nameN]]
```

### from … import * 语句

把一个模块的所有内容全都导入到当前的命名空间也是可行的，只需使用如下声明：

```py
from modname import *
```

这提供了一个简单的方法来导入一个模块中的所有项目。不推荐。

### __name__属性

一个模块被另一个程序第一次引入时，其主程序将运行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用 `__name__` 属性来使该程序块仅在该模块自身运行时执行。

```py
#!/usr/bin/python3
# Filename: using_name.py

if __name__ == '__main__':
   print('程序自身在运行')
else:
   print('我来自另一模块')
```

运行输出如下：

```py
$ python using_name.py
程序自身在运行
$ python
>>> import using_name
我来自另一模块
>>>
```

说明： 每个模块都有一个 `__name__` 属性，当其值是 `__main__` 时，表明该模块自身在运行，否则是被引入。

说明：`__name__` 与 `__main__` 底下是双下划线，`_ _` 是这样去掉中间的那个空格。

### as 语句

```py
# 只能通过name来引用
import modulename as name

# 只能通过name来引用
from modulename import attrname as name
```

## 实例

### 模块

```py
#!/usr/bin/python3
# 文件名: python1.py
 
import sys
 
print('命令行参数如下:')
for i in sys.argv:
   print(i)
 
print('\n\nPython 路径为：', sys.path, '\n')
```

输出：

```py
PS C:\Users\DELL> & python e:/git/pythontest/python1.py ab d
命令行参数如下:
e:/git/pythontest/python1.py
ab
d

Python 路径为： ['e:\\git\\pythontest', 'D:\\Users\\DELL\\AppData\\Local\\Programs\\Python\\Python39\\python39.zip', 'D:\\Users\\DELL\\AppData\\Local\\Programs\\Python\\Python39\\DLLs', 'D:\\Users\\DELL\\AppData\\Local\\Programs\\Python\\Python39\\lib', 'D:\\Users\\DELL\\AppData\\Local\\Programs\\Python\\Python39', 'C:\\Users\\DELL\\AppData\\Roaming\\Python\\Python39\\site-packages', 'D:\\Users\\DELL\\AppData\\Local\\Programs\\Python\\Python39\\lib\\site-packages']
```






参考：

[](https://www.runoob.com/python3/python3-module.html)