---
title: DotNet面试题解析06-委托
date: 2018-12-12 16:13:15
tags:
- DotNet
- CSharp
- 委托与事件
- CSharp基础
- DotNet面试题解析
categories: 
- DotNet面试题解析
---
## 1.委托是什么

什么是委托？简单来说，委托类似于 C或 C++中的函数指针，允许将方法作为参数进行传递。

## 2.为什么需要委托

在很多场景下直接调用方法是比较简单方便的，但是在某些场景下，使用委托来调用方法能达到减少代码量

## 3.委托能用来做什么

1,启动线程和任务

Thread t = new Thread(new ThreadStart(Go));//public static GO(){}

2,设计模式中的简单工厂模式。

向一个方法中传递一个子类的方法。

3,事件。

## 4.如何自定义委托

```csharp
public delegate void Feedback(int num);
```

.Net的委托本质上就是指向函数的指针，只不过这种指针是经过封装后类型安全的。委托和线程是两个不同的概念，线程是动态的，委托就是一个或一组内存地址，是静态的。线程执行时如果遇到了指向函数的指针就执行这个函数。

## 5..NET默认的委托类型有哪几种

* 1，Action<T>:无返回值

* 2，Func<T>:有返回值

## 6.多播委托是什么

包含多个方法的委托叫做多播委托。

多播委托的签名就必须返回void；否则，就只能得到委托调用的最后一个方法的结果。

```csharp
//----多播委托-------
Feedback fbChain = null;
//将feedback1添加到fbChain委托中
fbChain += feedback1;
//将feedback2添加到fbChain委托中
fbChain += feedback2;
//输出:
//2
//4
fbChain(2);
```

## 7.什么是泛型委托

Action<T>,Func<T>等

## 8.什么事匿名方法

匿名方法是用作委托的参数的一段代码。

参考：

[不惧面试：委托](http://www.cnblogs.com/jackson0714/p/5111347.html)