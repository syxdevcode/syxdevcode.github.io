---
title: Python3学习-基础语法
date: 2021-01-04 21:06:00
tags:
- Python3
- Python
categories: Python3
---

## 编码

Python2 中默认编码为 ASCII，内容如果为汉字，不指定编码则会乱码，想要指定编码为 UTF-8，Python 中通过在开头加入:

```py
# -*- coding: UTF-8 -*-
```

Python3 中默认编码为 UTF-8，因此在使用 Python3 时，通常不需指定编码。

## 标识符

标识符是用于给变量、函数、语句块等命名，Python 中标识符由字母、数字、下划线组成，第一个字符必须是字母表中字母或下划线 _，不能以数字开头，区分大小写。

在 Python 3 中，可以用中文作为变量名，非 ASCII 标识符也是允许的了。

以下划线开头的标识符的特殊含义：

* 单下划线开头的标识符，如：_xxx ，表示不能直接访问的类属性，需通过类提供的接口进行访问，不能用 `from xxx import *` 导入；
* 双下划线开头的标识符，如：__xx，表示私有成员；
* 双下划线开头和结尾的标识符，如：__xx__，表示 Python 中内置标识，如：__init__() 表示类的构造函数。

## 关键字

```py
>>> import keyword
>>> keyword.kwlist
['False', 'None', 'True', '__peg_parser__', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

## 注释

* 单行注释以 # 开头。
* 多行注释可以用多个 # 号，三个单引号（'''）或三个双引号（"""）。

```py
# 第一个注释
# 第二个注释
 
'''
第三注释
第四注释
'''
 
"""
第五注释
第六注释
"""
```

## 行与缩进

Python 使用缩进来表示代码块，控制类、函数、逻辑判断等，，不需要使用大括号 {} ，缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数，如果语句缩进数的空格数不一致，会导致运行错误。

```py
if True:
    print ("True")
else:
    print ("False")
```

## 多行语句

如果Python 的语句很长，可以使用反斜杠 `(\)` 来实现多行语句，例如：

```py
total = one + \
        two + \
        three
```

如果包含在 []、{}、() 括号中，则不需要使用反斜杠 `(\)`。如下所示：

```py
total = ['item_one', 'item_two', 'item_three',
        'item_four', 'item_five']
```