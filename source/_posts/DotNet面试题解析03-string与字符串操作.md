---
title: DotNet面试题解析03-string与字符串操作
date: 2018-12-06 14:20:09
tags:
- DotNet
- CSharp
- CSharp基础
- DotNet面试题解析
categories: 
- DotNet面试题解析
---
## 字符串操作

### 认识string

string 是一个特殊的引用类型,其对象值存储在托管堆中。string 的内部是一个 char 集合，他的长度 Length 就是字符char 数组的字符个数，一个字符两个字节。string 不允许使用 new string() 的方式创建实例，而是另一种更简单的语法，直接赋值（string aa= “000” 这一点也类似值类型）。

```csharp
public void DoStringTest()
{
    var aa = "000";
    SetStringValue(aa);
    Console.WriteLine(aa);
}

private void SetStringValue(string aa)
{
    aa += "111";
}
```

<font color=#0099ff size=4 face="黑体">上面的输出结果为“000”。</font>

通过前面的值类型与引用类型的文章，我们知道 string 是一个引用类型，既然是一个引用类型，参数传递的是引用地址，那为什么不是输出“000111”呢？是不是很有值类型的特点呢！这一切的原因源于 string 类型的两个重要的特性：恒定性 与 驻留性。

### String的恒定性（不变性）

<font color=#0099ff size=4 face="黑体">字符串是不可变的，字符串一经创建，就不会改变，任何改变都会产生新的字符串。</font>

![151257-20160303222417112-1147871973.png](/img/151257-20160303222417112-1147871973.png)

上文中的 任何改变都会产生新的字符串 ，包括字符串的一些操作函数，如 str1.ToLower，Trim()，Remove(int startIndex, int count)，ToUpper() 等，都会产生新的字符串，因此在很多编程实践中，对于字符串忽略大小的比较：

```csharp
if（str1.ToLower()==str2.ToLower()） //这种方式会产生新的字符串，不推荐
if（string. Compare(str1,str2,true)） //这种方式性能更好
```

### String的驻留性

由于字符串的不变性，在大量使用字符串操作时，会导致创建大量的字符串对象，带来极大的性能损失。因此 CLR 又给 string 提供另外一个法宝，就是字符串驻留，先看看下面的代码，字符串 s1、s2 竟然是同一个对象！

```csharp
var s1 = "123";
var s2 = "123";
Console.WriteLine(System.Object.Equals(s1, s2));  //输出 True
Console.WriteLine(System.Object.ReferenceEquals(s1, s2));  //输出 True
```

<font color=#0099ff size=4 face="黑体">相同的字符串在内存（堆）中只分配一次，第二次申请字符串时，发现已经有该字符串是，直接返回已有字符串的地址，这就是驻留的基本过程。</font>

字符串驻留的基本原理：

* CLR 初始化时会在内存中创建一个驻留池，内部其实是一个哈希表，存储被驻留的字符串和其内存地址。
* 驻留池是进程级别的，多个 AppDomain 共享。同时她不受 GC 控制，生命周期随进程，意思就是不会被 GC 回收
* 当分配字符串时，首先会到驻留池中查找，如找到，则返回已有相同字符串的地址，不会创建新字符串对象。如果没有找到，则创建新的字符串，并把字符串添加到驻留池中。

如果大量的字符串都驻留到内存里，而得不到释放，不是很容易造成内存爆炸吗，当然不会了? 因为不是任何字符串都会驻留，只有通过 IL 指令 ldstr 创建的字符串才会留用。

```csharp
var s1 = "123";
var s2 = s1 + "abc";
var s3 = string.Concat(s1, s2);
var s4 = 123.ToString();
var s5 = s2.ToUpper();
```

IL代码如下：
![151257-20160303221217190-205612505.png](/img/151257-20160303221217190-205612505.png)

在上面的代码中，出现两个字符串常量，“123” 和 “abc”，这个两个常量字符串在 IL 代码中都是通过 IL 指令 ldstr 创建的，只有该指令创建的字符串才会被驻留，其他方式产生新的字符串都不会被驻留，也就不会共享字符串了，会被 GC 正常回收。

那该如何来验证字符串是否驻留呢，string 类提供两个静态方法：

* string.Intern(string str)  可以主动驻留一个字符串；
* string.IsInterned(string str)  检测指定字符串是否驻留，如果驻留则返回字符串，否则返回NULL

```csharp
var s1 = "123";
var s2 = s1 + "abc";
Console.WriteLine(s2);   //输出：123abc
Console.WriteLine(string.IsInterned(s2) ?? "NULL");   //输出：NULL。因为“123abc”没有驻留

string.Intern(s2);   //主动驻留字符串
Console.WriteLine(string.IsInterned(s2) ?? "NULL");   //输出：123abc
```

### 认识StringBuilder

大量的编程实践和意见中，都说大量字符串连接操作，应该使用 StringBuilder。相对于 string 的不可变，StringBuilder 代表可变字符串，不会像字符串，在托管堆上频繁分配新对象。

首先 StringBuilder 内部同 string 一样，有一个 char[] 字符数组，负责维护字符串内容。因此，与 char 数组相关，就有两个很重要的属性：

* public int Capacity：StringBuilder 的容量，其实就是字符数组的长度。
* public int Length：StringBuilder 中实际字符的长度，>=0，<= 容量 Capacity。

StringBuilder 之所以比 string 效率高，主要原因就是不会创建大量的新对象，StringBuilder 在以下两种情况下会分配新对象：

* 追加字符串时，当字符总长度超过了当前设置的容量 Capacity，这个时候，会重新创建一个更大的字符数组，此时会涉及到分配新对象。
* 调用 StringBuilder.ToString()，创建新的字符串。

**追加字符串的过程**：

* StringBuilder 的默认初始容量为16；
* 使用 stringBuilder.Append() 追加一个字符串时，当字符数大于16，StringBuilder 会自动申请一个更大的字符数组，一般是倍增；
* 在新的字符数组分配完成后，将原字符数组中的字符复制到新字符数组中，原字符数组就被GC回收；
* 最后把需要追加的字符串追加到新字符数组中；

简单来说，当 StringBuilder 的容量 Capacity 发生变化时，就会引起托管对象申请、内存复制等操作，带来不好的性能影响，因此设置合适的初始容量是非常必要的，尽量减少内存申请和对象创建。

验证代码：

```csharp
StringBuilder sb1 = new StringBuilder();
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=16; Length=0;   //初始容量为16 
sb1.Append('a', 12);    //追加12个字符
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=16; Length=12;  
sb1.Append('a', 20);    //继续追加20个字符，容量倍增了
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=32; Length=32;  
sb1.Append('a', 41);    //追加41个字符，新容量=32+41=73
Console.WriteLine("Capacity={0}; Length={1};", sb1.Capacity, sb1.Length); //输出：Capacity=73; Length=73;  

StringBuilder sb2 = new StringBuilder(80); //设置一个合适的初始容量
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=0;
sb2.Append('a', 12);
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=12;
sb2.Append('a', 20);
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=32;
sb2.Append('a', 41);
Console.WriteLine("Capacity={0}; Length={1};", sb2.Capacity, sb2.Length); //输出：Capacity=80; Length=73;
```

为什么少量字符串不推荐使用 StringBuilder 呢？因为 StringBuilder 本身是有一定的开销的，少量字符串就不推荐使用了，使用 String.Concat 和 String.Join 更合适。

### 高效的使用字符串

* 在使用线程锁的时候，不要锁定一个字符串对象，因为字符串的驻留性，可能会引发不可以预料的问题；
* 理解字符串的不变性，尽量避免产生额外字符串，如：

```csharp
if（str1.ToLower()==str2.ToLower()） //这种方式会产生新的字符串，不推荐
if（string. Compare(str1,str2,true)） //这种方式性能更好
```

* 在处理大量字符串连接的时候，尽量使用 StringBuilder，在使用 StringBuilder 时，尽量设置一个合适的长度初始值；

```csharp
StringBuilder myStringBuilder = new StringBuilder("Hello World!", 25);
或
myStringBuilder.Capacity = 25;
```

* 少量字符串连接建议使用 String.Concat 和 String.Join 代替。

## 题目答案解析

### 1.字符串是引用类型类型还是值类型？

引用类型。

### 2.在字符串连加处理中，最好采用什么方式，理由是什么？

少量字符串连接，使用 String.Concat，大量字符串使用 StringBuilder，因为 StringBuilder 的性能更好，如果 string 的话会创建大量字符串对象。

### 3.使用 StringBuilder时，需要注意些什么问题？

* 少量字符串时，尽量不要用，StringBuilder 本身是有一定性能开销的；
* 大量字符串连接使用 StringBuilder 时，应该设置一个合适的容量。

### 4.以下代码执行后内存中会存在多少个字符串？分别是什么？输出结果是什么？为什么呢？

```csharp
string st1 = "123" + "abc";
string st2 = "123abc";
Console.WriteLine(st1 == st2);
Console.WriteLine(System.Object.ReferenceEquals(st1, st2));
```

输出结果：

```text
True
True
```

内存中的字符串只有一个“123abc”,第一行代码（string st1 = "123" + "abc"; ）常量字符串相加会被编译器优化。由于字符串驻留机制，两个变量st1、st2都指向同一个对象。IL代码如下：

![151257-20160303221219330-60155453.png](/img/151257-20160303221219330-60155453.png)

### 5.以下代码执行后内存中会存在多少个字符串？分别是什么？输出结果是什么？为什么呢？

```csharp
string s1 = "123";
string s2 = s1 + "abc";
string s3 = "123abc";
Console.WriteLine(s2 == s3);
Console.WriteLine(System.Object.ReferenceEquals(s2, s3));
```

结果：

```text
True
False
```

4个字符串。
分别是123，abc，123abc，123abc。

理由：
s1+abc时产生的不是可驻留的的字符串，而下句则是个驻留字符串。
字符串是不可变的，字符串一经创建，就不会改变，任何改变都会产生新的字符串。

### 6.使用C#实现字符串反转算法，例如：输入"12345", 输出"54321"

```csharp
public static string Reverse(string str)
{
    if (string.IsNullOrEmpty(str))
    {
        throw new ArgumentException("参数不合法");
    }

    StringBuilder sb = new StringBuilder(str.Length);  //注意：设置合适的初始长度，可以显著提高效率（避免了多次内存申请）
    for (int index = str.Length - 1; index >= 0; index--)
    {
        sb.Append(str[index]);
    }
    return sb.ToString();
}
```

```csharp
public static string Reverse(string str)
{
    if (string.IsNullOrEmpty(str))
    {
        throw new ArgumentException("参数不合法");
    }
    char[] chars = str.ToCharArray();
    int begin = 0;
    int end = chars.Length - 1;
    char tempChar;
    while (begin < end)
    {
        tempChar = chars[begin];
        chars[begin] = chars[end];
        chars[end] = tempChar;
        begin++;
        end--;
    }
    string strResult = new string(chars);
    return strResult;
}
```

```csharp
public static string Reverse(string str)
{
    char[] arr = str.ToCharArray();
    Array.Reverse(arr);
    return new string(arr);
}
```

### 7.下面的代码输出结果？为什么？

```csharp
object a = "123";
object b = "123";
Console.WriteLine(System.Object.Equals(a,b));
Console.WriteLine(System.Object.ReferenceEquals(a,b));
string sa = "123";
Console.WriteLine(System.Object.Equals(a, sa));
Console.WriteLine(System.Object.ReferenceEquals(a, sa));
```

输出结果全是 True，因为他们都指向同一个字符串实例，使用 object 声明和 string 声明在这里并没有区别（string是引用类型）。

### 8.C#中string.Empty、""和null 之间的区别

实际上 Empty 是 string 类中的一个静态的只读字段，他的定义是这样的：

public static readonly String Empty = "";

也就是说 string.Empty 的内部实现是等于""的。

引用类型是将对象是实际数据保存在堆上, 将对象在堆上的地址保存在栈上。因此 string.Empty 与 "" 都会在栈上保存一个地址这个地址占4字节，指向内存堆中的某个长度为0的空间，这个空间保存的是 string.Empty 的实际值

"" 是通过 CLR 进行优化的,CLR 会维护一个字符串池,以防在堆中创建重复的字符串。
而 string.Empty 是一种 c# 语法级别的优化，是在 C# 编译器将代码编译为 IL (即 MSIL )时进行了优化，即所有对 string 类的静态字段 Empty 的访问都会被指向同一引用，以节省内存空间。也就是说，"" 与 string.Empty 在用法与性能上基本没区别。string.Empty 是在语法级别对 "" 的优化。

那就是 string.Empty 会在堆上占用一个长度为0的空间，而null不会。具体内容如下：

```csharp
string str1="";
string str2=null;
```

如刚才所说 str1 会在栈上保存一个地址,这个地址占4字节，指向内存堆中的某个长度为0的空间，这个空间保存的是 str1 的实际值。

str2 同样会在栈上保存一个地址,这个地址也占4字节，但是这个地址是没有明确指向的，它哪也不指，其内容为 0x00000000。

参考：

[.NET面试题解析(03)-string与字符串操作](http://www.cnblogs.com/anding/p/5240313.html)

[C#基础知识梳理系列九：StringBuilder](http://www.cnblogs.com/solan/archive/2012/08/06/CSharp09.html)

[深入理解string和如何高效地使用string](http://www.cnblogs.com/artech/archive/2007/05/06/737130.html)

[C#中string.Empty、""和null 之间的区别](https://blog.csdn.net/henulwj/article/details/7830615)