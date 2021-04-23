---
title: Python3-标准库1
date: 2021-04-23 11:17:12
tags:
- Python3
- Python
categories: Python3
---

## os模块

```py
import os

# 返回当前的工作目录
print(os.getcwd())

# 修改当前的工作目录
os.chdir('/server/accesslogs')

# 执行系统命令 mkdir 
os.system('mkdir today')

# 显示所有模块，方法
dir(os)

# 显示帮助
help(os)
```

## shutil高阶文件操作

shutil 模块提供了一系列对文件和文件集合的高阶操作。 特别是提供了一些支持文件拷贝和删除的函数。

参考：[shutil --- 高阶文件操作](https://docs.python.org/zh-cn/3/library/shutil.html#:~:text=shutil,%E6%A8%A1%E5%9D%97%E6%8F%90%E4%BE%9B%E4%BA%86%E4%B8%80%E7%B3%BB%E5%88%97%E5%AF%B9%E6%96%87%E4%BB%B6%E5%92%8C%E6%96%87%E4%BB%B6%E9%9B%86%E5%90%88%E7%9A%84%E9%AB%98%E9%98%B6%E6%93%8D%E4%BD%9C%E3%80%82)

```py
import shutil

shutil.copyfile('data.db', 'archive.db')

shutil.move('/build/executables', 'installdir')
```

## 文件通配符

glob模块提供了一个函数用于从目录通配符搜索中生成文件列表:

```py
import glob

print(glob.glob('*.py'))
```

## 命令行参数

通用工具脚本调用命令行参数。这些命令行参数以链表形式存储于 sys 模块的 argv 变量。

```py
import sys

print(sys.argv)
```

## 错误输出重定向和程序终止

sys 模块有 stdin，stdout 和 stderr 属性，即使在 stdout 被重定向时，后者也可以用于显示警告和错误信息。

```py
sys.stderr.write('Warning, log file not found starting a new one\n')
Warning, log file not found starting a new one
```

大多脚本的定向终止使用 `sys.exit()`。

## 字符串正则匹配

```py
import re

print(re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest'))
# ['foot', 'fell', 'fastest']

print(re.sub(r'(\b[a-z]+) \1', r'\1', 'cat in the the hat'))
# cat in the hat

print('tea for too'.replace('too', 'two'))
# tea for two
```

## 数学

math模块为浮点运算提供了对底层C函数库的访问:

```py
>>> import math
>>> math.cos(math.pi / 4)
0.70710678118654757
>>> math.log(1024, 2)
10.0
```

random提供了生成随机数的工具。

```py
import random

print(random.choice(['apple', 'pear', 'banana']))

print(random.sample(range(100), 10))

print(random.random())
```

## 访问互联网

`urllib.request` 处理从 urls 接收的数据的

`smtplib` 用于发送电子邮件:

```py
>>> from urllib.request import urlopen
>>> for line in urlopen('http://tycho.usno.navy.mil/cgi-bin/timer.pl'):
...     line = line.decode('utf-8')  # Decoding the binary data to text.
...     if 'EST' in line or 'EDT' in line:  # look for Eastern Time
...         print(line)

<BR>Nov. 25, 09:43:32 PM EST

# 需要本地有一个在运行的邮件服务器
>>> import smtplib
>>> server = smtplib.SMTP('localhost')
>>> server.sendmail('soothsayer@example.org', 'jcaesar@example.org',
... """To: jcaesar@example.org
... From: soothsayer@example.org
...
... Beware the Ides of March.
... """)
>>> server.quit()
```

## 日期和时间

具体参考：{% post_link Python3-时间模块 %}

```py
from datetime import date
import datetime

now = date.today()
print(now)

birthday = date(1964, 7, 31)
age = now - birthday
print(age.days)

print(now.strftime("%m-%d-%y. %d %b %Y is a %A on the %d day of %B."))
```

## 数据压缩

`zlib` 支持支持通用的数据打包和压缩格式：zlib，gzip，bz2，zipfile，以及 tarfile。

```py
import zlib

s = b'witch which has which witches wrist watch'
print(len(s))

t = zlib.compress(s)
print(len(t))

print(zlib.decompress(t))

print(zlib.crc32(s))
```

## 性能度量

相对于 timeit 的细粒度，profile 和 pstats 模块提供了针对更大代码块的时间度量工具。

```py
from timeit import Timer

print(Timer('t=a; a=b; b=t', 'a=1; b=2').timeit())

print(Timer('a,b = b,a', 'a=1; b=2').timeit())
```

## 测试模块

`doctest`,`unittest`

参考：

[性能度量](https://www.runoob.com/python3/python3-stdlib.html)