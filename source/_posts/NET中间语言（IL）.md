---
title: .NET中间语言（IL）
date: 2020-01-08 10:10:34
tags: 
- DotNet
- CSharp
- MSIL指令
- Emit(反射发出)
categories: 
- MSIL指令
---
转载自：

[.NET中间语言（IL）1](https://docs.microsoft.com/zh-tw/previous-versions/dd229210%28v%3dmsdn.10%29)

[.NET中间语言（IL）2](https://docs.microsoft.com/zh-tw/previous-versions/dd229211%28v%3dmsdn.10%29)

.NET CLR 和 Java VM 都是堆栈式虚拟机（Stack-Based VM），也就是说，它们的指令集（Instruction Set）都是采用堆栈运算的方式：执行时的数据都是先放在堆栈中，再进行运算。 Java VM 有约 200 个指令（Instruction），每个指令都是 1 byte 的 opcode（操作码），后面接不等数目的参数；.NET CLR 有超过 220 个指令，但是有些指令使用相同的 opcode，所以 opcode 的 数目比指令数略少。 特别注意，.NET 的 opcode 长度并不固定，大部分的 opcode 长度是 1 byte，少部分是 2 byte。

本文章以一个实际的例子，让你了解堆栈式 VM 的运作原理，并对 .NET IL（Intermediate Language）有最基本的领略。

下面是一个简单的 C# 原始码：

```cs
using System;

public class Test {
    public static void Main(String[] args) {
        int i=1;
        int j=2;
        int k=3;
        int answer = i+j+k;
        Console.WriteLine("i+j+k="+answer);
    }
}
```

将此原始码编译之后，可以得到一个 EXE 档案。 我们可以透过 ILDASM. EXE 来反组译 EXE 以观察 IL。 我将 Main() 的 IL 反组译条列如下，这里共有十八道 IL 指令，有的指令（例如 ldstr 与 box）后面需要接参数，有的指令（例如 ldc.i4.1 与 add）后面不需要接参数。

```cs
ldc.i4.1
stloc.0
ldc.i4.2
stloc.1
ldc.i4.3
stloc.2
ldloc.0
ldloc.1
add
ldloc.2
add
stloc.3
ldstr      "i+j+k="
ldloc.3
box        [mscorlib]System.Int32
call       string [mscorlib]System.String::Concat(object, object)
call       void [mscorlib]System.Console::WriteLine(string)
ret
```

此程序执行时，关键的内存有三种，分别是：

* **Managed Heap：**这是动态配置（Dynamic Allocation）的内存，由 Garbage Collector（GC）在执行时自动管理，整个 Process 共享一个 Managed Heap。

* **Call Stack：**这是由 .NET CLR 在执行时自动管理的内存，每个 Thread 都有自己专属的 Call Stack。 每呼叫一次 method，就会使得 Call Stack 上多了一个 Record Frame；呼叫完毕之后，此 Record Frame 会被丢弃。 一般来说，Record Frame 内纪录着 method 参数（Parameter）、返回地址（Return Address）、以及局部变量（Local Variable）。 Java VM 和 .NET CLR 都是使用 0, 1, 2... 编号的方式来识别局部变量。

* **Evaluation Stack：**这是由 .NET CLR 在执行时自动管理的内存，每个 Thread 都有自己专属的 Evaluation Stack。 前面所谓的堆栈式虚拟机，指的就是这个堆栈。

后面有一连串的示意图，用来解说在执行时此三种内存的变化。 首先，在进入 Main() 之后，尚未执行任何指令之前，内存的状况如图 1 所示：

![dd229210.il_f1(zh-tw,msdn.10).jpg](/img/dd229210.il_f1(zh-tw,msdn.10).jpg)
图 1

接着要执行第一道指令 ldc.i4.1。 此指令的意思是：在 Evaluation Stack 置入一个 4 byte 的常数，其值为 1。 执行完此道指令之后，内存的变化如图 2 所示：

![dd229210.il_f2(zh-tw,msdn.10).jpg](/img/dd229210.il_f2(zh-tw,msdn.10).jpg)
图 2

接着要执行第二道指令 stloc.0。 此指令的意思是：从 Evaluation Stack 取出一个值，放到第 0 号变量（V0）中。 这里的第 0 号变量其实就是原始码中的 i。 执行完此道指令之后，内存的变化如图 3 所示：

![dd229210.il_f3(zh-tw,msdn.10).jpg](/img/dd229210.il_f3(zh-tw,msdn.10).jpg)
图 3

后面的第三道指令和第五道指令雷同于第一道指令，且第四道指令和第六道指令雷同于第二道指令。 为了节省篇幅，我不在此一一赘述。 提醒大家第 1 号变量（V1）其实就是原始码中的 j，且第 2 号变量（V2）其实就是源码中的 k。 图 4~7 分别是执行完第三~六道指令之后，内存的变化图：

![dd229210.il_f4(zh-tw,msdn.10).jpg](/img/dd229210.il_f4(zh-tw,msdn.10).jpg)
图 4

![dd229210.il_f5(zh-tw,msdn.10).jpg](/img/dd229210.il_f5(zh-tw,msdn.10).jpg)
图 5

![dd229210.il_f6(zh-tw,msdn.10).jpg](/img/dd229210.il_f6(zh-tw,msdn.10).jpg)
图 6

![dd229210.il_f7(zh-tw,msdn.10).jpg](/img/dd229210.il_f7(zh-tw,msdn.10).jpg)
图 7

接着要执行第七道指令 ldloc.0 以及第八道指令 ldloc.1：分别将 V0（也就是 i）和 V1（也就是 j）的值放到 Evaluation Stack，这是相加前的准备动作。 图 8 与图 9 分别是执行完第七、第八道指令之后，内存的变化图：

![dd229210.il_f8(zh-tw,msdn.10).jpg](/img/dd229210.il_f8(zh-tw,msdn.10).jpg)
图 8

![dd229210.il_f9(zh-tw,msdn.10).jpg](/img/dd229210.il_f9(zh-tw,msdn.10).jpg)
图 9

接着要执行第九道指令 add。 此指令的意思是：从 Evaluation Stack 取出两个值（也就是 i 和 j），相加之后将结果放回 Evaluation Stack 中。 执行完此道指令之后，内存的变化如图 10 所示：

![dd229210.il_f10(zh-tw,msdn.10).jpg](/img/dd229210.il_f10(zh-tw,msdn.10).jpg)
图 10

接着要执行第十道指令 ldloc.2。 此指令的意思是：分别将 V2（也就是 k）的值放到 Evaluation Stack，这是相加前的准备动作。 执行完此道指令之后，内存的变化如图 11 所示：

![dd229211.il_f11(zh-tw,msdn.10).jpg](/img/dd229211.il_f11(zh-tw,msdn.10).jpg)
图 11

接着要执行第十一道指令 add。 从 Evaluation Stack 取出两个值，相加之后将结果放回 Evaluation Stack 中，此为 i+j+k 的值。 执行完此道指令之后，内存的变化如图 12 所示：

![dd229211.il_f12(zh-tw,msdn.10).jpg](/img/dd229211.il_f12(zh-tw,msdn.10).jpg)
图 12

接着要执行第十二道指令 stloc.3。 从 Evaluation Stack 取出一个值，放到第 3 号变量（V3）中。 这里的第3号变量其实就是原始码中的 answer。 执行完此道指令之后，内存的变化如图 13 所示：

![dd229211.il_f13(zh-tw,msdn.10).jpg](/img/dd229211.il_f13(zh-tw,msdn.10).jpg)
图 13

接着要执行第十三道指令 ldstr "i+j+k="。 此指令的意思是：将 "i+j+k=" 的 Reference 放进 Evaluation Stack。 执行完此道指令之后，内存的变化如图 14 所示：

![dd229211.il_f14(zh-tw,msdn.10).jpg](/img/dd229211.il_f14(zh-tw,msdn.10).jpg)
图 14

接着要执行第十四道指令 ldloc.3。 将 V3 的值放进 Evaluation Stack。 执行完此道指令之后，内存的变化如图 15 所示：

![dd229211.il_f15(zh-tw,msdn.10).jpg](/img/dd229211.il_f15(zh-tw,msdn.10).jpg)
图 15

接着要执行第十五道指令 box [mscorlib]System.Int32。 此指令的意思是：从 Evaluation Stack 中取出一个值，将此 Value Type 包装（box）成为 Reference Type。 执行完此道指令之后，内存的变化如图 16 所示：

![dd229211.il_f16(zh-tw,msdn.10).jpg](/img/dd229211.il_f16(zh-tw,msdn.10).jpg)
图 16

接着要执行第十六道指令 call string [mscorlib] System.String::Concat(object, object)。 此指令的意思是：从 Evaluation Stack 中取出两个值，此二值皆为 Reference Type，下面的值当作第一个参数，上面的值当作第二个参数，呼叫 mscorlib.dll 所提供的 System.String.Concat() method 来将此二参数进行字符串接合（String Concatenation），将接合出来的新字符串放在 Managed Heap，将其 Reference 放进 Evaluation Stack。 值得注意的是：由于 System.String.Concat() 是 static method，所以此处使用的指令是 call，而非 callvirt（呼叫虚拟）。 执行完此道指令之后，内存的变化如图 17 所示：

![dd229211.il_f17(zh-tw,msdn.10).jpg](/img/dd229211.il_f17(zh-tw,msdn.10).jpg)
图 17

请注意：此时 Managed Heap 中的 Int32(6) 以及 String("i+j+k=") 已经不再被参考到，所以变成垃圾，等待 GC 的回收。

接着要执行第十七道指令 call void [mscorlib] System.Console::WriteLine(string)。 此指令的意思是：从 Evaluation Stack 中取出一个值，此值为 Reference Type，将此值当作参数，呼叫 mscorlib.dll 所提供的 System.Console.WriteLine() method 来将此字符串显示在 Console 窗口上。 System.Console.WriteLine() 也是 static method。 执行完此道指令之后，内存的变化如图 18 所示：

![dd229211.il_f18(zh-tw,msdn.10).jpg](/img/dd229211.il_f18(zh-tw,msdn.10).jpg)
图 18

接着要执行第十八道指令 ret。 此指令的意思是：结束此次呼叫（也就是 Main 的呼叫）。 此时会检查 Evaluation Stack 内剩下的数据，由于 Main() 宣告不需要传出值（void），所以 Evaluation Stack 内必须是空的，本范例符合这样的情况，所以此时可以顺利结束此次呼叫。 而 Main 的呼叫一结束，程序也随之结束。 执行完此道指令之后（且在程序结束前），内存的变化如图 19 所示：

![dd229211.il_f19(zh-tw,msdn.10).jpg](/img/dd229211.il_f19(zh-tw,msdn.10).jpg)
图 19

透过此范例，读者应该可以对于 IL 有最基本的认识。 对 IL 感兴趣的读者应该自行阅读 Serge Lidin 所著的《Inside Microsoft .NET IL Assembler》（Microsoft Press 出版）。 我认为：熟知 IL 每道指令的作用，是 .NET 程序员必备的知识。.NET 程序员可以不会用 IL Assembly 写程序，但是至少要看得懂 ILDASM 反组译出来的 IL 组合码。
