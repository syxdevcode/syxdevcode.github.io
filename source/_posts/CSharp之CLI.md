---
title: CSharp之CLI
date: 2019-07-02 17:34:09
tags:
- DotNet
- CSharp
- CSharp基础
- .Net Standard
categories: 
- .Net Standard
---
简介：CLI是一个开放的技术规范。它是由微软联合惠普以及英特尔于2000年向ECMA倡议的。通用语言基础架构定义了构成.NET Framework基础结构的可执行码以及代码的运行时环境的规范，它定义了一个语言无关的跨体系结构的运行环境，这使得开发者可以用规范内定义的各种高级语言来开发软件，并且无需修正即可将软件运行在不同的计算机体系结构上。CLI有时候会和CLR混用。但严格意义上说，这是错误的。因为CLI是一种规范，而CLR则是对这种规范的一个实现。

首先，引入一张图片:

![2010012620512419.png](/img/2010012620512419.png)
图片引用自：[https://www.cnblogs.com/eshizhan/archive/2010/01/26/1657041.html](https://www.cnblogs.com/eshizhan/archive/2010/01/26/1657041.html)

流程图：

![CLI-00001.png](/img/CLI-00001.png)

## CLI

<font color=#ff0000 size=4 face="黑体">CLI：通用语言基础架构(Common Language Infrastructure，CLI)；</font>

CLI是一个开放的技术规范。它是由微软联合惠普以及英特尔于2000年向ECMA倡议的。通用语言基础架构定义了构成.NET Framework基础结构的可执行码以及代码的运行时环境的规范，它定义了一个语言无关的跨体系结构的运行环境，这使得开发者可以用规范内定义的各种高级语言来开发软件，并且无需修正即可将软件运行在不同的计算机体系结构上。CLI有时候会和CLR混用。但严格意义上说，这是错误的。因为CLI是一种规范，而CLR则是对这种规范的一个实现。

欧洲计算机制造商协会（ECMA）已经于2001年10月13日批准C#语言规范（ECMA-334）成为一种新诞生的计算机产业标准。同时国际标准组织ISO也同意该标准进入该组织的审批阶段。并且，作为.NET与CLR的核心部分，CLI与C#也同时获得了ECMA的批准（ECMA-335）。拥有了C#与CLI这两项标准，你可以自己写出能够运行于任何操作系统上的.NET平台（只要你愿意）。如前所述，著名的MONO项目就是这么干的，MONO项目包括三个核心的部分：一个C#语言的编译器，一个CLI和一个类库。

## CLR

<font color=#ff0000 size=4 face="黑体">CLR：通用语言运行平台(Common Language Runtime，CLR)；</font>

无论通过任何语言构建产品，都必须寄宿到一个平台中运行，这正如我们的软件运行在操作系统环境一样，操作系统为CLR提供了运行环境，使用.NET构建的程序又运行在CLR之上，CRL为.NET程序的运行提供了温床，CLR提供基本的类库和运行引擎，基本类库封装操作系统函数供开发者方便调用，运行引擎用于编译并运行我们开发的程序。CLR包含.NET运行引擎和符合CLI的类库。通过.NET平台构建的程序都基于CLR基础类库来实现，并且运行在CLR提供的运行引擎之上。

编译为托管代码时，编译器将源代码翻译为 Microsoft 中间语言 (MSIL)，这是一组可以有效地转换为本机代码且独立于 CPU 的指令。MSIL 包括用于加载、存储和初始化对象以及对对象调用方法的指令，还包括用于算术和逻辑运算、控制流、直接内存访问、异常处理和其他操作的指令。要使代码可运行，必须先将 MSIL 转换为特定于 CPU 的代码，这通常是通过实时 (JIT) 编译器来完成的。由于公共语言运行库为它支持的每种计算机结构都提供了一种或多种 JIT 编译器，因此同一组 MSIL 可以在所支持的任何结构上 JIT 编译和运行。

## CIL

<font color=#ff0000 size=4 face="黑体">CIL：公共中间语言（Common Intermediate Language，CIL)；</font>

CLI，简称**微软中间语言（MSIL）或者中间语言（IL）**。CIL是编译器将.NET代码编译成公共语言运行时（CLR）能够识别的中间代码。它是一种介于高级语言（例如C#）和CPU指令之间的一种语言。当用户编译一个.NET程序时，编译器（例如VisualStudio）将C#源代码编译转换成中间语言 (MSIL)，它是一种能够被CLR转换成CPU指令的中间语言，当执行这些中间语言时，CLR提供的实时（JIT）编译器将它们转化为CPU特定的代码。由于公共语言运行库支持多种实时编译器，因此同一段中间语言代码可以被不同的编译器实时编译并运行在不同的CPU结构上。从理论上来说，MSIL将消除多年以来业界中不同语言之间的纷争。在.NET的世界中可能出现下面的情况一部分代码可以用C++实现，另一部分代码使用C#或VB.NET完成的，但是最后这些代码都将被转换为中间语言。这给程序员提供了极大的灵活性，程序员可以选择自己熟悉的语言，并且再也不用为学习不断推出的新语言而烦恼了。

## FCL

<font color=#ff0000 size=4 face="黑体">FCL：框架类库(Framework Class Library,FCL)；</font>

FCL提供了大粒度的编程框架，它是针对不同应用设计的框架 ，FCL大部分实现都引用了BCL，例如我们常说的开发框架：ASP.NET、MVC、WCF和WPF等等，提供了针对不同层面的编程框架 。

FCL框架类库可以划分为三层：

** 最内一层**：由BCL的大部分组成，主要作用是对.NET框架，.NET运行时以及CIL语言本身进行支持，例如基元类型，集合类型，线程处理，应用程序域，运行时，安全性，互操作等。
** 中间一层**：包含了对操作系统功能的封装，例如文件系统，网络连接，图形图像，XML操作等。
** 最外一层**：包含各种类型的应用程序，例如 Window Forms，Asp.NET，WPF，WCF，WF等。

## BCL

<font color=#ff0000 size=4 face="黑体">BCL：基类库(Base Class Library，BCL)；</font>

BCL是一个公共编程框架，称为基类库，所有语言的开发者都能利用它。是CLI（Common Language Infrastructure，公共语言基础结构）的规范之一，主要包括：执行网络操作，执行I/O操作，安全管理，文本操作，数据库操作，XML操作，与事件日志交互，跟踪和一些诊断操作，使用非托管代码，创建与调用动态代码等，粒度相对较小，为所有框架提供基础支持。

## CLS

<font color=#ff0000 size=4 face="黑体">CLS：公共语言规范(Common Language Specification，CLS);</font>

CLS是CTS的一个子集，所有.NET语言都应该遵循此规则才能创建与其他语言可互操作的应用程序，但要注意的是为了使各语言可以互操作，只能使用CLS所列出的功能对象，这些功能统称为与CLS兼容的功能，它是在.NET平台上运行语言的最小规范，正因为.NET上不同语言能够轻松交互一样，例如C#编写程序时可以直接引用并使用VB.NET编写的类库。为了达到这样的交互，才制定出CLS规范，在.NET框架本身提供的所有类库（并非所有）都是与CLS兼容的，在查看MSDN文档时，不兼容的类和方法都被特别标记为不兼容，例如C#中的System.UInt32就标记为”此API不兼容CLS。兼容 CLS的替代API为 Int64。“，这说明并不是所有的语言(例如VB.NET或J#)都支持无符号的数据类型，但这种数据类型与CLS不兼容的。

## CTS

<font color=#ff0000 size=4 face="黑体">CTS：公共类型系统(Common Type System，CTS);</font>

CTS是一种类型系统和语言规范，它能够确保CLR能够识别和处理的类型，所有.NET开发语言中的类型，无论时VS.NET类型还是C#.NET类型最终都会被编译成CLR能够识别的CTS类型，因此CTS是.NET平台类型的抽象。例如VB.NET中的integer类型和C#中的int类型都编译成CTS的System.Int32类型。如果某种语言编写的程序能够在CLR上运行，并不能说明这种语言完全符合CTS的规范。例如使用C++编写的代码，部分代码并不符合CTS规范，在编译时把这部分不符合CTS的代码会被编译成原始代码本地CPU指令而非中间代码。

参考：

[Microsoft Win32 to Microsoft .NET Framework API Map](https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/aa302340(v=msdn.10))

[Common Language Infrastructure](https://en.wikipedia.org/wiki/Common_Language_Infrastructure)

[Overview of the Common Language Infrastructure.svg](https://en.wikipedia.org/wiki/File:Overview_of_the_Common_Language_Infrastructure.svg)

[BCL和FCL](https://blog.csdn.net/shanyongxu/article/details/50879598)

[关于CLR、CIL、CTS、CLS、CLI、BCL和FCL 的区分与总结](https://blog.csdn.net/Eiceblue/article/details/46602343)
