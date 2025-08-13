---
title: Python3学习-输入和输出
date: 2021-04-15 22:39:25
tags:
- Python3
- Python
categories: Python3
---

## 输出

Python输出值的方式: 
* 表达式语句;
* print() 函数;
* 使用文件对象的 `write()` 方法，标准输出文件可以引用 `sys.stdout` 。

如果你希望输出的形式更加多样，可以使用 `str.format()` 函数来格式化输出值。

将输出的值转成字符串，可以使用 `repr()` 或 `str()` 函数来实现。

`str()`： 函数返回一个用户易读的表达形式。
`repr()`： 产生一个解释器易读的表达形式。

<!--more-->
```py
>>> s='Hello,dd'
>>> str(s)
'Hello,dd'
>>> repr(s)
"'Hello,dd'"

# repr() 函数可以转义字符串中的特殊字符
>>> hello = 'hello, dd\n'
>>> hellos = repr(hello)
>>> print(hellos)
'hello, dd\n'

# repr() 的参数可以是 Python 的任何对象
>>> x=123
>>> y=456*123
>>> repr((x, y, ('Hello', 'dd')))
"(123, 56088, ('Hello', 'dd'))"
```

`rjust()` 方法, 它会将字符串靠右, 并在左边填充空格。
`zfill()` 方法, 它会在数字的左边填充 0。

还有类似的方法, 如 `ljust()` 和 `center()`。 这些方法并不会写任何东西, 它们仅仅返回新的字符串。

输出一个平方与立方的表:

```py
for x in range(1, 11):
   print('{0:2d} {1:3d} {2:4d}'.format(x, x*x, x*x*x))

# 或
for x in range(1, 11):
     print(repr(x).rjust(2), repr(x*x).rjust(3), end=' ')
     print(repr(x*x*x).rjust(4))

# 输出
 1   1    1
 2   4    8
 3   9   27
 4  16   64
 5  25  125
 6  36  216
 7  49  343
 8  64  512
 9  81  729
10 100 1000
```

zfill() 用法

```py
>>> '12'.zfill(5)
'00012'
>>> '-3.14'.zfill(7)
'-003.14'
>>> '3.14159265359'.zfill(5)
'3.14159265359'
>>> '3.14159265359'.zfill(15)
'003.14159265359'
```

`str.format()` 的基本使用

```py
print('{},"{}!"'.format('Hello', 'dd'))
# 输出：
Hello,"dd!"

print('{0} , {1}'.format('Hello', 'dd'))
# 输出：
Hello , dd

print('{1} , {0}'.format('Hello', 'dd'))
# 输出：
dd , Hello

print('{var0} , {var1}'.format(var0='Hello', var1='dd'))
# 输出：
Hello , dd

# 位置及关键字参数可以任意的结合
print('Python {0}, {1}, 和 {other}。'.format('Hello', 'dd', other='cc'))
Python Hello, dd, 和 cc。
```

`!a` (使用 `ascii()`), `!s` (使用 str()) 和 `!r` (使用 repr()) 可以用于在格式化某个值之前对其进行转化:

```py
import math
print('常量 PI 的值近似为： {}。'.format(math.pi))
print('常量 PI 的值近似为： {!r}。'.format(math.pi))

常量 PI 的值近似为： 3.141592653589793。
常量 PI 的值近似为： 3.141592653589793。
```

## 输入

### 读取键盘输入

`input()` 内置函数从标准输入(键盘)读入一行文本，`input()` 默认输入的为 str 格式，若用数学计算，则需要转换格式。

```py
str = input("请输入：");
print ("你输入的内容是: ", str)

a=input('请输入数字:')
print(a*2)
```

### 文件输入

参考：{% post_link Python3学习-文件操作 %}

### 以流的形式输入

```py
from urllib import request

response = request.urlopen("http://www.baidu.com/")  # 打开网站
fi = open("project.txt", 'w')                        # open一个txt文件
page = fi.write(str(response.read()))                # 网站代码写入
fi.close()                                           # 关闭txt文件
```

参考：

[Python3 输入和输出](https://www.runoob.com/python3/python3-inputoutput.html)