---
title: Python3学习-函数装饰器
date: 2021-04-22 15:26:17
tags:
- Python3
- Python
categories: Python3
---

装饰器(Decorators)是 Python 的一个重要部分。简单地说：装饰器是修改其他函数的功能的函数。

## 一切皆对象

首先我们来理解下 Python 中的函数:

```PY
def hi(name="Python"):
    return "hi " + name
 
print(hi())
# output: 'hi Python'
 
# 可以将一个函数赋值给一个变量
# # 注意没有 使用小括号，并不是在调用hi函数
greet = hi
 
print(greet())
# output: 'hi Python'
 
# 删掉旧的hi函数
del hi
print(hi())
#outputs: NameError
 
print(greet())
#outputs: 'hi Python'
```

## 在函数中定义函数

在函数中定义另外的函数。

```py
def hi(name="Python"):
    print("start hi() function")
 
    def greet():
        return "greet() function"
 
    def welcome():
        return "welcome() function"
 
    print(greet())
    print(welcome())
    print("end hi() function")
 
hi()
# output: start hi() function
#         greet() function
#         welcome() function
#         end hi() function

greet()
#   greet()
# NameError: name 'greet' is not defined
```

## 从函数中返回函数

```py
def hi(name="Python"):
    def greet():
        return "greet() function"
 
    def welcome():
        return "welcome() function"
 
    if name == "Python":
        return greet
    else:
        return welcome
 
a = hi()
print(a)
#outputs: <function hi.<locals>.greet at 0x000001FA8EC3F790>
# `a`现在指向到hi()函数中的greet()函数
 
print(a())
#outputs: greet() function

print(hi('we')())
# outputs: welcome() function
```

## 将函数作为参数传给另一个函数

```py
def hi():
    return "hi Python!"
 
def doSomethingBeforeHi(func):
    print("before executing hi()")
    print(func())
 
doSomethingBeforeHi(hi)
#outputs:before executing hi()
#        hi Python!
```

## 装饰器

`@` 作用：将函数传入装饰器函数。

不使用 `@`:

```py
def a_new_decorator(a_func):
 
    def wrapTheFunction():
        print("before executing a_func()")
 
        a_func()
 
        print("after executing a_func()")
 
    return wrapTheFunction
 
def a_function_requiring_decoration():
    print("executing a_function_requiring_decoration()")
 
a_function_requiring_decoration()
#outputs: "executing a_function_requiring_decoration()"

# 装饰器
a_function_requiring_decoration = a_new_decorator(a_function_requiring_decoration)
 
a_function_requiring_decoration()
#outputs:before executing a_func()
#        executing a_function_requiring_decoration()
#        after executing a_func()
```

使用 `@`：

```py
def a_new_decorator(a_func):
 
    def wrapTheFunction():
        print("before executing a_func()")
 
        a_func()
 
        print("after executing a_func()")
 
    return wrapTheFunction

@a_new_decorator
def a_function_requiring_decoration():
    print("executing a_function_requiring_decoration()")
 
a_function_requiring_decoration()
#outputs:before executing a_func()
#        executing a_function_requiring_decoration()
#        after executing a_func()
```

通过代码可以发现，`@a_new_decorator` 的作用相当于：

```py
a_function_requiring_decoration = a_new_decorator(a_function_requiring_decoration)
```

## 解决装饰器重写问题

```py
print(a_function_requiring_decoration.__name__)
# Output: wrapTheFunction
```

原因就是装饰器重写了我们函数的名字和注释文档(docstring)。

解决方法：引入 `functools.wraps`

```py
from functools import wraps

def a_new_decorator(a_func):
    @wraps(a_func)
    def wrapTheFunction():
        print("before executing a_func()")
 
        a_func()
 
        print("after executing a_func()")
 
    return wrapTheFunction

@a_new_decorator
def a_function_requiring_decoration():
    print("executing a_function_requiring_decoration()")

print(a_function_requiring_decoration.__name__)
# Output: a_function_requiring_decoration
```

`@wraps` 接受一个函数来进行装饰，并加入了复制函数名称、注释文档、参数列表等等的功能。这样可以在装饰器里面访问在装饰之前的函数的属性。

```py
from functools import wraps
def decorator_name(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if not can_run:
            return "Function will not run"
        return f(*args, **kwargs)
    return decorated
 
@decorator_name
def func():
    return("Function is running")
 
can_run = True
print(func())
# Output: Function is running
 
can_run = False
print(func())
# Output: Function will not run
```

## 带参数的装饰器

```py
from functools import wraps
 
def logit(logfile='out.log'):
    def logging_decorator(func):
        @wraps(func)
        def wrapped_function(*args, **kwargs):
            log_string = func.__name__ + " was called"
            print(log_string)
            # 打开logfile，并写入内容
            with open(logfile, 'a') as opened_file:
                # 现在将日志打到指定的logfile
                opened_file.write(log_string + '\n')
            return func(*args, **kwargs)
        return wrapped_function
    return logging_decorator
 
@logit()
def myfunc1():
    pass
 
myfunc1()
# Output: myfunc1 was called
# 现在一个叫做 out.log 的文件出现了，里面的内容就是上面的字符串
 
@logit(logfile='func2.log')
def myfunc2():
    pass
 
myfunc2()
# Output: myfunc2 was called
# 现在一个叫做 func2.log 的文件出现了，里面的内容就是上面的字符串
```

## 装饰器类

使用类装饰器主要依靠类的 `__call__` 方法，当使用 `@` 形式将装饰器附加到函数上时，就会调用此方法。

```py
from functools import wraps
 
class logit(object):
    def __init__(self, logfile='out.log'):
        self.logfile = logfile
 
    def __call__(self, func):
        @wraps(func)
        def wrapped_function(*args, **kwargs):
            log_string = func.__name__ + " was called"
            print(log_string)
            # 打开logfile并写入
            with open(self.logfile, 'a') as opened_file:
                # 现在将日志打到指定的文件
                opened_file.write(log_string + '\n')
            # 现在，发送一个通知
            self.notify()
            return func(*args, **kwargs)
        return wrapped_function
 
    def notify(self):
        # logit只打日志，不做别的
        pass

@logit()
def myfunc1():
    pass

myfunc1()
# outputs: myfunc1 was called
```

给 `logit` 创建子类，模拟添加 `email` 的功能，`@email_logit `将会和 `@logit` 产生同样的效果，但是在打日志的基础上，还会多发送一封邮件给管理员。

```py
from python1 import logit

class email_logit(logit):
    '''
    一个logit的实现版本，可以在函数调用时发送email给管理员
    '''
    def __init__(self, email='admin@myproject.com', *args, **kwargs):
        self.email = email
        super(email_logit, self).__init__(*args, **kwargs)
 
    def notify(self):
        # 发送一封email到self.email
        # 这里就不做实现了
        pass
```

## 装饰器顺序

一个函数还可以同时定义多个装饰器，比如：

```py
@a
@b
@c
def f ():
    pass
```

它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于

```py
f = a(b(c(f)))
```

## 装饰器使用场景

### 授权(Authorization)

```py
from functools import wraps
 
def requires_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = request.authorization
        if not auth or not check_auth(auth.username, auth.password):
            authenticate()
        return f(*args, **kwargs)
    return decorated
```

### 日志(Logging)

```py
from functools import wraps
 
def logit(func):
    @wraps(func)
    def with_logging(*args, **kwargs):
        print(func.__name__ + " was called")
        return func(*args, **kwargs)
    return with_logging
 
@logit
def addition_func(x):
   """Do some math."""
   return x + x

result = addition_func(4)
# Output: addition_func was called
```

参考：

[Python 函数装饰器](https://www.runoob.com/w3cnote/python-func-decorators.html)