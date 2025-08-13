---
title: Python3学习-文件操作
date: 2021-04-16 15:12:44
tags:
- Python3
- Python
categories: Python3
---

Python open() 方法用于打开一个文件，并返回文件对象，在对文件进行处理过程都需要使用到这个函数，如果该文件无法被打开，会抛出 OSError。

注意：使用 open() 方法一定要保证关闭文件对象，即调用 close() 方法。

open() 函数常用形式是接收两个参数：文件名(file)和模式(mode)。

```py
open(file, mode='r')
```

完整的语法格式为：

```py
open(file, mode='r', buffering=-1, encoding=None, errors=None
, newline=None, closefd=True, opener=None)
```
<!--more-->

参数说明:

* file: 必需，文件路径（相对或者绝对路径）。
* mode: 可选，文件打开模式
* buffering: 设置缓冲
* encoding: 用于解码或编码文件的编码的名称，一般使用utf8
* errors: 报错级别，用于指定如何处理编码和解码错误（不能在二进制模式下使用）。
* newline: 区分换行符
* closefd: 传入的file参数类型
* opener: 设置自定义开启器，开启器的返回值必须是一个打开的文件描述符。

mode参数：默认为文本模式，如果要以二进制模式打开，加上 `b`。

* `t`	文本模式 (默认)。
* `x`	写模式，新建一个文件，如果该文件已存在则会报错。
* `b`	二进制模式。
* `+`	打开一个文件进行更新(可读可写)。
* `r`	以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
* `rb`	以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。一般用于非文本文件如图片等。
* `r+`	打开一个文件用于读写。文件指针将会放在文件的开头。
* `rb+`	以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。一般用于非文本文件如图片等。
* `w`	打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
* `wb`	以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。
* `w+`	打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
* `wb+`	以二进制格式打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。
* `a`	打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
* `ab`	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
* `a+`	打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
* `ab+`	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。

## file 对象常用的函数

* `file.close()` 关闭文件。关闭后文件不能再进行读写操作。
* `file.flush()` 刷新文件内部缓冲，直接把内部缓冲区的数据立刻写入文件, 而不是被动的等待输出缓冲区写入。
* `file.fileno()` 返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上。
* `file.isatty()` 如果文件连接到一个终端设备返回 True，否则返回 False。
* `file.read([size])` 从文件读取指定的字节数，如果未给定或为负则读取所有。
* `file.readline([size])` 读取整行，包括 "\n" 字符。
* `file.readlines([sizeint])` 读取所有行并返回列表，若给定sizeint>0，返回总和大约为sizeint字节的行, 实际读取值可能比 sizeint 较大, 因为需要填充缓冲区。
* `file.seek(offset[, whence])` 移动文件读取指针到指定位置
* `file.tell()` 返回文件当前位置，它是从文件开头开始算起的字节数。
* `file.truncate([size])` 从文件的首行首字符开始截断，截断文件为 size 个字符，无 size 表示从当前位置截断；截断之后后面的所有字符被删除，其中 windows 系统下的换行代表2个字符大小。
* `file.write(str)` 将字符串写入文件，返回的是写入的字符长度。
* `file.writelines(sequence)` 向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符。


### f.seek()

如果要改变文件当前的位置, 可以使用 `f.seek(offset, from_what)` 函数。

`from_what` 的值, 如果是 0 表示开头, 如果是 1 表示当前位置, 2 表示文件的结尾，例如：

`seek(x,0)` ： 从起始位置即文件首行首字符开始移动 x 个字符
`seek(x,1)` ： 表示从当前位置往后移动x个字符
`seek(-x,2)`：表示从文件的结尾往前移动x个字符

```py
>>> f = open('/tmp/foo.txt', 'rb+')
>>> f.write(b'0123456789abcdef')
16
>>> f.seek(5)     # 移动到文件的第六个字节
5
>>> f.read(1)
b'5'
>>> f.seek(-3, 2) # 移动到文件的倒数第三字节
13
>>> f.read(1)
b'd'
```

当处理一个文件对象时, 使用 `with` 关键字是非常好的方式。在结束后, 它会帮你正确的关闭文件。 而且写起来也比 try - finally 语句块要简短:
`with` 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的 "清理" 操作，释放资源，比如文件使用后自动关闭/线程中锁的自动获取和释放等。

```py
>>> with open('/tmp/foo.txt', 'r') as f:
...     read_data = f.read()
>>> f.closed
True
```

`write()`后，再进行读取，这样试读不到数据的，因为数据对象到达的地方为文件最后，读取是向后读的，因此，会读到空白，应该先把文件对象移到文件首位：

```py
f = open("forwrite.txt", "w+",encoding='utf-8')
f.write("7777777 6666")  # 此时文件对象在最后一行，如果读取，将读不到数据
s=f.tell()     # 返回文件对象当前位置
f.seek(0,0)    # 移动文件对象至第一个字符
str=f.read()
print(s,str,len(str))
```

**isatty() 和 truncate()**

```py
with open('test.txt', 'r+') as f:
    # 检测文件对象是否连接到终端设备
    print(f.isatty())
    # 截取两个字节
    f.truncate(2)
    print(f.read())
```

## pickle 模块

python的pickle模块实现了基本的数据序列和反序列化，从 `file` 中读取一个字符串，并将它重构为原来的 `python` 对象。

通过pickle模块的序列化操作我们能够将程序中运行的对象信息保存到文件中去，永久存储。

基本接口：

```py
pickle.dump(obj, file, [,protocol])
x = pickle.load(file)
```

实例1：

```py
#!/usr/bin/python3
import pickle

# 使用pickle模块将数据对象保存到文件
data1 = {'a': [1, 2.0, 3, 4+6j],
         'b': ('string', u'Unicode string'),
         'c': None}

selfref_list = [1, 2, 3]
selfref_list.append(selfref_list)

output = open('data.pkl', 'wb')

# Pickle dictionary using protocol 0.
pickle.dump(data1, output)

# Pickle the list using the highest protocol available.
pickle.dump(selfref_list, output, -1)

output.close()
```

实例2：

```py
#!/usr/bin/python3
import pprint, pickle

#使用pickle模块从文件中重构python对象
pkl_file = open('data.pkl', 'rb')

data1 = pickle.load(pkl_file)
pprint.pprint(data1)

data2 = pickle.load(pkl_file)
pprint.pprint(data2)

pkl_file.close()
```