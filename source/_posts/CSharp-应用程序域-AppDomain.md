---
title: CSharp-应用程序域(AppDomain)
date: 2019-01-24 14:28:36
tags:
- CSharp
- 程序集
- 应用程序域
- AppDomain
categories: 
- AppDomain
---

## AppDomain介绍

为了保证代码的键壮性CLR希望不同服务功能的代码之间相互隔离，这种隔离可以通过创建多个进程来实现，但操作系统中创建进程是即耗时又耗费资源的一件事，所以在CLR中引入了`AppDomain`的概念，`AppDomain` 主要是用来实现同一进程中的各 `AppDomain` 之间的隔离。

`AppDomain` 是.Net 平台里一个很重要的特性，在.Net以前，每个程序是 `封装` 在不同的进程中的，这样导致的结果就造就占用资源大，可复用性低等缺点。而 `AppDomain` 在同一个进程内划分出多个 `域`，一个进程可以运行多个应用，提高了资源的复用性，数据通信等。

CLR在启动的时候会创建`系统域(System Domain)`，`共享域(Shared Domain)`和`默认域(Default Domain)`，系统域与共享域对于用户是不可见的，默认域也可以说是当前域，它承载了当前应用程序的各类信息(堆栈)，所以，我们的一切操作都是在这个默认域上进行。`插件式` 开发很大程度上就是依靠 `AppDomain` 来进行。

应用程序域具有以下特点：

* 必须先将程序集加载到应用程序域中，然后才能执行该程序集。
* 一个应用程序域中的错误不会影响在另一个应用程序域中运行的其他代码。
* 能够在不停止整个进程的情况下停止单个应用程序并卸载代码。不能卸载单独的程序集或类型，只能卸载整个应用程序域。
<!--more-->

**备注：**

在 .NET Core 上， `AppDomain` 实现受设计限制，不提供隔离、卸载或安全边界。 对于 `.NET Core`，只有一个 `AppDomain` 。 通过提供隔离和卸载 `AssemblyLoadContext` 。 安全边界应由进程边界和适当的远程处理技术提供。

`AppDomain` 特征描述：

* `AppDomain` 概念并不存在于操作系统，而只存在于.net中并且`AppDomain`不可脱离进程单独存在，它是属于某一CLR或寄宿着CLR的进程的。
* 一个进程中可以有多个`AppDomain`,并且每个之间相互隔离（只保证安全代码的隔离，不安全代码并不能保证）,此可以理解为`AppDomain`是.net程序中的"进程"，在一个`AppDomain`中创建的对象只属于本`AppDomain`，多个`AppDomain`之间的对象不能相互访问，除非遵循CLR的一些规则。
* .net程序启动时在进程中创建一个默认的 `AppDomain`，入口代码将运行于此`AppDomain`，默认应用程序域只有在进程终止时才会被销毁，如果主动的调用`Unload`去卸载默认应用程序域，会抛出一个`CannotUnloadAppDomainException`。
* 每个`AppDomain`都单独的加载程序集，这意味着在A应用程序域中加载了的程序集，并不一定在B应用程序域中也被加载了。每个`AppDomain`有单独的`Loader堆`，互相不影响，每个`Loader堆`都记录了自`AppDomain` 建立以后所访问过的类型，`Loader堆`中的每个类型对象都有一个方法表，这些方法表指向已经被JIT编译过的本地代码（前提是这些方法是已经被至少执行过一次的）。因为`AppDomain`是相互隔离的所以相同的一个类型中的方法，在A应用程序域中是被JIT编译过的，但不一定在B应用程序域中也是被JIT编辑过的，方法是否被JIT编辑过取决于些方法是否在本`AppDomain`被调用过。
* 当`AppDomain`被卸载时，此`AppDomain`中的程序集会被卸载，因为每个`APPDomain`加载的程序集都是独立的，所以一个应用程序域被卸载并不会影响其它`AppDomain`中加载的程序集,另外本`AppDomain`的`loader堆`也会被回收，每个程序域的`loader堆`是独立的，所以也不会影响到其它程序域的`Loader堆`，因为`loader堆`是独立的（静态字段是存在于类型对象上的），所以一个类型中的静态字段在不同应用程序域中也是不同的存在，所以静态字段也是不被影响的，唯一受影响的是线程，因为线程可以跨越应用程序域访问不同的应用程序域上的代码。
* 有一种程序集可以被多个`AppDomain`使用，这种程序集叫做 `AppDomain中立` 的程序集，比如`MSCorLib.dll`，该程序集包含了`System.Object,System.Int32`以及其它的与.net framework 密不可分的类型，这个程序集在CLR初始化时会自动加载，JIT会为这些程序集创建一个特殊的`Loader堆`，并且程序集中的方法被编译成的本地代码可被所有`AppDomain共享`，这种程序集不可被卸载只有当进程结束时这种程序集才会被卸载。

## AppDomain间对象访问

AppDomain之间的对象传送有两种方式，以下分别介绍：

* 按引用封送方式

* 按值封送方式

### 按引用封送方式

如果一种类型继承自`MarshalByRefObject`就标志着此类型所实例化出来的对象可以按引用封送传递到另一应用程序域中，所谓引用封送就是将对象传递到另一`AppDomain`后对对象所做的修改照实的反应到原对象身上，而不是创建副本对副本进行修改。这里面有个意外就是静态字段，静态字段的数据总是保存在当前应用程序域的`Loader堆`中，对静态字段的访问没有办法创建代理，所以就意味着按引用封送的类型的静态字段并没有引用原对象的静态字段，对目标类型的静态字段所做的修改也不会反映到原对象中。

```c#
public class Person : MarshalByRefObject
{
    public void SomeMethod()
    {
        Console.WriteLine("current domain is:" + AppDomain.CurrentDomain.FriendlyName);
    }

    public string Name
    {
        get;
        set;
    }

    public int Age
    {
        get;
        set;
    }

    public override string ToString()
    {
        return this.Name + ":" + this.Age.ToString();
    }
} 

class Program
{
    static void Main(string[] args)
    {
        AppDomain exampleAppDomain = AppDomain.CreateDomain("new appdomain");
        Person xylee = (Person)exampleAppDomain.CreateInstanceAndUnwrap("ConsoleApp2", "ConsoleApp2.Person");

        Console.WriteLine("Type:{0}", xylee.GetType().ToString());
        Console.WriteLine("Is proxy:{0}", System.Runtime.Remoting.RemotingServices.IsTransparentProxy(xylee));
        xylee.SomeMethod();
        AppDomain.Unload(exampleAppDomain);

        try
        {
            xylee.SomeMethod();
            Console.WriteLine("call is successed");
        }
        catch (AppDomainUnloadedException)
        {
            Console.WriteLine("throw AppDomainUnloadException");
        }


        Console.ReadKey();
    }
}
```

输出的信息为：

```c#
Type:ConsoleApp2.Person
Is proxy:True
current domain is:new appdomain
throw AppDomainUnloadException
```

解析：

首先代码新建了个应用程序域，并给他赋予了一个友好名称 `new appdomain`,此时新创建的`AppDomain`的`Loader堆`是空的，此时线程还处于默认的`AppDomain`中，接着调用`AppDomain`的实例方法`CreateInstanceAndUnwarp`在创建的`AppDomain`中加载程序集并实例了一个`AOPTest.Person`类型的对象，执行`CreateInstanceAndUnwarp`时线程会跨越应用程序域来到`new appdomain`域中，创建好对象后把对象传递到默认`AppDomain`中，这里因为一个应用程序域的对象不能被另一个应用程序域的变量(根)所引用，因为应用程序域的主要作用就是隔离不同的应用程序域，当`CreateInstanceAndUnwarp`要将一个`AppDomain`中的对象传递到另一个`AppDomain`时发现对象的类型是继承自`MarshalByRefObject`类型时就会知道此对象要按引用封送，CLR会在目标`AppDomain`中创建代理类型并实例化，代理类型所公开的成员看起来跟源`AppDomain`中要传递的对象的类型看起来是一样的，但其成员的实现只是调用源`AppDomain`中相应的实现，在示例中我们用`xylee.GetType()`得到的信息跟原类型看起来是一样的就是这个道理，我们可以用`System.Runtime.Remoting.RemotingServices.IsTransparentProxy(xylee)`来判断一个对象是否是代理类型所创建的对象。

接下来示例中卸载了新建的`AppDomain`，新建的`AppDomain`中的程序集和`Loader堆`都会被销毁，这时由于代理对象所引用的原`AppDomain`已经不存在了，那么再调用代理对象中的任何成员都会抛出一个错误，即下一句的：`AppDomainUnloadedException`。

由于按引用封送对象，需要在目标应用程序域创建代理，对对象的所有成员的访问都通过代理转换的缘故，所以执行效率就不会太高，实际上继承自`MarshalByRefObject`的类型不但跨应用程序域访问效率不高，即使在本应用程序域实例化的对象对字段的访问效率也比较低

```c#
public class Demo1 : Object
{
    public Int32 Age;
}

public class Demo2 : MarshalByRefObject
{
    public Int32 Age;
}

class Program
{
    static void Main(string[] args)
    {
        const Int32 count = 10000000;
        Demo1 d1 = new Demo1();
        Demo2 d2 = new Demo2();
        Stopwatch watch = new Stopwatch();
        watch.Start();

        for (int i = 0; i < count; i++)
        {
            d1.Age += 1;
        }

        Console.WriteLine(watch.ElapsedMilliseconds);

        watch.Restart();
        for (int i = 0; i < count; i++)
        {
            d2.Age += 1;
        }
        Console.WriteLine(watch.ElapsedMilliseconds);
        watch.Stop();
        Console.ReadKey();
    }
}
```

结果：

```c#
18
86
```

在本例中由于源应用程序域所创建的对象并没有其应用程序域的根引用它，那它是否会被垃圾回收机制回收了呢？答案是如果一个对象被跨应用程序域按引用封送到另一个应用程序域后，那么它在源应用程序域在没有根引用的情况下会保留5分钟内不会被垃圾回收，在此时间内如果此对象被其它应用程序域的代理访问那么CLR会自动续约2分钟，当然两分钟这个时间是可以改的，你只需要重写继承自`MarshalByRefObject`类的虚函数`public virtual object InitializeLifetimeService()`;即可定制这个时间。

### 按值封送方式

把一个类标识上可以序列化，即为类型添加`SerializableAttribute`即标志着类型实便化出来的对象可以按值封送传递到另一应用程序域中，其内部原理就是在原`AppDomain`中把类序列化成字节流，传递到另一`AppDomain`中，在目标`AppDomain`中再实例化出一个原对象的副本然后用传递过来的字节流对副本进行填充，这样目标对象看起来跟原对象是一样的，但其实只是原对象的一个副本，对副本所做的任何修改并不反应到原对象上。

```c#
[Serializable]
public class Person
{
    public void SomeMethod()
    {
        Console.WriteLine("current domain is:" + AppDomain.CurrentDomain.FriendlyName);
    }

    public string Name
    {
        get;
        set;
    }

    public int Age
    {
        get;
        set;
    }

    public override string ToString()
    {
        return this.Name + ":" + this.Age.ToString();
    }
} 

class Program
{
    static void Main(string[] args)
    {
        AppDomain exampleAppDomain = AppDomain.CreateDomain("new appdomain");
        Person xylee = (Person)exampleAppDomain.CreateInstanceAndUnwrap("ConsoleApp2", "ConsoleApp2.Person");

        Console.WriteLine("Type:{0}", xylee.GetType().ToString());
        Console.WriteLine("Is proxy:{0}", System.Runtime.Remoting.RemotingServices.IsTransparentProxy(xylee));
        xylee.SomeMethod();
        AppDomain.Unload(exampleAppDomain);

        try
        {
            xylee.SomeMethod();
            Console.WriteLine("call is successed");
        }
        catch (AppDomainUnloadedException)
        {
            Console.WriteLine("throw AppDomainUnloadException");
        }
        
        Console.ReadKey(); 
    }
}
```

输出：

```c#
Type:ConsoleApp2.Person
Is proxy:False
current domain is:ConsoleApp2.exe
current domain is:ConsoleApp2.exe
call is successed
```

解析：

当调用`CreateInstanceAndUnwarp`后目标应用程序域加载了程序集并创建了对象之后，如果发现类型没有继承自`MarshalByRefObjec`t,而是标识了可序列化后，将会按值封送传递对象，首先将要封送的对象序列化为字节流，传递回目标应用程序域，然后在目标应用程序域中加载定义此对象类型所在的程序集，并实例化一个类型的实例，再用传递回来的字节流反序列化填充此实例。此时的实例是一个新的对象，跟源程序的对象并没有关系，调用其成员也会在目标应用程序域内的对象上完成，并不会跨域，源应用程序域被卸载也不会影响目标应用程序域里的此对象，所以上例中的`AppDomainUnloadException`异常并不会触发。

**关于System.Runtime.Remoting.ObjectHandle**

在上面两个例子中跨`AppDomain`访问对象的方法`CreateInstanceAndUnwrap`都自动的将传递过来的对象具体化了，方法名中的`Unwrap`也表达了这一点，在对象具体化时会在具体化所在的`AppDomain`中加载要具体化对象类型所在的程序集，然后如果对象是按引用封送的那会在当前`AppDomain`中创建代理类型各实例化，如果对象是按值封送的，那对象的副本会反序列化。

你也可以调用创建实例的非自动具体化版本的方法，即`CreateInstance`，这个方法返回的是`System.Runtime.Remoting.ObjectHandle`,这个类型继承自`MarshalByRefObject`,所以可以穿越`AppDomain`边界，并且可以使对象不具体化，只有当需要的时候才具体化，调用`ObjectHandle`的`Unwarp`实例方法即可。

**如果对象即不继承自MarshalByRefObject又没有标识上SerializableAttribute，那对象在跨越应用程序域时会发生什么？**

答案是通过`CreateInstanceAndUnwarp`是目标应用程序域的对象会创建但调用并不会成功，会抛出`SerializationException`，原因就是`CreateInstanceAndUnwarp`会自动具体化，当具体化时CLR发现类型即没有继承自`MarshalByRefObject`又没标识上可序列化。如果只需要在目标应用程序域里执行一些操作而又不想传递对象建议使用`CreateInstance`方法，当然如果你用这个方法返回的`ObjectHandle`实例试图`Unwarp`时也会报错。

## AppDomain的销毁

卸载`AppDomain`使用`AppDomain`的静态方法`AppDomain.Unload`就可以了，传入要卸载的应用程序域对象。

销毁`AppDomain`时CLR会有以下几个动作。

* 1，挂起所有托管代码中的线程。
* 2，查找所有线程的线程栈，看有那些线程正在执行要卸载应用程序域中，或栈处于要卸载应用程序域中，并给这些线程抛出一个`ThreadAbortException`,并恢复这个线程的执行，将这个异常展开，在展开过程中遇到的所有`Finally`块，执行资源清理，如果没有代码捕获这个`ThreadAbortException`，CLR会吞噬这个异常，线程会终止，但进程可继续运行，这一点非常特别，因为对于其他所有未处理的异常CLR都会结束进程。
* 3，当所有线程都离开要卸载应用程序域后，CLR遍历所有域的堆，将所有对所有引用卸载应用程序域里对象的代理对象设置一个标识(flag),这样当这些代理对象再访问已卸载程序域里的对象时会抛出`AppDomainUnloadedException`。
* 4，垃圾回收器会回收所有要卸载应用程序域里的对象，期间对象的`Finalize`会被调用以保证资源清理干源。
* 5，CLR恢复所有线程的执行，`AppDomain.Unload`的设用返回并继续执行接下来的代码，`AppDomain.Unload`的调用是同步的，当然调用也是有限时的，如果对`AppDomain.Unload`的调用在10秒内没有返回，CLR会抛出一个`CannotUnloadAppDomainException`的异常，并且应用程序域有可能被卸载也有可能未被卸载。

如果调用`AppDomain.Unload`的线程正好在被卸载的`AppDomain`中，那么CLR会新建一个线程序继续卸载，原线程会抛出一个`ThreadAbortException`并展开(unwind),新建的线程等待`AppDomain`的卸载，卸载成功后自动退出，如果卸载失败那么新线程处理`CannotUnloadAppDomainException`，因为我们并没有为新线程写这方面的代码，所以新线程无法捕获这个异常。

对象创建后两个线程先后在新建的应用程序域里运行，`_unload`线程卸载当前应用程序域，此时`_test`线程还在`Thread.Sleep`中，收到`ThreadAbortException`后到达`catch`块打钱出信息。

如果一个线程当前正在`finally`块中，`catch`块中，类构造器，临界执行区域或非托管代码中，CLR不会立即终止该线程，只有在代码跳出这些块时CLR才会向此线程抛出`ThreadAbortException`. 

## AppDomain的异常通知

**AppDomain.CurrentDomain.UnhandledException**

捕获应用程序域中未处理的异常，只是收到异常的通知并不能处理异常也不能吞噬掉异常最终异常还是会抛出，你可以在些事件处理函数中写日志或打印详细信息

**AppDomain.CurrentDomain.FirstChanceException**

异常首次抛出时的通知，只是接收到异常通知并不能吞噬掉异常，当异常首次抛出时，CLR会调用在此登记过的回调方法然后CLR会在当前`AppDomain`栈上查找能处理掉这次异常的catch块如果catch可以处理掉这个异常那么继续执行，如果当前`AppDomain`中没有一个catch块可以处理那么CLR会沿着栈向上查找来到调用`AppDomain`,这时调用`AppDomain`会接到一个新的异常(序列化原来的异常并在本`AppDomain`中反序化)，CLR此时会调用订阅了当前`AppDomain FirstChanceException` 的回调方法，然后异常查找过程依旧，直至栈顶如果没有catch块可以处理此异常那CLR将终止进程。

需要注意的是以上两种异常回调均可以在另一个应用程序域中被订阅，`UnhandleException`指的是CLR通过搜索线程栈里的catch块一直搜索到栈顶还未搜索到的一种事件，如果此时线程栈在另一个`AppDomain`中则算为另一个`AppDomain`的`UnhandleException`,也就是说线程栈的栈顶在哪个`AppDomain`中，就算做哪个`AppDomain`的`UnhandleExcetpion`。

任何一个线程中的异常不管激发那个`AppDomain`中的`UnhandleException`之后都将结束线程，当然也有例外，其它章节我们会介绍。

## AppDomain与线程的迷惑与理解

`AppDomain`是个静态的概念，线程是个动态的概念，一个线程可能运行于一个`AppDomain`中也可跨越多个`AppDomain`运行，CLR中的线程(`System.Threading.Thread`)其实是个`Soft Thread`，操作系统中的线程是`Hard Thread`并且操作系统只可识别`Hard Thread`,`Soft Thread`是CLR对`Hard Thread`的封装（在MSDN中的描述`Hard Thread`与`Soft Thread`没有一对一的对应关系，但就目前CLR的实现来说`Hard Thread`和`Soft Thread`是一对一的关系），`Soft Thread`只属于某个`AppDomain`,穿越`AppDomain`的是`Hard Thread`,当`Hard Thread`访问一个`AppDomain`时其会为之产生一个`Soft Thread`，`Hard Thread`中有个叫TLS的结构，这个区域被CLR用来存储`Hard Thread`当前执行代码的`AppDomain`引用以及`Soft Thread`的引用，当`Hard Thread`穿越到另一个`AppDomain`时，TLS中的这些引用也会随之改变。

参考：

[AppDomain 类](https://docs.microsoft.com/zh-cn/dotnet/api/system.appdomain?view=net-5.0)

[AssemblyLoadContext 类](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.loader.assemblyloadcontext?view=net-5.0)

[C#基础--应用程序域(Appdomain)](https://www.cnblogs.com/asminfo/p/3999412.html)

[在C#中使用AppDomain实现【插件式】开发](https://www.cnblogs.com/mq0036/p/14646523.html)