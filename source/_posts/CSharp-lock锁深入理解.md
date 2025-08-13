---
title: CSharp lock锁深入理解
date: 2017-05-02 10:41:19
tags: 
- CSharp
- DotNet
- CSharp基础
categories: 
- CSharp基础
---
# CSharp lock锁深入理解

## 定义

### 临界区（Critical Section）

临界区是一段在同一时候只被一个线程进入/执行的代码块。

### Lock关键字

将语句块标记为临界区，方法是获取给定对象的互斥锁，执行语句，然后释放该锁。此语句的形式如下：

````csharp
Object thisLock = new Object();

lock (thisLock)
{
    // Critical code section
}
````

* lock 确保当一个线程位于代码的临界区时，另一个线程不进入临界区。如果其他线程试图进入锁定的代码，则它将一直等待（即被阻止），直到该对象被释放。

### Lock使用准则

Lock 调用块开始位置的 Enter 和块结束位置的 Exit。

通常，应避免锁定 public 类型，否则实例将超出代码的控制范围。常见的结构 lock (this)、lock (typeof (MyType)) 和 lock ("myLock") 违反此准则：

* 如果实例可以被公共访问，将出现 lock (this) 问题。

* 如果 MyType 可以被公共访问，将出现 lock (typeof (MyType)) 问题。

* 由于进程中使用同一字符串的任何其他代码将共享同一个锁，所以出现lock(“myLock”) 问题。

最佳做法是定义 private 对象来锁定, 或 private static 对象变量来保护所有实例所共有的数据。

针对上述三点原则我们用具体的代码来加深理解

#### 为什么不要Lock值类型

为什么不能lock值类型，比如lock(1)呢？让我们对上面的实例代码进行修改,将lock(thisLock)改为Lock(1),编译器时会出现int不是Lock语句要求的引用对象

![编译失败](/img/lock1.png)

lock本质上Monitor.Enter，Monitor.Enter会使值类型装箱，每次lock的是装箱后的对象。lock其实是类似编译器的语法糖，因此编译器直接限制住不能lock值类型。

#### 不要使用lock((object)1)

经lock(thisLock)改为Lock((object)1)编译可以成功，说明代码块并没有锁.究其原因是因为Lock(对象)中,对象为引用类型,其是通过判断object.ReferenceEquals((object)1, (object)1)始终返回false（因为每次装箱后都是不同对象）,也就是说每次都会判断成未申请互斥锁，这样在同一时间，别的线程照样能够访问里面的代码，达不到同步的效果。

#### 不要使用Lock(this)

````csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;

namespace TestLock
{
    public class TestLockThis
    {
        private bool deadlocked = true;
        private static readonly object lockobject = new object();

        //这个方法用到了lock，我们希望lock的代码在同一时刻只能由一个线程访问
        public void LockMethod(object o)
        {
            //lock (this)
            lock (lockobject)
            {
                while (deadlocked)
                {
                    deadlocked = (bool)o;
                    Console.WriteLine("I am locked ");
                    Thread.Sleep(500);
                }
            }
        }

        //所有线程都可以同时访问的方法
        public void NotLockMethod()
        {
            Console.WriteLine("I am not locked ");
        }

    }
}
        static void Main(string[] args)
        {
            TestLockThis TestLockThis = new TestLockThis();

            //在t1线程中调用LockMe，并将deadlock设为true（将出现死锁）
            Thread t1 = new Thread(TestLockThis.LockMethod);
            t1.Start(true);
            Thread.Sleep(100);

            //在主线程中lock c1
            lock (TestLockThis)
            {
                //调用没有被lock的方法
                TestLockThis.NotLockMethod();
                //调用被lock的方法，并试图将deadlock解除
                TestLockThis.LockMethod(false);
            }

            Console.Read();
        }
````

在t1线程中，LockMethod调用了lock(this), 也就是Main函数中的c1，这时候在主线程中调用lock(TestLockThis)时，因为TestLockThis已经被锁定，所以必须要等待t1中的lock块执行完毕之后才能访问锁定的代码，即lock块中的所有操作都无法完成，于是我们看到连TestLockThis.NotLockMethod()都没有执行。

#### Lock为什么不要Lock(null对象)

使用Lock(null),会抛出异常

#### Lock 本质

Lock 关键字就是用Monitor 类来实现的。例如：

```csharp
lock(x)
{
  DoSomething();
}
```

这等效于：

```csharp
System.Object obj = (System.Object)x;
System.Threading.Monitor.Enter(obj);
try
{
  DoSomething();
}
finally
{
  System.Threading.Monitor.Exit(obj);
}
```

使用 lock 关键字通常比直接使用 Monitor 类更可取，一方面是因为 lock 更简洁，另一方面是因为 lock 确保了即使受保护的代码引发异常，也可以释放监视器。这是通过 finally 关键字来实现的，无论是否引发异常它都执行关联的代码块。

#### Lock 对象推荐为只读静态对象

```csharp
private static readonly object obj = new object();
```

为什么要设置成只读的呢？这是因为如果在lock代码段中改变obj的值，其它线程就畅通无阻了，因为互斥锁的对象变了，object.ReferenceEquals必然返回false。

更准确的说，如果lock(object），object是静态的。那么说要该类的实例请求的都是一个object，所有实例执行的lock的时候只有一个能running。其他都必须等等。
lock(this)则表示锁住当前实例，实例内的所有线程必须等待，同一类的其他实例则可以有一个线程runing.

简单说lock(static objcet)在所有实例里锁住这段代码。
lock（this)锁住当前实例的这段代码。

参考：

[http://www.cnblogs.com/chengqscjh/archive/2010/12/12/1903784.html](http://www.cnblogs.com/chengqscjh/archive/2010/12/12/1903784.html "深入理解CSharp Lock锁")