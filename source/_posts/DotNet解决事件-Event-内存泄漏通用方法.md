---
title: DotNet解决事件(Event)内存泄漏通用方法
date: 2018-12-13 15:05:14
tags:
- DotNet
- CSharp
- 内存泄漏(Memory Leak)
- 委托与事件
- CSharp基础
categories: 
- 内存泄漏
---
# DotNet解决事件(Event)内存泄漏通用方法

&emsp;&emsp;一个生命周期较短的对象（对象A）注册到一个生命周期较长（对象B）的某个事件（Event）上，两者便无形之间建立一个引用关系（B引用A）。这种引用关系导致GC在进行垃圾回收的时候不会将A是为垃圾对象，最终使其常驻内存（或者说将A捆绑到B上，具有了和B一样的生命周期）。这种让无用的对象不能被GC垃圾回收的现象，在托管环境下就是一种典型的内存泄漏问题。

[代码下载](https://pan.baidu.com/s/1_zrGMRwiyB6Kky8t5kgFlA)
提取码：xazz

<font color=#0099ff size=4 face="黑体">事实上，.NET内存泄漏的最普遍原因是静态变量引用了对象。</font>

## 造成事件(Event)内存泄漏的原因

事件本质上就是一个System.Delegate对象。
Delegate分解成两个部分：委托的事情和委托的对象。与之相似地，.NET的Delegate对象同样可以分解成两个部分：委托的功能（Method）和目标对象（Target），这可以直接从Delegate的定义就可以看出来：

```csharp
public abstract class Delegate : ICloneable, ISerializable
{
    // Others
    public MethodInfo Method { get; }
    public object Target { get; }
}
```

常用的事件处理类型`EventHandler`和`EventHandler<TEventArgs>`本质上就是一个Delegate。

他们继承自`System.MulticastDelegate`，`MulticastDelegate`派生于`Delegate`。

```csharp
[Serializable, ComVisible(true)]
public delegate void EventHandler(object sender, EventArgs e);

[Serializable]
public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e) where TEventArgs: EventArgs;
```

&emsp;&emsp;经过简单一句事件注册代码就通过一个EventHandler（EventHandler<TodoListArgs>）事件的源（TodoListManager）和事件的监听者（TodoListForm）两着关联起来，三者之间的关系如下图所示。从这张图中我们可以看到：TodoListForm实际上是通过注册的EventHandler的Target属性被TodoListManager间接引用着的。所以才会导致TodoListForm在关闭之后，永远不能拿成为垃圾对象，因为TodoListManager是一个基于static属性定义的Singleton对象，永远是GC的根。

![image_thumb_2.png](/img/image_thumb_2.png)

## 解决方法

&emsp;&emsp;当对象A注册到B的某个事件上，A并不受到B的“强制引用”。既然不能“强引用（Strong Reference）”，那就只能是“弱引用（Weak Reference）”。通过`System.WeakReference`来解决这个问题。

方法：采取某种机制，让事件源（Event Source）的EventHandler通过WeakReference的方式与事件监听者建立关系。只有在这种情况下，事件监听者没有了事件源的强制引用，在我们不用的时候才能及时成为垃圾对象，等待GC对它的清理。

![image_thumb_3.png](/img/image_thumb_3.png)

&emsp;&emsp;通过传入`EventHandler<TEventArgs>`对象构造WeakReferenceHandler，在EventHandler<TEventArgs>的Target属性基础上建立WeakReference对象，在执行处理事件的时候通过该WeakReference找到真正的目标对象，如果找得到则通过反射在其基础上调用相应的方法；反之，如果通过不能得到Target，那么表明该事件的监听对象已经被GC当作垃圾对象回收掉了。为了在注册事件的时候方遍，特定义了一个隐式的类型转换：WeakReferenceHandler转换成EventHandler<TEventArgs>。

```csharp
using System;
using System.Reflection;
namespace Artech.MemLeakByEvents
{
    public class WeakEventHandler<TEventArgs> where TEventArgs : EventArgs
    {
        public WeakReference Reference
        { get; private set; }

        public MethodInfo Method
        { get; private set; }

        public EventHandler<TEventArgs> Handler
         { get; private set; }

        public WeakEventHandler(EventHandler<TEventArgs> eventHandler)
        {
            Reference = new WeakReference(eventHandler.Target);
            Method = eventHandler.Method;
            Handler = Invoke;
        }

        public void Invoke(object sender, TEventArgs e)
        {
            object target = Reference.Target;
            if (null != target)
            {
                Method.Invoke(target, new object[] { sender, e });
            }
        }

        /// <summary>
        /// 隐式转换
        /// </summary>
        /// <param name="weakHandler"></param>
        public static implicit operator EventHandler<TEventArgs>(WeakEventHandler<TEventArgs> weakHandler)
        {
            return weakHandler.Handler;
        }
    }
}
```

实际进行事件注册:

```csharp
private void TodoListForm_Load(object sender, EventArgs e)
{
    SynchronizationContext = SynchronizationContext.Current;
    TodoListManager.Instance.TodoListChanged += new WeakEventHandler<TodoListEventArgs>(TodoListManager_TodoListChanged);
}
```

参考：

[事件(Event)，绝大多数内存泄漏（Memory Leak）的元凶[下篇] （提供Source Code下载）](http://www.cnblogs.com/artech/archive/2009/12/06/1618239.html)