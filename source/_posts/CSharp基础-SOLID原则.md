---
title: CSharp基础-SOLID原则
date: 2019-01-10 12:04:25
tags:
- DotNet
- CSharp
- 设计模式
categories: 
- 设计模式
---
# SOLID原则

设计类和模块时，遵守 SOLID 原则可以让软件更加健壮和稳定。

简写|全称|翻译
---|-----|----
SRP|The Single Responsibility Principle|单一责任原则
OCP|The Open Closed Principle| 开放封闭原则
LSP| The Liskov Substitution Principle| 里氏替换原则
ISP |The Interface Segregation Principle| 接口分离原则
DIP| The Dependency Inversion Principle| 依赖倒置原则

## 单一职责原则（SRP）

单一职责原则（SRP）表明一个类有且只有一个职责。

## 开放封闭原则（OCP）

开放封闭原则（OCP）指出，一个类应该对扩展开放，对修改关闭。这意味一旦你创建了一个类并且应用程序的其他部分开始使用它，你不应该修改它。即，可扩展(extension)，不可修改(modification)。

## 里氏替换原则（LSP）

里氏替换原则指出，派生的子类应该是可替换基类的，也就是说任何基类可以出现的地方，子类一定可以出现。一个对象在其出现的任何地方，都可以用子类实例做替换，并且不会导致程序的错误。换句话说，当子类可以在任意地方替换基类且软件功能不受影响时，这种继承关系的建模才是合理的。当一个子类的实例应该能够替换任何其超类的实例时，它们之间才具有is-A关系。

## 接口隔离原则（ISP）

接口隔离原则（ISP）表明类不应该被迫依赖他们不使用的方法，也就是说一个接口应该拥有尽可能少的行为。将接口拆分成更小和更具体的接口，有助于解耦，从而更容易重构、更改。

## 依赖倒置原则（DIP）

依赖倒置原则（DIP）表明高层模块不应该依赖低层模块，相反，他们应该依赖抽象类或者接口。这个设计原则的亮点在于任何被DI框架注入的类很容易用mock对象进行测试和维护，因为对象创建代码集中在框架中，客户端代码也不混乱。

参考：

[浅谈 SOLID 原则的具体使用](http://www.cnblogs.com/OceanEyes/p/overview-of-solid-principles.html#_label4)

[SOLID原则](http://www.cnblogs.com/lanxuezaipiao/archive/2013/06/09/3128665.html)