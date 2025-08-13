---
title: Python3学习-OS模块
date: 2021-04-19 16:08:35
tags:
- Python3
- Python
categories: Python3
---

## 常用函数

### os.getcwd()

查看当前路径。

```py
import os
print(os.getcwd())
```

### os.listdir(path)

返回指定目录下包含的文件和目录名列表。

```py
import os
print(os.listdir('D:/'))
```

<!--more-->
### os.path.abspath(path)

返回路径 path 的绝对路径。

```py
import os
# 当前路径,相对路径
print(os.path.abspath('.'))
```

### os.path.split(path)

将路径 path 拆分为目录和文件两部分，返回结果为元组类型。

```py
import os
print(os.path.split('D:/tmp.txt'))
```

### os.path.join(path, *paths)

将一个或多个 path（文件或目录） 进行拼接。

```py
import os
print(os.path.join('D:/', 'tmp.txt'))
```

### os.path.getctime(path)

返回 path（文件或目录） 在系统中的创建时间。

```py
import os
import datetime
print(datetime.datetime.utcfromtimestamp(os.path.getctime('D:/tmp.txt')))
```

### os.path.getmtime(path)

返回 path（文件或目录）的最后修改时间。

```py
import os
import datetime
print(datetime.datetime.utcfromtimestamp(os.path.getmtime('D:/tmp.txt')))
```

### os.path.getatime(path)

返回 path（文件或目录）的最后访问时间。

```py
import os
import datetime
print(datetime.datetime.utcfromtimestamp(os.path.getatime('D:/tmp.txt')))
```

### os.path.exists(path)

判断 path（文件或目录）是否存在，存在返回 True，否则返回 False。

```py
import os
print(os.path.exists('D:/tmp.txt'))
```

### os.path.isdir(path)

判断 path 是否为目录。

```py
import os
print(os.path.isdir('D:/'))
```

### os.path.isfile(path)

判断 path 是否为文件。

```py
import os
print(os.path.isfile('D:/tmp.txt'))
```

### os.path.getsize(path)

返回 path 的大小，以字节为单位，若 path 是目录则返回 0。

```py
import os
print(os.path.getsize('D:/tmp.txt'))
print(os.path.getsize('D:/work'))
```

### os.mkdir()

创建一个目录。

```py
import os
os.mkdir('D:/test')
```

### os.makedirs()

创建多级目录。

```py
import os
os.makedirs('D:/test1/test2')
```

### os.chdir(path)

将当前工作目录更改为 path。

```py
import os
print(os.getcwd())
os.chdir('/test')
print(os.getcwd())
```

### os.system(command)

调用 shell 脚本。

```py
import os
print(os.system('ping www.baidu.com'))
```

### os.rename(src, dst)

重命名文件或目录

```py
#!/usr/bin/python3

import os, sys

# 列出目录
print ("目录为: %s"%os.listdir(os.getcwd()))

# 重命名
os.rename("test","test2")

print ("重命名成功。")

# 列出重命名后的目录
print ("目录为: %s" %os.listdir(os.getcwd()))
```

输出：

```py
目录为:
[  'a1.txt','resume.doc','a3.py','test' ]
重命名成功。
[  'a1.txt','resume.doc','a3.py','test2' ]
```

### os.replace(old, new)

将文件重命名

首先创建两个文件：
1.txt 内容是1
2.txt 内容是2

```py
import OS
os.replace('1.txt', '2.txt')
```

执行后发现只剩下一个：2.txt，但内容是 1。

所以 os.replace(file1,file2) 这个函数相当于用 file2 给 file1 重命名，并删除 file2。


参考：

[Python3 OS 文件/目录方法](https://www.runoob.com/python3/python3-os-file-methods.html)

[os --- 多种操作系统接口](https://docs.python.org/zh-cn/3/library/os.html?highlight=os#)

