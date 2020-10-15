---
title: DotNet面试题解析02-拆箱与装箱
date: 2018-12-05 14:53:30
tags:
- DotNet
- CSharp
- CSharp基础
- DotNet面试题解析
categories: 
- DotNet面试题解析
---
## 装箱与拆箱

<font color=#0099ff size=4 face="黑体">
所有值类型都是继承自 System.ValueType ，而 System.ValueType 继承自 System.Object 。因此 Object 是 .NET 中的万物之源，几乎所有类型都来自她，这是装箱与拆箱的基础。</font>

### 基本概念

拆箱与装箱就是值类型与引用类型的转换，她是值类型和引用类型之间的桥梁，他们可以相互转换的一个基本前提就是上面所说的：Object 是 .NET 中的万物之源

```csharp
int x = 1023;
object o = x; //装箱
int y = (int) o; //拆箱
```

**装箱**：值类型转换为引用对象，一般是转换为 System.Object 类型或值类型实现的接口引用类型；
**拆箱**：引用类型转换为值类型，注意，这里的引用类型只能是被装箱的引用类型对象；

<font color=#0099ff size=4 face="黑体">
装箱是非常“昂贵”的操作，它需要 内存分配 和 内存复制，并且由于需要回收临时创建的装箱对象，因此会对垃圾回收器产生压力。</font>

### 装箱的过程

```csharp
int x = 1023;
object o = x; //装箱
```

<font color=#0099ff size=4 face="黑体">
当编译器检测到需要将值类型实例当作引用类型时，就会生成IL指令box。然后，JIT编译器解释该指令，调用某个方法分派堆内存，将值类型实例的内容复制到堆上，并用对象头（对象头字节和方法表指针（类型对象指针））包装起来。每当需要对象引用的时候，就使用这种“箱子”。</font>

### 拆箱的过程

```csharp
int x = 1023;
object o = x; //装箱
int y = (int) o; //拆箱
```

具体过程：

* 1.检查实例对象（ object o ）是否有效，如是否为 null，其装箱的类型与拆箱的类型（int）是否一致，如检测不合法，抛出异常；
* 2.指针返回，就是获取装箱对象（object o）中值类型字段值的地址；
* 3.字段拷贝，把装箱对象（object o）中值类型字段值拷贝到栈上，意思就是创建一个新的值类型变量来存储拆箱后的值；

### 总结

只有值类型才有装箱、拆箱两个状态，而引用类型一直都在箱子里。

一般来说，装箱的性能开销更大，因为引用对象的分配更加复杂，成本也更高，值类型分配在栈上，分配和释放的效率都很高。装箱过程是需要创建一个新的引用类型对象实例，拆箱过程需要创建一个值类型字段，开销更低。

为了尽量避免这种性能损失，尽量使用泛型，在代码编写中也尽量避免隐式装箱。

在值类型中重写 Equals ,重载 Equals ,实现 IEquatable<T> ,以及重载操作符==和!=（通过 constrained IL  前缀）。

注：constrained：强迫;强使;限制;约束

## 题目答案解析

### 1.什么是拆箱和装箱？

装箱就是值类型转换为引用类型，拆箱就是引用类型（被装箱的对象）转换为值类型。

### 2.什么是箱子？

值类型转换为引用类型的对象。

### 3.箱子放在哪里？

托管堆上。

### 4.装箱和拆箱有什么性能影响？

装箱和拆箱都涉及到内存的分配和对象的创建，有较大的性能影响。

### 5.如何避免隐身装箱？

编码中，多使用泛型、显示装箱。

### 6.箱子的基本结构？

上面说了，箱子就是一个引用类型对象，因此它的结构，主要包含两部分：

* 值类型字段值；
* 引用类型的标准配置，引用对象的额外空间：对象头字节（包括同步块索引）和 TypeHandle（或方法表指针，类型对象指针）

### 7.装箱的过程？

* 1.在堆中申请内存，内存大小为值类型的大小，再加上额外固定空间（引用类型的标配：对象头字节（包括同步块索引）和 TypeHandle（或方法表指针，类型对象指针））；
* 2.将值类型的字段值（x=1023）拷贝新分配的内存中；
* 3.返回新引用对象的地址（给引用变量object o）

### 8.拆箱的过程？

* 1.检查实例对象（object o）是否有效，如是否为null，其装箱的类型与拆箱的类型（int）是否一致，如检测不合法，抛出异常；
* 2.指针返回，就是获取装箱对象（object o）中值类型字段值的地址；
* 3.字段拷贝，把装箱对象（object o）中值类型字段值拷贝到栈上，意思就是创建一个新的值类型变量来存储拆箱后的值；

### 9.下面这段代码输出什么？共发生多少次装箱？多少次拆箱？

```csharp
int i = 5;
object obj = i;
IFormattable ftt = i;
Console.WriteLine(System.Object.ReferenceEquals(i, obj));
Console.WriteLine(System.Object.ReferenceEquals(i, ftt));
Console.WriteLine(System.Object.ReferenceEquals(ftt, obj));
Console.WriteLine(System.Object.ReferenceEquals(i, (int)obj));
Console.WriteLine(System.Object.ReferenceEquals(i, (int)ftt));
```

上面代码输出如下：

```code
False
False
False
False
False
```

参考：

[.NET面试题解析(02)-拆箱与装箱](http://www.cnblogs.com/anding/p/5236739.html)

[2md](https://phodal.github.io/2md/)