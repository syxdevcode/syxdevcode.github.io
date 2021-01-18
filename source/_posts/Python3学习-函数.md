---
title: Python3学习-函数
date: 2021-01-16 15:22:55
tags:
- Python3
- Python
categories: Python3
---

## 自定义函数

自定义函数规则：

* 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号 ()。
* 任何传入参数和自变量必须放在圆括号中间，圆括号之间可以用于定义参数。
* 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。
* 函数内容以冒号 : 起始，并且缩进。
* return [表达式] 结束函数，选择性地返回一个值给调用方，不带表达式的 return 相当于返回 None。

```py
def 函数名(参数):
	函数体
	return 返回值

# 空函数
def 函数名():
	pass

# 不确定参数的个数,使用不定长参数，在参数名前加 * 进行声明
def 函数名(*参数名):
	函数体

# lambda 定义匿名函数
lambda 参数 : 表达式
```

示例：

```py
# 空函数
def test_empty():
    pass

# 无返回值
def testprint(name):
    print('Hello', name)

# 有返回值
def test_sum(x, y):
    s = x + y
    print('s-->', s)
    return s
    
# 不定长参数
def test_variable(*params):
    for p in params:
        print(p)

# 匿名函数
test_sub = lambda x, y: x - y
```

## 函数调用

```py
# 定义函数
def testprint( str ):
   # 打印任何传入的字符串
   print (str)
   return

# 调用函数
testprint("调用自定义函数!")
testprint("再次调用同一函数")
```

## 参数传递

在 python 中，类型属于对象，变量是没有类型的

**可更改(mutable)与不可更改(immutable)对象**

在 python 中，strings, tuples, 和 numbers 是不可更改的对象，而 list,dict 等则是可以修改的对象。

* 不可变类型：变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变 a 的值，相当于新生成了 a。
* 可变类型：变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了。

python 函数的参数传递：

* 不可变类型：类似 C++ 的值传递，如 整数、字符串、元组。如 fun(a)，传递的只是 a 的值，没有影响 a 对象本身。如果在 fun(a)）内部修改 a 的值，则是新生成来一个 a。
* 可变类型：类似 C++ 的引用传递，如 列表，字典。如 fun(la)，则是将 la 真正的传过去，修改后 fun 外部的 la 也会受影响

**函数调用可以使用关键字参数来确定传入的参数值，并且不需要使用指定顺序：**

```py
def testprint(str):
   "打印任何传入的字符串"
   print (str)
   return
 
#调用testprint函数
testprint( str = "Hello")

# 不需要使用指定顺序
def testprint( name, age):
   "打印任何传入的字符串"
   print ("名字: ", name)
   print ("年龄: ", age)
   return
 
#调用testprint函数
testprint( age=20, name="jun")
```

### 默认参数:

```py
def testprint(name, age = 35):
   "打印任何传入的字符串"
   print ("名字: ", name)
   print ("年龄: ", age)
   return

# 调用
testprint(name="jun" )
```

### 不定长参数

如果在函数调用时没有指定参数，它就是一个空元组,可以不向函数传递未命名的变量。

加星号 `*` 的参数会以元组(tuple)的形式导入:

```py
def testprint(arg1, *vartuple ):
   "打印任何传入的参数"
   print ("输出: ")
   print (arg1)
   print (vartuple)

# 调用testprint 函数
testprint(10)
testprint(70, 60, 50)

# 声明函数时，参数中星号 * 可以单独出现，
# 如果单独出现星号 * 后的参数必须用关键字传入。
# 例如:
def f(a,b,*,c):
    return a+b+c

f(1,2,3)   # 报错
f(1,2,c=3) # 正常
```

### 匿名函数

语法：

```py
lambda [arg1 [,arg2,.....argn]]:expression
```

示例：

```py
sum = lambda arg1, arg2: arg1 + arg2
 
# 调用sum函数
print ("相加后的值为 : ", sum( 10, 20 ))
print ("相加后的值为 : ", sum( 20, 20 ))
```

### 强制位置参数

Python3.8 新增了一个函数形参语法 / 用来指明函数形参必须使用指定位置参数，不能使用关键字参数的形式。

在以下的例子中，形参 a 和 b 必须使用指定位置参数，c 或 d 可以是位置形参或关键字形参，而 e 或 f 要求为关键字形参:

```py
def f(a, b, /, c, d, *, e, f):
    print(a, b, c, d, e, f)


## 正确使用方法:
f(10, 20, 30, d=40, e=50, f=60)

# 错误使用方法:
f(10, b=20, c=30, d=40, e=50, f=60)   # b 不能使用关键字参数的形式
f(10, 20, 30, 40, 50, f=60)           # e 必须使用关键字参数的形式
```