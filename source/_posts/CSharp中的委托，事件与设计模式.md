---
title: CSharp中的委托，事件与设计模式
date: 2018-12-11 11:10:54
tags:
- 委托与事件
- 设计模式
- DotNet
- CSharp
- CSharp基础
categories: 
- 委托与事件
---
# CSharp中的委托，事件与设计模式

## Observer设计模式

假设我们有个高档的热水器，我们给它通上电，当水温超过95度的时候：1、扬声器会开始发出语音，告诉你水的温度；2、液晶屏也会改变水温的显示，来提示水已经快烧开了。

现在我们需要写个程序来模拟这个烧水的过程，我们将定义一个类来代表热水器，我们管它叫：Heater，它有代表水温的字段，叫做temperature；当然，还有必不可少的给水加热方法BoilWater()，一个发出语音警报的方法MakeAlert()，一个显示水温的方法，ShowMsg()。

我们先了解一下Observer设计模式，Observer设计模式中主要包括如下两类对象：

* 1,Subject：监视对象，它往往包含着其他对象所感兴趣的内容。在本范例中，热水器就是一个监视对象，它包含的其他对象所感兴趣的内容，就是temprature字段，当这个字段的值快到100时，会不断把数据发给监视它的对象。
* 2,Observer：监视者，它监视Subject，当Subject中的某件事发生的时候，会告知Observer，而Observer则会采取相应的行动。在本范例中，Observer有警报器和显示器，它们采取的行动分别是发出警报和显示水温。

在本例中，事情发生的顺序应该是这样的：

* 1,警报器和显示器告诉热水器，它对它的温度比较感兴趣(注册)。
* 2,热水器知道后保留对警报器和显示器的引用。
* 3,热水器进行烧水这一动作，当水温超过95度时，通过对警报器和显示器的引用，自动调用警报器的MakeAlert()方法、显示器的ShowMsg()方法。

Observer设计模式：Observer设计模式是为了定义对象间的一种一对多的依赖关系，以便于当一个对象的状态改变时，其他依赖于它的对象会被自动告知并更新。Observer模式是一种松耦合的设计模式。

## 实现范例的Observer设计模式

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace Delegate {
    // 热水器
    public class Heater {
       private int temperature;
       public delegate void BoilHandler(int param);   //声明委托
       public event BoilHandler BoilEvent;        //声明事件

       // 烧水
       public void BoilWater() {
           for (int i = 0; i <= 100; i++) {
              temperature = i;

              if (temperature > 95) {
                  if (BoilEvent != null) { //如果有对象注册
                      BoilEvent(temperature);  //调用所有注册对象的方法
                  }
              }
           }
       }
    }

    // 警报器
    public class Alarm {
       public void MakeAlert(int param) {
           Console.WriteLine("Alarm：嘀嘀嘀，水已经 {0} 度了：", param);
       }
    }

    // 显示器
    public class Display {
       public static void ShowMsg(int param) { //静态方法
           Console.WriteLine("Display：水快烧开了，当前温度：{0}度。", param);
       }
    }

    class Program {
       static void Main() {
           Heater heater = new Heater();
           Alarm alarm = new Alarm();

           heater.BoilEvent += alarm.MakeAlert;    //注册方法
           heater.BoilEvent += (new Alarm()).MakeAlert;   //给匿名对象注册方法
           heater.BoilEvent += Display.ShowMsg;       //注册静态方法

           heater.BoilWater();   //烧水，会自动调用注册过对象的方法
       }
    }
}
输出为：
Alarm：嘀嘀嘀，水已经 96 度了：
Alarm：嘀嘀嘀，水已经 96 度了：
Display：水快烧开了，当前温度：96度。
// 省略...
```

## .Net Framework中的委托与事件

尽管上面的范例很好地完成了我们想要完成的工作，但是我们不仅疑惑：为什么.Net Framework 中的事件模型和上面的不同？为什么有很多的EventArgs参数？

在回答上面的问题之前，我们先搞懂 .Net Framework的编码规范：

* 委托类型的名称都应该以EventHandler结束。
* 委托的原型定义：有一个void返回值，并接受两个输入参数：一个Object 类型，一个 EventArgs类型(或继承自EventArgs)。
* 事件的命名为 委托去掉 EventHandler之后剩余的部分。
* 继承自EventArgs的类型应该以EventArgs结尾。

再做一下说明：

* 1,委托声明原型中的Object类型的参数代表了Subject，也就是监视对象，在本例中是 Heater(热水器)。回调函数(比如Alarm的MakeAlert)可以通过它访问触发事件的对象(Heater)。
* 2,EventArgs 对象包含了Observer所感兴趣的数据，在本例中是temperature。

** 上面这些其实不仅仅是为了编码规范而已，这样也使得程序有更大的灵活性**。比如说，如果我们不光想获得热水器的温度，还想在Observer端(警报器或者显示器)方法中获得它的生产日期、型号、价格，那么委托和方法的声明都会变得很麻烦，而如果我们将热水器的引用传给警报器的方法，就可以在方法中直接访问热水器了。

```csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace Delegate {
    // 热水器
    public class Heater {
       private int temperature;
       public string type = "RealFire 001";       // 添加型号作为演示
       public string area = "China Xian";         // 添加产地作为演示
       //声明委托
       public delegate void BoiledEventHandler(Object sender, BoiledEventArgs e);
       public event BoiledEventHandler Boiled; //声明事件

       // 定义BoiledEventArgs类，传递给Observer所感兴趣的信息
       public class BoiledEventArgs : EventArgs {
           public readonly int temperature;
           public BoiledEventArgs(int temperature) {
              this.temperature = temperature;
           }
       }

       // 可以供继承自 Heater 的类重写，以便继承类拒绝其他对象对它的监视
       protected virtual void OnBoiled(BoiledEventArgs e) {
           if (Boiled != null) { // 如果有对象注册
              Boiled(this, e);  // 调用所有注册对象的方法
           }
       }

       // 烧水。
       public void BoilWater() {
           for (int i = 0; i <= 100; i++) {
              temperature = i;
              if (temperature > 95) {
                  //建立BoiledEventArgs 对象。
                  BoiledEventArgs e = new BoiledEventArgs(temperature);
                  OnBoiled(e);  // 调用 OnBolied方法
              }
           }
       }
    }

    // 警报器
    public class Alarm {
       public void MakeAlert(Object sender, Heater.BoiledEventArgs e) {
           Heater heater = (Heater)sender;     //这里是不是很熟悉呢？
           //访问 sender 中的公共字段
           Console.WriteLine("Alarm：{0} - {1}: ", heater.area, heater.type);
           Console.WriteLine("Alarm: 嘀嘀嘀，水已经 {0} 度了：", e.temperature);
           Console.WriteLine();
       }
    }

    // 显示器
    public class Display {
       public static void ShowMsg(Object sender, Heater.BoiledEventArgs e) {   //静态方法
           Heater heater = (Heater)sender;
           Console.WriteLine("Display：{0} - {1}: ", heater.area, heater.type);
           Console.WriteLine("Display：水快烧开了，当前温度：{0}度。", e.temperature);
           Console.WriteLine();
       }
    }

    class Program {
       static void Main() {
           Heater heater = new Heater();
           Alarm alarm = new Alarm();

           heater.Boiled += alarm.MakeAlert;   //注册方法
           heater.Boiled += (new Alarm()).MakeAlert;      //给匿名对象注册方法
           heater.Boiled += new Heater.BoiledEventHandler(alarm.MakeAlert);    //也可以这么注册
           heater.Boiled += Display.ShowMsg;       //注册静态方法

           heater.BoilWater();   //烧水，会自动调用注册过对象的方法
       }
    }
}

输出为：
Alarm：China Xian - RealFire 001:
Alarm: 嘀嘀嘀，水已经 96 度了：
Alarm：China Xian - RealFire 001:
Alarm: 嘀嘀嘀，水已经 96 度了：
Alarm：China Xian - RealFire 001:
Alarm: 嘀嘀嘀，水已经 96 度了：
Display：China Xian - RealFire 001:
Display：水快烧开了，当前温度：96度。
// 省略 ...
```

参考：

[C#中的委托和事件](http://www.cnblogs.com/jimmyzhang/archive/2007/09/23/903360.html)
