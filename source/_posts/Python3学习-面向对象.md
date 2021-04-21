---
title: Python3学习-面向对象
date: 2021-04-21 09:08:42
tags:
- Python3
- Python
categories: Python3
---

## 面向对象三大特性

封装：隐藏对象的属性和实现细节，仅对外提供公共访问方式，提高复用性和安全性。

继承：一个类继承一个基类便可拥有基类的属性和方法，可提高代码的复用性。

多态：父类定义的引用变量可以指向子类的实例对象，提高了程序的拓展性。

## Python面向对象简介

* **类(Class)**: 用来描述具有相同的属性和方法的对象的集合。它定义了该集合中每个对象所共有的属性和方法。对象是类的实例。
* **方法**：类中定义的函数。
* **类变量**：类变量在整个实例化的对象中是公用的。类变量定义在类中且在函数体之外。类变量通常不作为实例变量使用。
* **数据成员**：类变量或者实例变量用于处理类及其实例对象的相关的数据。
* **方法重写**：如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
* **局部变量**：定义在方法中的变量，只作用于当前实例的类。
* **实例变量**：在类的声明中，属性是用变量来表示的，这种变量就称为实例变量，实例变量就是一个用 `self` 修饰的变量。
* **继承**：即一个派生类（derived class）继承基类（base class）的字段和方法。继承也允许把一个派生类的对象作为一个基类对象对待。例如，有这样一个设计：一个Dog类型的对象派生自Animal类，这是模拟 `是一个（is-a）` 关系。
* **实例化**：创建一个类的实例，类的具体对象。
* **对象**：通过类定义的数据结构实例。对象包括两个数据成员（类变量和实例变量）和方法。

## 类对象

Python 中类的定义使用 `class` 关键字，语法如下所示：

```py
class 类名:
	属性
	...
	方法
	...
```

```py
#!/usr/bin/python3
 
class MyClass:
    """一个简单的类实例"""
    i = 12345
    def f(self):
        return 'hello world'
 
# 实例化类
x = MyClass()
 
# 访问类的属性和方法
print("MyClass 类的属性 i 为：", x.i)
print("MyClass 类的方法 f 输出为：", x.f())
```

执行以上程序输出结果为：

```py
MyClass 类的属性 i 为： 12345
MyClass 类的方法 f 输出为： hello world
```

`__init__()` 是类的构造方法，该方法在类实例化时会自动调用：

```py
def __init__(self):
    self.data = []
```

类定义了 `__init__()` 方法，类的实例化操作会自动调用 `__init__()` 方法。`__init__()` 方法可以有参数，参数通过 `__init__()` 传递到类的实例化操作上。

```py
#!/usr/bin/python3
 
class Complex:
    def __init__(self, realpart, imagpart):
        self.r = realpart
        self.i = imagpart
x = Complex(3.0, -4.5)
print(x.r, x.i)   # 输出结果：3.0 -4.5
```

## 类方法,普通方法和静态方法

```py
#!/usr/bin/python3
 
class ClassTest:

    # 类属性
    txt=" hi python" 

    # 构造方法
    def __init__(self,t1):
        self.txt=t1
        print(self.txt)
    
    # 普通方法，或实例方法
    def normalMethod(self,name):
        print(self.txt)

    # 类方法，可以访问到 类属性
    @classmethod
    def classMethod(cls,txt):
        print(cls.txt)

    # 静态方法，不能访问类属性
    @staticmethod
    def staticMethod(txt):
        print(txt)

    # self 代表的是类的实例，代表当前对象的地址，self.class 则指向类。
    def prt(self):
        print(self)
        print(self.__class__)
```

调用：

```py
x = ClassTest("init Method")
x.txt="new python"

print(x.normalMethod('this is new string'))
"""
输出：
init Method
new python        
None
"""

print(ClassTest.normalMethod('this is new string')) 
"""
报错：
Traceback (most recent call last):
  File "e:\git\pythontest\python1.py", line 36, in <module>
    print(ClassTest.normalMethod('this is new string'))
TypeError: normalMethod() missing 1 required positional argument: 'name'
"""

print(ClassTest.classMethod('this is new string'))
"""
输出：
init Method
 hi python
None
"""

print(ClassTest.staticMethod('this is new string'))
"""
输出：
init Method
this is new string
None
"""

print(x.prt())
"""
输出：
init Method
<__main__.ClassTest object at 0x0000022838D03F40>
<class '__main__.ClassTest'>
None
"""
```

调用结果总结：

**实例方法（普通方法）**-->随着实例属性的改变而改变

**类方法（类调用或实例调用）**--> 都是类属性的值，不随实例属性的变化而变化

**静态方法** -->不可以访问类属性，故直接输出传入方法的值

![1370165-20180513003736716-1840390911.png](/img/1370165-20180513003736716-1840390911.png)

## 继承

语法：

```py
class DerivedClassName(BaseClassName):
    <statement-1>
    .
    .
    .
    <statement-N>
```

子类（派生类 DerivedClassName）会继承父类（基类 BaseClassName）的属性和方法。

BaseClassName（实例中的基类名）必须与派生类定义在一个作用域内。

```py
#!/usr/bin/python3
 
#类定义
class people:
    #定义基本属性
    name = ''
    age = 0
    #定义私有属性,私有属性在类外部无法直接进行访问
    __weight = 0
    #定义构造方法
    def __init__(self,n,a,w):
        self.name = n
        self.age = a
        self.__weight = w
    def speak(self):
        print("%s 说: 我 %d 岁。" %(self.name,self.age))
 
#单继承示例
class student(people):
    grade = ''
    def __init__(self,n,a,w,g):
        #调用父类的构函
        people.__init__(self,n,a,w)
        self.grade = g
    #覆写父类的方法
    def speak(self):
        print("%s 说: 我 %d 岁了，我在读 %d 年级"%(self.name,self.age,self.grade))
 
 
 
s = student('ken',10,60,3)
s.speak()
```

输出结果为：

```py
ken 说: 我 10 岁了，我在读 3 年级
```

## 多继承

多继承的类定义:

```py
class DerivedClassName(Base1, Base2, Base3):
    <statement-1>
    .
    .
    .
    <statement-N>
```

若是父类中有相同的方法名，而在子类使用时未指定，python从左至右搜索 即方法在子类中未找到时，从左到右查找父类中是否包含方法。

```py
#!/usr/bin/python3
 
#类定义
class people:
    #定义基本属性
    name = ''
    age = 0
    #定义私有属性,私有属性在类外部无法直接进行访问
    __weight = 0
    #定义构造方法
    def __init__(self,n,a,w):
        self.name = n
        self.age = a
        self.__weight = w
    def speak(self):
        print("%s 说: 我 %d 岁。" %(self.name,self.age))
 
#单继承示例
class student(people):
    grade = ''
    def __init__(self,n,a,w,g):
        #调用父类的构函
        people.__init__(self,n,a,w)
        self.grade = g
    #覆写父类的方法
    def speak(self):
        print("%s 说: 我 %d 岁了，我在读 %d 年级"%(self.name,self.age,self.grade))
 
#另一个类，多重继承之前的准备
class speaker():
    topic = ''
    name = ''
    def __init__(self,n,t):
        self.name = n
        self.topic = t
    def speak(self):
        print("我叫 %s，我是一个演说家，我演讲的主题是 %s"%(self.name,self.topic))
 
#多重继承
class sample(speaker,student):
    a =''
    def __init__(self,n,a,w,g,t):
        student.__init__(self,n,a,w,g)
        speaker.__init__(self,n,t)
 
test = sample("Tim",25,80,4,"Python")
test.speak()   #方法名同，默认调用的是在括号中排前地父类的方法
```

执行以上程序输出结果为：

```py
我叫 Tim，我是一个演说家，我演讲的主题是 Python
```

## 方法重写

```py
#!/usr/bin/python3
 
class Parent:        # 定义父类
   def myMethod(self):
      print ('调用父类方法')
 
class Child(Parent): # 定义子类
   def myMethod(self):
      print ('调用子类方法')
 
c = Child()          # 子类实例
c.myMethod()         # 子类调用重写方法
super(Child,c).myMethod() #用子类对象调用父类已被覆盖的方法
```

super() 函数是用于调用父类(超类)的一个方法。

输出结果为：

```py
调用子类方法
调用父类方法
```

## 类属性与方法

### 类的私有属性

`__private_attrs`：两个下划线开头，声明该属性为私有，不能在类的外部被使用或直接访问。在类内部的方法中使用时 `self.__private_attrs`。

### 类的方法

默认有个 `cls` 参数，可以被类和对象调用，需要加上 `@classmethod` 装饰器。

### 类的私有方法

`__private_method`：两个下划线开头，声明该方法为私有方法，只能在类的内部调用 ，不能在类的外部调用。在类内部调用 `self.__private_methods`。

### 实例

```py
#!/usr/bin/python3
 
class JustCounter:
    __secretCount = 0  # 私有变量
    publicCount = 0    # 公开变量
 
    def count(self):
        self.__secretCount += 1
        self.publicCount += 1
        print (self.__secretCount)
    def __foo(self):          # 私有方法
        print('这是私有方法')
 
counter = JustCounter()
counter.count()
counter.count()
print (counter.publicCount)
print (counter.__secretCount)  
# 报错，实例不能访问私有变量
#     print (counter.__secretCount)  # 报错，实例不能访问私有变量
# AttributeError: 'JustCounter' object has no attribute '__secretCount'

print(counter.__foo())      
# 报错
# AttributeError: 'JustCounter' object has no attribute '__foo'
```

### 类的专有方法：

* `__init__` : 构造函数，在生成对象时调用
* `__del__` : 析构函数，释放对象时使用
* `__repr__` : 打印，转换
* `__setitem__` : 按照索引赋值
* `__getitem__`: 按照索引获取值
* `__len__`: 获得长度
* `__cmp__`: 比较运算
* `__call__`: 函数调用
* `__add__`: 加运算
* `__sub__`: 减运算
* `__mul__`: 乘运算
* `__truediv__`: 除运算
* `__mod__`: 求余运算
* `__pow__`: 乘方

反向运算符重载：

* `__radd__`: 加运算
* `__rsub__`: 减运算
* `__rmul__`: 乘运算
* `__rdiv__`: 除运算
* `__rmod__`: 求余运算
* `__rpow__`: 乘方

复合重载运算符：

* `__iadd__`: 加运算
* `__isub__`: 减运算
* `__imul__`: 乘运算
* `__idiv__`: 除运算
* `__imod__`: 求余运算
* `__ipow__`: 乘方

### 运算符重载

```py
#!/usr/bin/python3
 
class Vector:
   def __init__(self, a, b):
      self.a = a
      self.b = b
 
   def __str__(self):
      return 'Vector (%d, %d)' % (self.a, self.b)
   
   def __add__(self,other):
      return Vector(self.a + other.a, self.b + other.b)
 
v1 = Vector(2,10)
v2 = Vector(5,-2)
print (v1 + v2)
```

结果：

```py
Vector(7,8)
```








参考：

[9. 类](https://docs.python.org/zh-cn/3/tutorial/classes.html)

[一文详解python的类方法，普通方法和静态方法](https://www.cnblogs.com/jayliu/p/9030725.html)

