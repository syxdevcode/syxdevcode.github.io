---
title: linux chmod命令
date: 2018-05-11 08:56:22
tags:
- Linux
- Linux基础命令
categories:
- Linux基础命令
---
# linux chmod命令

chmod命令用于改变linux系统文件或目录的访问权限。用它控制文件或目录的访问权限。
该命令有两种用法:

** 一种是包含字母和操作符表达式的文字设定法；**
** 一种是包含数字的数字设定法。**

Linux系统中的每个文件和目录都有访问许可权限，用它来确定谁可以通过何种方式对文件和目录进行访问和操作。

文件或目录的访问权限分为只读，只写和可执行三种。以文件为例，只读权限表示只允许读其内容，而禁止对其做任何的更改操作。可执行权限表示允许将该文件作为一个程序执行。文件被创建时，文件所有者自动拥有对该文件的读、写和可执行权限，以便于对文件的阅读和修改。用户也可根据需要把访问权限设置为需要的任何组合。

有三种不同类型的用户可对文件或目录进行访问：文件所有者，同组用户、其他用户。所有者一般是文件的创建者。所有者可以允许同组用户有权访问文件，还可以将文件的访问权限赋予系统中的其他用户。在这种情况下，系统中每一位用户都能访问该用户拥有的文件或目录。

每一文件或目录的访问权限都有三组，每组用三位表示，分别为文件属主的读、写和执行权限；与属主同组的用户的读、写和执行权限；系统中其他用户的读、写和执行权限。当用ls -l命令显示文件或目录的详细信息时，最左边的一列为文件的访问权限。 例如：

``` bash
[toor@localhost cmdtest]$ ls -al
总用量 4
drwxrwxr-x.  2 toor toor   24 5月  11 09:19 .
drwx------. 15 toor toor 4096 5月  11 09:19 ..
-rw-rw-r--.  1 toor toor    0 5月  11 09:19 2018511.log
```

以2018511.log为例：

`-rw-rw-r--.  1 toor toor    0 5月  11 09:19 2018511.log`

第一列共有10个位置，第一个字符指定了文件类型。在通常意义上，一个目录也是一个文件。如果第一个字符是横线，表示是一个非目录的文件。如果是d，表示是一个目录。从第二个字符开始到第十个共9个字符，3个字符一组，分别表示了3组用户对文件或者目录的权限。权限字符用横线代表空许可，r代表只读，w代表写，x代表可执行。

例如：
`-rw-rw-r--`
　　表示2018511.log是一个普通文件；2018511.log的属主有读写权限；与log2012.log属主同组的用户有读写权限；其他用户也只有读权限。

确定了一个文件的访问权限后，用户可以利用Linux系统提供的chmod命令来重新设定不同的访问权限。也可以利用chown命令来更改某个文件或目录的所有者。利用chgrp命令来更改某个文件或目录的用户组。

chmod命令是非常重要的，用于改变文件或目录的访问权限。用户用它控制文件或目录的访问权限。

## 命令格式

```bash
chmod [-cfvR] [--help] [--version] mode file
```

## 命令功能

用于改变文件或目录的访问权限，用它控制文件或目录的访问权限。

## 命令参数

** 必要参数：**

```bash
-c 当发生改变时，报告处理信息
-f 错误信息不输出
-R 处理指定目录以及其子目录下的所有文件
-v 运行时显示详细处理信息
```

** 选择参数 **

```bash
--reference=<目录或者文件> 设置成具有指定目录或者文件具有相同的权限
--version 显示版本信息
<权限范围>+<权限设置> 使权限范围内的目录或者文件具有指定的权限
<权限范围>-<权限设置> 删除权限范围的目录或者文件的指定权限
<权限范围>=<权限设置> 设置权限范围内的目录或者文件的权限为指定的值
```

** 权限范围 **

``` bash
u ：目录或者文件的当前的用户
g ：目录或者文件的当前的群组
o ：除了目录或者文件的当前用户或群组之外的用户或者群组
a ：所有的用户及群组
```

** 权限代号**

``` bash
r ：读权限，用数字4表示
w ：写权限，用数字2表示
x ：执行权限，用数字1表示
- ：删除权限，用数字0表示
s ：特殊权限
```

该命令有两种用法。一种是包含字母和操作符表达式的文字设定法；另一种是包含数字的数字设定法。

*** 文字设定法 ***

格式：

```bash
chmod ［who］ ［+ | - | =］ ［mode］ 文件名
```

*** 数字设定法 ***

格式：

```bash
chmod ［mode］ 文件名
```

0表示没有权限，1表示可执行权限，2表示可写权限，4表示可读权限，然后将其相加。所以数字属性的格式应为3个从0到7的八进制数，其顺序是（u）（g）（o）。

例如，如果想让某个文件的属主有“读/写”二种权限，需要把4（可读）+2（可写）＝6（读/写）。

数字与字符对应关系如下：

r=4，w=2，x=1
若要rwx属性则4+2+1=7
若要rw-属性则4+2=6；
若要r-x属性则4+1=5。

## 使用实例

** 增加文件所有用户组可执行权限 **

命令：

``` bash
chmod a+x 2018511.log
```

输出：

``` bash
[toor@localhost cmdtest]$ ls -al 2018511.log
-rw-rw-r--. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$ chmod a+x 2018511.log
[toor@localhost cmdtest]$ ls -al 2018511.log
-rwxrwxr-x. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$
```

说明：
即设定文件2018511.log的属性为：文件属主（u） 增加执行权限；与文件属主同组用户（g） 增加执行权限；其他用户（o） 增加执行权限。

** 同时修改不同用户权限**

命令：

```bash
chmod ug+w,o-x 2018511.log
```

输出：

``` bash
[toor@localhost cmdtest]$ ls -al 2018511.log
-r-xr-xr-x. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$ chmod ug+w,o-x 2018511.log
[toor@localhost cmdtest]$ ls -al 2018511.log
-rwxrwxr--. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$
```

说明：
即设定文件text的属性为：文件属主（u） 增加写权限;与文件属主同组用户（g） 增加写权限;其他用户（o） 删除执行权限

** 删除文件权限 **

命令：

```bash
chmod a-x 2018511.log
```

输出：

``` bash
[toor@localhost cmdtest]$ ls -al 2018511.log
-rwxrwxr--. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$ chmod a-x 2018511.log
[toor@localhost cmdtest]$ ls -al 2018511.log
-rw-rw-r--. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$
```

说明：
删除所有用户的可执行权限

** 使用“=”设置权限**

命令：

```bash
chmod u=x 2018511.log
```

输出：

```bash
[toor@localhost cmdtest]$ ls -al 2018511.log
-rw-rw-r--. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$ chmod u=x 2018511.log
[toor@localhost cmdtest]$ ls -al 2018511.log
---xrw-r--. 1 toor toor 0 5月  11 09:19 2018511.log
[toor@localhost cmdtest]$
```

说明：
文件属主（u）等于执行权限;其它权限不变

** 对一个目录及其子目录所有文件添加权限**

命令：

```bash
chmod -R u+x test1
```

输出：

```bash
[toor@localhost test1]$ ls -al
总用量 0
drwxrwxr-x. 2 toor toor 36 5月  11 10:46 .
drwxrwxr-x. 3 toor toor 36 5月  11 10:46 ..
-rw-rw-r--. 1 toor toor  0 5月  11 10:46 log1.log
-rw-rw-r--. 1 toor toor  0 5月  11 10:46 log2.log
[toor@localhost test1]$ cd ..
[toor@localhost cmdtest]$ chmod -R u+x test1
[toor@localhost cmdtest]$ cd test1
[toor@localhost test1]$ ls -al
总用量 0
drwxrwxr-x. 2 toor toor 36 5月  11 10:46 .
drwxrwxr-x. 3 toor toor 36 5月  11 10:46 ..
-rwxrw-r--. 1 toor toor  0 5月  11 10:46 log1.log
-rwxrw-r--. 1 toor toor  0 5月  11 10:46 log2.log
[toor@localhost test1]$
```

说明：
递归地给test1目录下所有文件和子目录的属主分配权限

** 其他实例：**

给file的属主分配读、写、执行(7)的权限，给file的所在组分配读、执行(5)的权限，给其他用户分配执行(1)的权限，命令：

``` bash
chmod 751 file
chmod u=rwx,g=rx,o=x file
```

为所有用户分配读权限，命令：

```bash
chmod 444 file
chmod a-wx,a+r   file
```

参考：

[http://www.cnblogs.com/peida/archive/2012/11/29/2794010.html](http://www.cnblogs.com/peida/archive/2012/11/29/2794010.html)