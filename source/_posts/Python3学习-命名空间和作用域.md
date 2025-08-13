---
title: Python3学习-命名空间和作用域
date: 2021-04-22 10:59:42
tags:
- Python3
- Python
categories: Python3
---

## 命名空间

* **内置名称（built-in names）**， Python 语言内置的名称，比如函数名 abs、char 和异常名称 BaseException、Exception 等等。
* **全局名称（global names）**，模块中定义的名称，记录了模块的变量，包括函数、类、其它导入的模块、模块级的变量和常量。
* **局部名称（local names）**，函数中定义的名称，记录了函数的变量，包括函数的参数和局部定义的变量。（类中定义的也是）

![types_namespace-1.png](/img/types_namespace-1.png)
<!--more-->
**查找顺序为：局部的命名空间去 -> 全局命名空间 -> 内置命名空间。**

命名空间的生命周期取决于对象的作用域，如果对象执行完成，则该命名空间的生命周期就结束。相同的对象名称可以存在于多个命名空间中。

因此，无法从外部命名空间访问内部命名空间的对象。

三种命名空间的生命周期

* **内置**：在 Python 解释器启动时创建，退出时销毁。
* **全局**：在模块定义被读入时创建，在 Python 解释器退出时销毁。
* **局部**：对于类，在 Python 解释器读到类定义时创建，类定义结束后销毁；对于函数，在函数被调用时创建，函数执行完成或出现未捕获的异常时销毁。

```py
# var1 是全局名称
var1 = 5
def some_func():
 
    # var2 是局部名称
    var2 = 6
    def some_inner_func():
 
        # var3 是内嵌的局部名称
        var3 = 7
```

## 作用域

Python 中只有模块（module），类（class）以及函数（def、lambda）才会引入新的作用域，其它的代码块（如 `if/elif/else/`、`try/except`、`for/while` 等）是不会引入新的作用域的，也就是说这些语句内定义的变量，外部也可以访问，如下代码：

```py
>>> if True:
...  msg = 'I am from Runoob'
... 
>>> msg
'I am from Runoob'
>>> 
```

实例中 msg 变量定义在 if 语句块中，但外部还是可以访问的。

如果将 msg 定义在函数中，则它就是局部变量，外部不能访问：

```py
>>> def test():
...     msg_inner = 'I am from Runoob'
... 
>>> msg_inner
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'msg_inner' is not defined
>>> 
```

说明了 `msg_inner` 未定义，无法使用，因为它是局部变量，只有在函数内可以使用。

Python 四种作用域：

* **L（Local）**：最内层，包含局部变量，比如一个函数/方法内部。
* **E（Enclosing）**：包含了非局部(non-local)也非全局(non-global)的变量。比如两个嵌套函数，一个函数（或类） A 里面又包含了一个函数 B ，那么对于 B 中的名称来说 A 中的作用域就为 `nonlocal`。
* **G（Global）**：当前脚本的最外层，比如当前模块的全局变量。
* **B（Built-in）**： 包含了内建的变量/关键字等。，最后被搜索

规则顺序：**L –> E –> G –>gt; B**。

在局部找不到，便会去局部外的局部找（例如闭包），再找不到就会去全局找，再者去内置中找。

![1418490-20180906153626089-1835444372.png](/img/1418490-20180906153626089-1835444372.png)

```py
g_count = 0  # 全局作用域
def outer():
    o_count = 1  # 闭包函数外的函数中
    def inner():
        i_count = 2  # 局部作用域
```

内置作用域是通过一个名为 `builtin` 的标准模块来实现的，但是这个变量名自身并没有放入内置作用域内，所以必须导入这个文件才能够使用它。

查看预定义变量:

```py
>>> import builtins
>>> dir(builtins)
```

### global 和 nonlocal关键字

* 全局变量：定义在函数外部的变量。
* 局部变量：定义在函数内部的变量。

调用函数时，所有在函数内声明的变量名称都将被加入到作用域中。

**global** ：声明为全局变量
**nonlocal** ：包含了非局部(non-local)也非全局(non-global)的变量。比如两个嵌套函数，一个函数（或类） A 里面又包含了一个函数 B ，那么对于 B 中的名称来说 A 中的作用域就为 `nonlocal`。

`global` 实例：

```py
#!/usr/bin/python3
 
num = 1
def fun1():
    global num  # 需要使用 global 关键字声明
    print(num) 
    num = 123
    print(num)
fun1()
print(num)
```

输出：

```py
1
123
123
```

`nonlocal` 实例：

```py
#!/usr/bin/python3
 
def outer():
    num = 10
    def inner():
        nonlocal num   # nonlocal关键字声明
        num = 100
        print(num)
    inner()
    print(num)
outer()
```

输出：

```py
100
100
```

参考：

[Python3 命名空间和作用域](https://www.runoob.com/python3/python3-namespace-scope.html)