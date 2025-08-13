---
title: CSharp中的委托与事件
date: 2018-12-11 08:41:21
tags:
- 委托与事件
- DotNet
- CSharp
- CSharp基础
categories: 
- 委托与事件
---

## 委托

### 将方法作为方法的参数

委托在编译的时候确实会编译成类。因为Delegate是一个类，所以在任何可以声明类的地方都可以声明委托。

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace Delegate {
     //定义委托，它定义了可以代表的方法的类型
     public delegate void GreetingDelegate(string name);
        class Program {

           private static void EnglishGreeting(string name) {
               Console.WriteLine("Morning, " + name);
           }

           private static void ChineseGreeting(string name) {
               Console.WriteLine("早上好, " + name);
           }

           //注意此方法，它接受一个GreetingDelegate类型的方法作为参数
           private static void GreetPeople(string name, GreetingDelegate MakeGreeting) {
               MakeGreeting(name);
            }

           static void Main(string[] args) {
               GreetPeople("Jimmy Zhang", EnglishGreeting);
               GreetPeople("张子阳", ChineseGreeting);
               Console.ReadKey();
           }
        }
    }

输出如下：
Morning, Jimmy Zhang
早上好, 张子阳
```

我们现在对委托做一个总结：

<font color=#0099ff size=4 face="黑体">
委托是一个类，它定义了方法的类型，使得可以将方法当作另一个方法的参数来进行传递，这种将方法动态地赋给参数的做法，可以避免在程序中大量使用If-Else(Switch)语句，同时使得程序具有更好的可扩展性。</font>

### 将方法绑定到委托

<font color=#0099ff size=4 face="黑体">
可以将多个方法赋给同一个委托，或者叫将多个方法绑定到同一个委托，当调用这个委托的时候，将依次调用其所绑定的方法。</font>

```csharp
static void Main(string[] args) {
    GreetingDelegate delegate1;
    delegate1 = EnglishGreeting; // 先给委托类型的变量赋值
    delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法

     // 将先后调用 EnglishGreeting 与 ChineseGreeting 方法
    GreetPeople("Jimmy Zhang", delegate1);  
    Console.ReadKey();
}

输出为：
Morning, Jimmy Zhang
早上好, Jimmy Zhang
```

实际上，我们可以也可以绕过GreetPeople方法，通过委托来直接调用EnglishGreeting和ChineseGreeting：

```csharp
static void Main(string[] args) {
    GreetingDelegate delegate1;
    delegate1 = EnglishGreeting; // 先给委托类型的变量赋值
    delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法

    // 将先后调用 EnglishGreeting 与 ChineseGreeting 方法
    delegate1 ("Jimmy Zhang");
    Console.ReadKey();
}
```

<font color=#0099ff size=4 face="黑体">
注意这里，第一次用的“=”，是赋值的语法；第二次，用的是“+=”，是绑定的语法。如果第一次就使用“+=”，将出现“使用了未赋值的局部变量”的编译错误。</font>

我们也可以使用下面的代码来这样简化这一过程：

```csharp
GreetingDelegate delegate1 = new GreetingDelegate(EnglishGreeting);
delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法
```

如下，这样会出现编译错误： “GreetingDelegate”方法没有采用“0”个参数的重载。尽管这样的结果让我们觉得有点沮丧，但是编译的提示：“没有0个参数的重载”再次让我们联想到了类的构造函数。

```csharp
GreetingDelegate delegate1 = new GreetingDelegate();
delegate1 += EnglishGreeting;   // 这次用的是 “+=”，绑定语法。
delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法
```

既然给委托可以绑定一个方法，那么也应该有办法取消对方法的绑定，很容易想到，这个语法是“-=”：

```csharp
static void Main(string[] args) {
    GreetingDelegate delegate1 = new GreetingDelegate(EnglishGreeting);
    delegate1 += ChineseGreeting;   // 给此委托变量再绑定一个方法

    // 将先后调用 EnglishGreeting 与 ChineseGreeting 方法
    GreetPeople("Jimmy Zhang", delegate1);  
    Console.WriteLine();

    delegate1 -= EnglishGreeting; //取消对EnglishGreeting方法的绑定
    // 将仅调用 ChineseGreeting 
    GreetPeople("张子阳", delegate1);
    Console.ReadKey();
}
输出为：
Morning, Jimmy Zhang
早上好, Jimmy Zhang
早上好, 张子阳
```

让我们再次对委托作个总结：
<font color=#0099ff size=4 face="黑体">
使用委托可以将多个方法绑定到同一个委托变量，当调用此变量时(这里用“调用”这个词，是因为此变量代表一个方法)，可以依次调用所有绑定的方法。</font>

## 事件

Event：在类的内部，不管你声明它是public还是protected，它总是private的。在类的外部，注册“+=”和注销“-=”的访问限定符与你在声明事件时使用的访问符相同。

```csharp
public class GreetingManager{
    //这一次我们在这里声明一个事件
    public event GreetingDelegate MakeGreet;

    public void GreetPeople(string name) {
        MakeGreet(name);
    }
}
```

很容易注意到：MakeGreet 事件的声明与之前委托变量delegate1的声明唯一的区别是多了一个event关键字。看到这里，在结合上面的讲解，你应该明白到：<font color=#0099ff size=4 face="黑体">事件其实没什么不好理解的，声明一个事件不过类似于声明一个进行了封装的委托类型的变量而已。</font>

为了证明上面的推论，如果我们像下面这样改写Main方法：

```csharp
static void Main(string[] args) {
    GreetingManager gm = new  GreetingManager();
    gm.MakeGreet = EnglishGreeting;         // 编译错误1
    gm.MakeGreet += ChineseGreeting;

    gm.GreetPeople("Jimmy Zhang");
}
```

## 事件和委托的编译代码

这时候，我们注释掉编译错误的行，然后重新进行编译，再借助Reflactor来对 event的声明语句做一探究，看看为什么会发生这样的错误：

```csharp
public event GreetingDelegate MakeGreet;
```

![01.gif](/img/01.gif)

可以看到，实际上尽管我们在GreetingManager里将 MakeGreet 声明为public，但是，实际上MakeGreet会被编译成 私有字段，难怪会发生上面的编译错误了，因为它根本就不允许在GreetingManager类的外面以赋值的方式访问，从而验证了我们上面所做的推论。

我们再进一步看下MakeGreet所产生的代码：

```csharp
private GreetingDelegate MakeGreet; //对事件的声明 实际是 声明一个私有的委托变量

[MethodImpl(MethodImplOptions.Synchronized)]
public void add_MakeGreet(GreetingDelegate value){
    this.MakeGreet = (GreetingDelegate) Delegate.Combine(this.MakeGreet, value);
}

[MethodImpl(MethodImplOptions.Synchronized)]
public void remove_MakeGreet(GreetingDelegate value){
    this.MakeGreet = (GreetingDelegate) Delegate.Remove(this.MakeGreet, value);
}
```

现在已经很明确了：MakeGreet事件确实是一个GreetingDelegate类型的委托，只不过不管是不是声明为public，它总是被声明为private。另外，它还有两个方法，分别是add_MakeGreet和remove_MakeGreet，这两个方法分别用于注册委托类型的方法和取消注册。实际上也就是： “+= ”对应 add_MakeGreet，“-=”对应remove_MakeGreet。而这两个方法的访问限制取决于声明事件时的访问限制符。

在add_MakeGreet()方法内部，实际上调用了System.Delegate的Combine()静态方法，这个方法用于将当前的变量添加到委托链表中。我们前面提到过两次，说委托实际上是一个类，在我们定义委托的时候：

```csharp
public delegate void GreetingDelegate(string name);
```

当编译器遇到这段代码的时候，会生成下面这样一个完整的类：

```csharp
public sealed class GreetingDelegate:System.MulticastDelegate{
    public GreetingDelegate(object @object, IntPtr method);
    public virtual IAsyncResult BeginInvoke(string name, AsyncCallback callback, object @object);
    public virtual void EndInvoke(IAsyncResult result);
    public virtual void Invoke(string name);
}
```

![02.gif](/img/02.gif)

参考：

[C#中的委托和事件](http://www.cnblogs.com/jimmyzhang/archive/2007/09/23/903360.html)