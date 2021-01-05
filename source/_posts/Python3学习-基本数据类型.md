---
title: Python3学习-基本数据类型
date: 2021-01-05 11:45:05
tags:
- Python3
- Python
categories: Python3
---

Python 中的变量不需要声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。

```py
var1 = 1
var2 = 'a'
```

在 Python 中，所说的"类型"是变量所指的内存中对象的类型。变量可以通过赋值指向不同类型的对象。

可以使用 del 语句删除一些对象引用。

del语句的语法是：

```py
del var1[,var2[,var3[....,varN]]]
```

可以通过使用del语句删除单个或多个对象：

```py
del var
del var_a, var_b
```

## 变量赋值

等号（=）用来给变量赋值。

等号（=）运算符左边是一个变量名，等号（=）运算符右边是存储在变量中的值。

```py
counter = 100           # 整型变量
miles = 1000.0          # 浮点型变量
name= "Hello, Python!"  # 字符串
```

1，同时为多个变量赋值。例如：

创建一个整型对象，值为 1，从后向前赋值，三个变量被赋予相同的数值。

```py
a = b = c = 1
```

2，为多个对象指定多个变量。例如：

两个整型对象 1 和 2 的分配给变量 a 和 b，字符串对象 "Python" 分配给变量 c。

```py
a, b, c = 1, 2, "Python"
```

## 标准数据类型

Python3 中有六个标准的数据类型：

* Number（数字）
* String（字符串）
* List（列表）
* Tuple（元组）
* Set（集合）
* Dictionary（字典）

标准数据类型中：

不可变数据（3 个）：Number（数字）、String（字符串）、Tuple（元组）；
可变数据（3 个）：List（列表）、Dictionary（字典）、Set（集合）。

string、list 和 tuple 都属于 sequence（序列）。

空值用 None 表示

## type()和isinstance()函数

isinstance 和 type ：

* type()：不会认为子类是一种父类类型。
* isinstance()：会认为子类是一种父类类型。

内置函数 type() 可以用来查询变量所指的对象类型。

```py
>>> a,b,c,d=20,5.5,True,5+4j
>>> print(type(a), type(b), type(c), type(d))
<class 'int'> <class 'float'> <class 'bool'> <class 'complex'>
```

isinstance 

```py
>>> a=1234
>>> isinstance(a,int)
True
```

## 数字(Number)类型

Python3 中数字有四种类型：int(整数)、bool(布尔型)、float(浮点数)、complex（复数）。

* int (整数), 可以为任意大小、包含负数，如 1, 只有一种整数类型 int，表示为长整型，没有 python2 中的 Long。
  整型有四种进制表示，分别为：二进制、八进制、十进制、十六进制，说明如下表所示：
    | 种类       | 描述           | 引导符 |
    | :-----:| :----: | :----: |
    |二进制	    |由 0 和 1 组成   |	0b 或 0B
    |八进制	    |由 0 到 7 组成	  | 0o 或 0O
    |十进制	    |默认情况	      |无
    |十六进制	|由 0 到 9、a 到 f、A 到 F 组成，不区分大小写	|0x 或 0X

* bool (布尔), 如 True。
* float (浮点数), 由整数部分和小数部分组成，如 1.23、3E-2
* complex (复数), 由实数部分和虚数部分组成，如 1 + 2j、 1.1 + 2.2j

注意：混合计算时，Python 会把整型转换成为浮点数。

复数由实数部分和虚数部分构成，可以用a + bj,或者 complex(a,b) 表示， 复数的实部a和虚部b都是浮点型

## String（字符串）

字符串用单引号 ' 或双引号 " 括起来，同时使用反斜杠 \ 转义特殊字符。Python 没有单独的字符类型，一个字符就是长度为1的字符串。Python 字符串不能被改变。向一个索引位置赋值，比如 word[0] = 'm' 会导致错误。

字符串的截取：**索引值以 0 为开始值，-1 为从末尾的开始位置。**

**注意：截取的字符串，不包括尾标。**

```py
变量[头下标:尾下标]
```

实例：

```py
str = 'Hello, Python!'

print (str)          # 输出字符串
print (str[0:-1])    # 输出第一个到倒数第二个的所有字符
print (str[0])       # 输出字符串第一个字符
print (str[2:5])     # 输出从第三个开始到第五个的字符
print (str[2:])      # 输出从第三个开始的后的所有字符
print (str * 2)      # 输出字符串两次，也可以写成 print (2 * str)
print (str + "TEST") # 连接字符串
```

加号 + 是列表连接运算符，星号 * 是重复操作。

反斜杠 \ 转义特殊字符，如果你不想让反斜杠发生转义，可以在字符串前面添加一个 r （raw），表示原始字符串

```py
print('RHello, \nPython!')
RHello, 
Python!

print(r'Hello, \nPython!')
Hello, \nPython!
```

* ord() 函数返回单个字符的编码;
* chr() 函数把编码转成相应字符;

```py
s = 'A'
print(ord(s))
print(chr(65))

65
A
```

### 格式化

Python 使用 `%` 格式化字符串，常用占位符如下表所示：

| 占位符       | 描述      |
| :-----:| :----: | 
| %s  | 	格式化字符串| 
| %d  |   	格式化整数| 
| %f  | 	格式化浮点数| 

实例：

```py
print('Hello %s' % 'Python')
Hello Python
```

format() 方法格式化：

```py
print('{0} {1}'.format('Hello', 'Python'))
```

## List（列表）

列表是写在方括号 [] 之间、用逗号分隔开的元素列表。List中的元素是可以改变的。列表中元素的类型可以不相同，它支持数字，字符串甚至可以包含列表（所谓嵌套）。和字符串一样，列表同样可以被索引和截取，列表被截取后返回一个包含所需元素的新列表。

列表截取的语法：**索引值以 0 为开始值，-1 为从末尾的开始位置。**

```py
变量[头下标:尾下标]
```

加号 + 是列表连接运算符，星号 * 是重复操作。

* 更新

append() 向列表中添加新元素

```py
l = ['Hello', 'Python',1]

# 修改列表中第3个元素
l[1] = 2

# 向列表中添加新元素
l.append('Hello')

print('l[2] -->', l[1])
print('l -->', l)
```

输出：

```py
l[2] --> 2
l --> ['Hello', 'Python', 2, '666']
```

* 删除

del 删除列表中元素

```py
l = ['Hello', 'Python',1]
# 删除列表中第二个元素
del l[1]
```

* count()：统计列表中某个元素出现的次数
* index()：查找某个元素在列表中首次出现的位置（即索引）
* remove()：移除列表中某个值的首次匹配项
* sort()：对列表中元素进行排序
* copy()：复制列表

```py
l = ['d', 'b', 'a', 'f', 'd']
print("l.count('d') -->", l.count('d'))
print("l.index('d') -->", l.index('d'))
l.remove('d')
l.sort()
lc = l.copy()
```

## Tuple（元组）

元组（tuple）与列表类似，不同之处在于元组的元素不能修改，常用于保存不可修改的内容。元组写在小括号 () 里，元素之间用逗号隔开，元组中的元素类型也可以不相同，元组也可以使用+操作符进行拼接。

常用方法

* len()：计算元组中元素个数
* max()：返回元组中元素最大值
* min()：返回元组中元素最小值
* tuple()：将列表转换为元组

```py
tup1 = ()    # 空元组
tup2 = (20,) # 一个元素，需要在元素后添加逗号
tinytuple = (123, 'Python')
print (tinytuple * 2)     # 输出两次元组
print (tuple + tinytuple) # 连接元组

t = ('Hello','Python','666')
t = ('Hello','Python','666','777')

del t
print('len(t) -->', len(t))
print('max(t) -->', max(t))
print('min(t) -->', min(t))

l = ['d', 'b', 'a', 'f', 'd']
t = tuple(l)
```

## Set（集合）

集合（set）是由一个或数个形态各异的大小整体组成的，构成集合的事物或对象称作元素或是成员。

基本功能是进行成员关系测试和删除重复元素。

可以使用大括号 { } 或者 set() 函数创建集合，注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典。








