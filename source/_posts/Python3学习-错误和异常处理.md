---
title: Python3学习-错误和异常处理
date: 2021-04-20 09:10:04
tags:
- Python3
- Python
categories: Python3
---

Python 有两种错误：语法错误和异常。

Python assert（断言）用于判断一个表达式，在表达式条件为 false 的时候触发异常。

![assets-py.webp](/img/assets-py.webp)

<!--more-->
## 异常处理

### try/except

![try_except.png](/img/try_except.png)

```py
while True:
    try:
        x = int(input("请输入一个数字: "))
        break
    except ValueError:
        print("您输入的不是数字，请再次尝试输入！")
```

一个except子句可以同时处理多个异常，这些异常将被放在一个括号里成为一个元组，例如:

```py
except (RuntimeError, TypeError, NameError):
    pass
```

### try/except...else

![try_except_else.png](/img/try_except_else.png)

### try-finally 语句

![try_except_else_finally.png](/img/try_except_else_finally.png)

## 抛出异常

Python 使用 raise 语句抛出一个指定的异常。

raise语法格式如下：

```py
raise [Exception [, args [, traceback]]]
```

raise 唯一的一个参数指定了要被抛出的异常。它必须是一个异常的实例或者是异常的类（也就是 Exception 的子类）。

如果你只想知道这是否抛出了一个异常，并不想去处理它，那么一个简单的 raise 语句就可以再次把它抛出。

```py
try:
        raise NameError('HiThere')
    except NameError:
        print('An exception flew by!')
        raise
```

## 用户自定义异常

通过创建一个新的异常类来拥有自己的异常。异常类继承自 Exception 类，可以直接继承，或者间接继承，大多数的异常的名字都以"Error"结尾，跟标准的异常命名一样。例如:

```py
>>> class MyError(Exception):
        def __init__(self, value):
            self.value = value
        def __str__(self):
            return repr(self.value)
   
>>> try:
        raise MyError(2*2)
    except MyError as e:
        print('My exception occurred, value:', e.value)
```

在这个例子中，类 `Exception` 默认的 `__init__()` 被覆盖。

当创建一个模块有可能抛出多种不同的异常时，一种通常的做法是为这个包建立一个基础异常类，然后基于这个基础类为不同的错误情况创建不同的子类:

```py
class Error(Exception):
    """Base class for exceptions in this module."""
    pass

class InputError(Error):
    """Exception raised for errors in the input.

    Attributes:
        expression -- input expression in which the error occurred
        message -- explanation of the error
    """

    def __init__(self, expression, message):
        self.expression = expression
        self.message = message

class TransitionError(Error):
    """Raised when an operation attempts a state transition that's not
    allowed.

    Attributes:
        previous -- state at beginning of transition
        next -- attempted new state
        message -- explanation of why the specific transition is not allowed
    """

    def __init__(self, previous, next, message):
        self.previous = previous
        self.next = next
        self.message = message
```

## 定义清理行为

如果一个异常在 try 子句里（或者在 except 和 else 子句里）被抛出，而又没有任何的 except 把它截住，那么这个异常会在 finally 子句执行后被抛出。

```py
>>> try:
...     raise KeyboardInterrupt
... finally:
...     print('Goodbye, world!')
...
Goodbye, world!
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
KeyboardInterrupt
```

## 预定义的清理行为

关键词 with 语句就可以保证诸如文件之类的对象在使用完之后一定会正确的执行他的清理方法:

```py
with open("myfile.txt") as f:
    for line in f:
        print(line, end="")
```

## 捕获所有异常

```py
try:
   ...
except Exception as e:
   ...
   log('Reason:', e)       # Important!
```

这个将会捕获除了 `SystemExit` 、 `KeyboardInterrupt` 和 `GeneratorExit` 之外的所有异常。 如果你还想捕获这三个异常，将 `Exception` 改成 `BaseException` 即可。

## 内置异常

BaseException为所有异常的基类，其下面分为：SystemExit、KeyboardInterrupt、GeneratorExit、Exception 四类异常，Exception 为所有非系统退出类异常的基类，Python 提倡继承 Exception 或其子类派生新的异常；Exception 下包含我们常见的多种异常如：MemoryError（内存溢出）、BlockingIOError（IO异常）、SyntaxError（语法错误异常）… 详细说明可以查看下面列表：

```py
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

* BaseException	所有异常的基类
* SystemExit	解释器请求退出
* KeyboardInterrupt	用户中断执行(通常是输入^C)
* Exception	常规错误的基类
* StopIteration	迭代器没有更多的值
* GeneratorExit	生成器(generator)发生异常来通知退出
* StandardError	所有的内建标准异常的基类
* ArithmeticError	所有数值计算错误的基类
* FloatingPointError	浮点计算错误
* OverflowError	数值运算超出最大限制
* ZeroDivisionError	除(或取模)零 (所有数据类型)
* AssertionError	断言语句失败
* AttributeError	对象没有这个属性
* EOFError	没有内建输入,到达EOF 标记
* EnvironmentError	操作系统错误的基类
* IOError	输入/输出操作失败
* OSError	操作系统错误
* WindowsError	系统调用失败
* ImportError	导入模块/对象失败
* LookupError	无效数据查询的基类
* IndexError	序列中没有此索引(index)
* KeyError	映射中没有这个键
* MemoryError	内存溢出错误(对于Python 解释器不是致命的)
* NameError	未声明/初始化对象 (没有属性)
* UnboundLocalError	访问未初始化的本地变量
* ReferenceError	弱引用(Weak reference)试图访问已经垃圾回收了的对象
* RuntimeError	一般的运行时错误
* NotImplementedError	尚未实现的方法
* SyntaxError	Python 语法错误
* IndentationError	缩进错误
* TabError	Tab 和空格混用
* SystemError	一般的解释器系统错误
* TypeError	对类型无效的操作
* ValueError	传入无效的参数
* UnicodeError	Unicode 相关的错误
* UnicodeDecodeError	Unicode 解码时的错误
* UnicodeEncodeError	Unicode 编码时错误
* UnicodeTranslateError	Unicode 转换时错误
* Warning	警告的基类
* DeprecationWarning	关于被弃用的特征的警告
* FutureWarning	关于构造将来语义会有改变的警告
* OverflowWarning	旧的关于自动提升为长整型(long)的警告
* PendingDeprecationWarning	关于特性将会被废弃的警告
* RuntimeWarning	可疑的运行时行为(runtime behavior)的警告
* SyntaxWarning	可疑的语法的警告
* UserWarning	用户代码生成的警告

参考：

[内置异常](https://docs.python.org/zh-cn/3/library/exceptions.html)

[Python3 错误和异常](https://www.runoob.com/python3/python3-errors-execptions.html)