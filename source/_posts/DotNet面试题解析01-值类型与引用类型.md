---
title: DotNet面试题解析01-值类型与引用类型
date: 2018-12-04 10:17:55
tags:
- DotNet
- CSharp
- CSharp基础
- DotNet面试题解析
categories: 
- DotNet面试题解析
---
## 基本概念

`CLR` 支持两种类型：**引用类型**和**值类型**。这是 `.NET` 语言的基础和关键，他们从类型定义、实例创建、参数传递，到内存分配都有所不同。

![typesystem.gif](/img/typesystem.gif)

下图清晰了展示了 `.NET` 中类型分类，值类型主要是一些简单的、基础的数据类型，引用类型主要用于更丰富的、复杂的、复合的数据类型。

![2009020510331710.jpg](/img/2009020510331710.jpg)

### 使用值类型最佳实践

* 对象很小，并且打算创建很多这样的对象
* 需要高密度的内存集合
* 在值类型中重写 `Equals` ,重载 `Equals` ,实现 `IEquatable<T>` ,以及重载操作符==和!=。
* 在值类型中重写 `GetHashCode`。
* 考虑将值类型实现为不可变的。

### 为什么要为值类型重定义相等性

原因主要有以下几点：

* 值类型默认无法使用 == 操作符，除非对它进行重写 ( `System.ValueType` 里面对 `object.Equals()` 方法的重写)
* 性能原因，因为值类型默认的相等性比较会使用装箱和反射，所以性能很差
* 根据业务需求，其实际相等性的意义和默认的比较结果可能会不同，但是这种情况可能不较少

![986268-20190413072415659-1667812156.png](/img/986268-20190413072415659-1667812156.png)

值类型和 `string` 类型除外，因为所有值类型继承于 `System.ValueType()` ( `System.ValueType()` 同样继承于`Object`，但是 `System.ValueType()` 本身却是引用类型)，而 `System.ValueType()` 对 `Equals()` 和==操作符进行了重写，是逐字节比较的。而 `string` 类型是比较特殊的引用类型，所以 `string` 在很多地方都是特殊处理的。

在值类型使用 `Equals()` 时，因为 `Equals()` 使用了反射，在比较时会影响效率。

注意：

如果要把引用类型做为 `Dictionary` 或 `HashTable` 的 `key` 使用时，必须重写这两个方法。

原因：当我们把引用类型( `string` 除外)做为 `Dictionary` 或 `HashTable` 的key时，有可能永远无法根据 `Key` 获得 `value` 的值，或者说两个类型的 `HashCode` 永远不会相等。

拿 `Dictionary` 来说，虽然我们存储的时候是键值对，但是 `CLR` 会先把 `key` 转成 `HashCode` 并且验证 `Equals` 后再做存储，根据 `key` 取值的时候也是把 `key` 转换成 `HashCode` 并且验证 `Equals` 后再取值。

注意验证时 `HashCode` 和 `Equals` 的关系是并且( `&&` )的关系。也就是说，只要 `GetHashCode` 和 `Equlas` 中有一个方法没有重写，在验证时没有重写的那个方法会调用基类的默认实现，而这两个方法的默认实现都是根据内存地址判断的，也就是说，只重写一个方法的返回值永远会是 `false` 。

参考：

[聊一聊C#的Equals()和GetHashCode()方法](https://www.cnblogs.com/xiaochen-vip8/p/5506478.html)

### 实现步骤

* 重写 `object.Equals()` 方法
* 实现 `IEquatable<T>.Equals()` 接口方法
* 重写 == 和 != 操作符
* 重写 `object.GetHashCode()`

参考：

[C# 为值类型重定义相等性](https://www.cnblogs.com/cgzl/p/10699667.html)

### 如何重写GetHashCode方法

`Object` 中 `GetHasCode` 的算法保证了同一对象返回同一 `HashCode`，而不同的对象返回不同的 `HashCode`，但对于值类型等，视内容相等的对象为相等对象的类型时，默认的 `GetHashCode` 算法并不正确。重写后的 `GetHashCode` 必须要保证同一对象无论何时都返回同一 `HashCode` 值，而相等的对象也必须返回相同的值。并且在此基础上，保证 `HashCode` 尽量随机地散列分布。

### 引用类型

标准配置，引用对象的额外空间：`对象头字节（包括同步块索引）`和 `TypeHandle（或方法表指针，类型对象指针）`

方法表指针指向一个 `CLR` 内部数据结构--方法表（ `MT` ）。方法表指向另一个内部数据结构---`EEClass` (EE代表`Execution Engine`,执行引擎)。方法表和 `EEClass` 包含一些信息，可用来分发虚方法和接口方法调用，访问静态变量，确定对象的运行时类型，高效访问基类型方法等。

## 内存结构

`值类型` 和 `引用类型` 最根源的区别就是其内存分配的差异，在这之前首先要理解 `CLR` 的内存中两个重要的概念：

**Stack 栈**：线程栈，由操作系统管理，**存放值类型、引用类型变量（就是引用对象在托管堆上的地址）**。栈是基于线程的，也就是说一个线程会包含一个线程栈，线程栈中的值类型在对象作用域结束后会被清理，效率很高。

**GC Heap托管堆**：进程初始化后在进程地址空间上划分的内存空间，存储 `.NET` 运行过程中的对象，**所有的引用类型都分配在托管堆上**，托管堆上分配的对象是由 `GC` 来管理和释放的。托管堆是基于进程的，当然托管堆内部还有其他更为复杂的结构。

结合下图理解，变量a及其值3都是存储在栈上面。变量b在栈上存储，其值指向字符串“123”的托管堆对象地址(字符串是引用类型，字符串对象是存储在托管堆上面。字符串是一个特殊的引用类型)

![151257-20160301092426314-1061559039.png](/img/151257-20160301092426314-1061559039.png)

**值类型一直都存储在栈上面吗？所有的引用类型都存储在托管堆上面吗？**

* 1.单独的值类型变量，如局部值类型变量都是存储在栈上面的；
* 2.当值类型是自定义 `class` 的一个字段、属性时，它随引用类型存储在托管堆上，此时她是引用类型的一部分；
* 4.所有的引用类型肯定都是存放在托管堆上的。
* 5.结构体（值类型）中定义引用类型字段，结构体是存储在栈上，其引用变量字段只存储内存地址，指向堆中的引用实例。

## 对象的传递

<font color=#0099ff size=4 face="黑体">将值类型的变量赋值给另一个变量（或者作为参数传递），会执行一次值复制。
将引用类型的变量赋值给另一个引用类型的变量，它复制的值是引用对象的内存地址，因此赋值后就会多个变量指向同一个引用对象实例。</font>

理解这一点非常重要，下面代码测试验证一下：

```c#
int v1 = 0;
int v2 = v1;
v2 = 100;
Console.WriteLine("v1=" + v1); //输出：v1=0
Console.WriteLine("v2=" + v2); //输出：v2=100

User u1=new User();
u1.Age = 0;
User u2 = u1;
u2.Age = 100;
Console.WriteLine("u1.Age=" + u1.Age); //输出：u1.Age=100
Console.WriteLine("u2.Age=" + u2.Age); //输出：u2.Age=100，因为u1/u2指向同一个对象
```

当把对象作为参数传递的时候，效果同上面一样，他们都称为按值传递，但因为值类型和引用类型的区别，导致其产生的效果也不同。

### 参数-按值传递：

```csharp
private void DoTest(int a)
{
    a *= 2;
}

private void DoUserTest(User user)
{
    user.Age *= 2;
}

[NUnit.Framework.Test]
public void DoParaTest()
{
    int a = 10;
    DoTest(a);
    Console.WriteLine("a=" + a); //输出：a=10
    User user = new User();
    user.Age = 10;
    DoUserTest(user);
    Console.WriteLine("user.Age=" + user.Age); //输出：user.Age=20
}
```

上面的代码示例，两个方法的参数，都是按值传递

* 对于值类型( `int a`) ：传递的是变量a的值拷贝副本，因此原本的a值并没有改变。
* 对于引用类型( `User user`) ：传递的是变量 `user` 的引用地址（ `User` 对象实例的内存地址）拷贝副本，因此他们操作都是同一个 `User` 对象实例。

### 参数-按引用传递：

<font color=#0099ff size=4 face="黑体">
按引用传递的两个主要关键字：out 和 ref不管值类型还是引用类型，按引用传递的效果是一样的，都不传递值副本，而是引用的引用（类似c++的指针的指针）。out 和 ref告诉编译器方法传递额是参数地址，而不是参数本身，理解这一点很重要。
</font>

```csharp
private void DoTest(ref int a)
{
    a *= 2;
}

private void DoUserTest(ref User user)
{
    user.Age *= 2;
}

public void DoParaTest()
{
    int a = 10;
    DoTest(ref a);
    Console.WriteLine("a=" + a); //输出：a=20 ,a的值改变了
    User user = new User();
    user.Age = 10;
    DoUserTest(ref user);
    Console.WriteLine("user.Age=" + user.Age); //输出：user.Age=20
}
```

```csharp
int m = 0;
Console.WriteLine(RefValue(1, ref m)); //输出 1
Console.WriteLine(m);  //输出 222

int n;
Console.WriteLine(OutValue(1, out n));  //输出 223
Console.WriteLine(n);  //输出 222

static int RefValue(int i, ref int j)
{
    int k = j;
    j = 222;
    return i + k;
}

static int OutValue(int i, out int j)
{
    j = 222;
    return i + j;
}
```

`out` 和 `ref`的主要异同:

* `out` 和 `ref` 都指示编译器传递参数地址，在行为上是相同的；
* 他们的使用机制稍有不同，ref 必须要求参数在使用之前要显式初始化，out 必须要在方法内部初始化；
* 使用 ref 和 out 时，在方法的参数和执行方法时，都要加Ref或Out关键字，以满足匹配；
* out 适合用在需要 retrun 多个返回值的地方，而 ref 则用在需要被调用的方法修改调用者的引用的时候。
* out 和 ref 不可以重载，就是不能定义 `Method(ref int a)` 和 `Method(out int a)` 这样的重载，从编译角度看，二者的实质是相同的，只是使用时有区别；
* ref 是有进有出，而 out 是只出不进。

<font color=#ff0000 size=4 face="黑体">
注：在C#中，方法的参数传递有四种类型：传值（by value），传址（by reference），输出参数（by output），数组参数（by array）。传值参数无需额外的修饰符，传址参数需要修饰符ref，输出参数需要修饰符out，数组参数需要修饰符params。</font>

### 常见问题

![值类型和引用类型区别.jpg](/img/值类型和引用类型区别.jpg)

## 题目答案解析

### 1. 值类型和引用类型的区别？

值类型包括简单类型、结构体类型和枚举类型，引用类型包括自定义类、数组、接口、委托等。

* 1、赋值方式：将一个值类型变量赋给另一个值类型变量时，将复制包含的值。这与引用类型变量的赋值不同，引用类型变量的赋值只复制对象的引用（即内存地址，类似 `C++` 中的指针），而不复制对象本身。
* 2、继承：值类型不可能派生出新的类型，所有的值类型均隐式派生自 `System.ValueType`。但与引用类型相同的是，结构也可以实现接口。
* 3、null：与引用类型不同，值类型不可能包含 null 值。然而，可空类型允许将 null 赋给值类型（他其实只是一种语法形式，在 `clr` 底层做了特殊处理）。
* 4、每种值类型均有一个隐式的默认构造函数来初始化该类型的默认值，值类型初始会默认为0，引用类型默认为null。
* 5、值类型存储在栈中，引用类型存储在托管堆中。

### 2. 结构和类的区别？

结构体是值类型，类是引用类型，主要区别如题1。其他的区别：

* 结构不支持无惨构造函数，不支持析构函数，并且不能有 protected 修饰；
* 结构常用于数据存储，类 class 多用于行为；
* class 需要用 new 关键字实例化对象，struct 可以不适用 new 关键字；
* class 可以为抽象类，struct 不支持抽象；

### 3. delegate是引用类型还是值类型？enum、int[]和string呢？

enum 枚举是值类型，其他都是引用类型。

### 4. 堆和栈的区别？

线程堆栈：简称栈 Stack
托管堆： 简称堆 Heap

* 值类型大多分配在栈上，引用类型都分配在堆上；
* 栈由操作系统管理，栈上的变量在其作用域完成后就被释放，效率较高，但空间有限。堆受 CLR 的 GC 控制；
* 栈是基于线程的，每个线程都有自己的线程栈，初始大小为1M。堆是基于进程的，一个进程分配一个堆，堆的大小由 GC 根据运行情况动态控制；

### 5.“结构”对象可能分配在堆上吗？什么情况下会发生，有什么需要注意的吗？

结构是值类型，有两种情况会分配在对上面：

* 结构作为class的一个字段或属性，会随class一起分配在堆上面；
* 装箱后会在堆中存储，尽量避免值类型的装箱，值类型的拆箱和装箱都有性能损失；

### 6. 理解参数按值传递？以及按引用传递？

* 按值传递：对于值类型传递的它的值拷贝副本，而引用类型传递的是引用变量的内存地址，他们还是指向的同一个对象。
* 按引用传递：通过关键字 out 和 ref 传递参数的内存地址，值类型和引用类型的效果是相同的。

### 7. `out` 和 `ref`的区别与相同点？

* `out` 和 `ref` 都指示编译器传递参数地址，在行为上是相同的；
* 他们的使用机制稍有不同，ref 要求参数在使用之前要显式初始化，out 要在方法内部初始化；
* `out` 和 `ref` 不可以重载，就是不能定义 Method(ref int a) 和 Method(out int a) 这样的重载，从编译角度看，二者的实质是相同的，只是使用时有区别；

### 8. C#支持哪几个预定义的值类型？C#支持哪些预定义的引用类型？

值类型：整数、浮点数、字符、bool 和 decimal

引用类型：Object，String

### 9. 有几种方法可以判定值类型和引用类型？

简单来说，继承自 System.ValueType 的是值类型，反之是引用类型。

### 10. 说说值类型和引用类型的生命周期？

值类型在作用域结束后释放。

引用类型由 GC 垃圾回收期回收。

### 11. 如果结构体中定义引用类型，对象在内存中是如何存储的？例如下面结构体中的class类 User对象是存储在栈上，还是堆上？

```c#
public struct MyStruct
{
    public int Index;
    public User User;
}
```

MyStruct 存储在栈中，其字段 User 的实例存储在堆中，MyStruct.User 字段存储指向 User 对象的内存地址。

参考：

[.NET面试题解析(01)-值类型与引用类型](http://www.cnblogs.com/anding/p/5229756.html)

[2md](https://phodal.github.io/2md/)

[C#基础：ref和out的区别](https://www.cnblogs.com/gjahead/archive/2008/02/28/1084871.html)