---
title: WinDbg快速分析异常情况Dump文件
date: 2017-12-04 14:26:29
tags:
- WinDbg
- 异常诊断
categories:
- WinDbg
---
# WinDbg快速分析异常情况Dump文件

生产环境偶尔会出现一些异常问题，WinDbg 或 GDB 就是解决此类问题的利器。调试工具 WinDbg 如同医生的听诊器，是系统生病时做问题诊断的逆向分析工具，Dump 文件类似于飞机的黑匣子，记录着生产环境程序运行的状态。
本文主要介绍了调试工具 WinDbg 和抓包工具 ProcDump 的使用。

## 一、简介

### 1、WinDbg

WinDbg 是在 Windows 平台下的、强大的用户态和内核态调试工具。相比较于 Visual Studio，它是一个轻量级的调试工具，所谓轻量级指的是它的安装文件大小较小，但是其调试功能，却比 VS 更为强大。
它的另外一个用途是可以用来分析 Dump 数据。WinDbg 是 Microsoft 公司免费调试器调试集合中的 GUI 的调试器，支持 Source 和 Assembly 两种模式的调试。
WinDbg 不仅可以调试应用程序，还可以进行 Kernel Debug。结合 Microsoft 的 Symbol Server，可以获取系统符号文件，便于应用程序和内核的调试。
WinDbg 支持的平台包括 x86、IA64、AMD64。虽然 WinDbg 也提供图形界面操作，但它最强大的地方还是有着强大的调试命令，一般情况会结合 GUI 和命令行进行操作，常用的视图有：局部变量、全局变量、调用栈、线程、命令、寄存器、白板等。其中“命令”视图是默认打开的。

### 2、DebugDiag

DebugDiag 最初是为了帮助分析 IIS 的性能问题而开发的，它同样可以用于任何其他的进程。DebugDiag 工具主要用于帮助解决如挂起、 速度慢、 内存泄漏或内存碎片，和任何用户模式进程崩溃等问题。
该工具包括附加调试脚本，侧重于互联网信息服务（IIS）应用程序、 Web 数据访问组件、 COM+ 和相关 Microsoft 技术、SharePoint 和 .NET。它提供可扩展对象模型中的 COM 对象的形式，并具有一个内置的报告框架提供的脚本主机。它由 3 部分组成，包括调试服务、 调试器主机和用户界面。

### 3、ProcDump

ProcDump 是 System Internal 提供的一个专门用来监测程序 CPU 高使用率从而生成进程 Dump 文件的工具。ProcDump 可以根据系统的 CPU 使用率或者指定的性能计数器来针对特定进程生成一系列的 Dump 文件，以便调试者对事故原因进行分析。

## 二，工具下载地址

1、WinDbg 下载

x86 位版本下载：http://download.microsoft.com/download/A/6/A/A6AC035D-DA3F-4F0C-ADA4-37C8E5D34E3D/setup/WinSDKDebuggingTools/dbg_x86.msi

x64 位版本下载：http://download.microsoft.com/download/A/6/A/A6AC035D-DA3F-4F0C-ADA4-37C8E5D34E3D/setup/WinSDKDebuggingTools_amd64/dbg_amd64.msi

2、DebugDiag v2 Update 2 下载：https://www.microsoft.com/en-us/download/details.aspx?id=49924

3、ProcDump v9.0 下载：https://download.sysinternals.com/files/Procdump.zip

## 三、获取异常进程的 Dump 文件

以下四种方式获取 Dump 文件：

### 1、通过【任务管理器】获取 Dump 文件，这样获取的是 MinDump

![通过任务管理器](/img/MinDump.png)

### 2、利用 WinDbg 的 adplus 获取 Dump 文件，这样获取的是 FullDump

C:\Program Files\Debugging Tools for Windows (x64)>adplus -hang -pn explorer.exe
 -o D:\dumps

![WinDbg-1](/img/windbg-1.png)

### 3、通过 DebugDiag 创建.NET 异常转储 Dump 文件

![debugdiag-2](/img/debugdiag-2.png)

### 4、通过 ProcDump 抓取异常线程 Dump 文件。

#### 4.1、命令行

进入ProcDump目录，运行命令行：

````cmd
procdump [-a] [[-c|-cl CPU usage] [-u] [-s seconds]] [-n exceeds] 
[-e [1 [-b]] [-f <filter,...>] [-g] [-h] [-l] [-m|-ml commit usage] 
[-ma | -mp] [-o] [-p|-pl counter threshold] [-r] [-t] 
[-d <callback DLL>] [-64] <[-w] <process name or service name or PID>
[dump file] | -i <dump file> | -u | -x <dump file> <image file> 
[arguments] >] [-? [ -e]]
````

实例：

```WinDbg
procdump -c 70 -s 5 -ma -n 3 w3wp
  - 当系统 CPU 使用率持续 5 秒超过 70% 时，连续抓 3 个 Full Dump。
procdump outlook -p "\Processor(_Total)\% Processor Time" 80
  - 当系统 CPU 使用率超过 80%，抓取 Outlook 进程的 Mini Dump。
procdump -ma outlook -p "\Process(Outlook)\Handle Count" 10000
  - 当 Outlook 进程 Handle 数超过 10000 时抓取 Full Dump
procdump -ma 4572
  - 直接生成进程号位 4572 的 Full Dump。
```

注意：

* ProcDump 需要进程已经启动，并且中途不能停止。比如需要抓取 IIS Worker Process 的 High CPU Dump，由于 IIS Worker Process 默认会

配置 Idle Timeout = 20 min，即该进程在 20 分钟内没有任何请求的话就会自动结束，这种情况下 ProcDump 也会自动结束。需要重新运行命令。

因此如果目标程序存在这样的配置，需要暂时将该配置取消。

* 有些系统管理员希望能够运行该工具后退出用户 session，ProcDump 是做不到的，如果有这种需求可以考虑使用 DebugDiag。

* 在调试 High CPU 问题的时候经常用到的一个命令是!runaway，但是有些时候!runway 在 ProcDump 抓取 Dump 文件的过程中运行不出来，报错信息如下：

0:000> !runaway ERROR: !runaway: extension exception 0x80004002. "Unable to get thread times - dumps may not have time information"

解决方法是将 Debugging Tools for Windows (WinDbg) 安装目录下的 dbghelp.dll 拷贝到 procdump.exe 所在目录下，然后再运行命令抓取 Dump。

## 四、WinDbg 使用方法

操作步骤如下：

### 1、抓取异常程序的 Dump 文件。

### 2、设置符号表

符号表是 WinDbg 关键的“数据库”，如果没有它，WinDbg 基本上就是个废物，无法分析更多问题。所以使用 WinDbg 设置符号表，是必须要走的一步。

* a、运行 WinDbg 软件，然后按【Ctrl+S】弹出符号表设置窗。

* b、将符号表地址：SRV*C:\Symbols*

http://msdl.microsoft.com/download/symbols 粘贴在输入框中（不能换行），点击确定即可。点击确定之前，请先确认红色字的文件夹是否已被新建。

注：红色字`C:\Symbols`表示符号表本地存储路径，建议固定路径，可避免符号表重复下载。

### 3、学会打开第一个 Dump 文件！

使用【Ctrl+D】快捷键，或者点击 WinDbg 界面上的【File=>Open Crash Dump...】按钮，来打开一个 Dump 文件。

当你想打开第二个 Dump 文件时，可能因为上一个分析记录未清除，导致无法直接分析 Dump 文件，此时你可以使用快捷键【Shift+F5】来关闭上一个对 Dump 文件的分析记录。

SOS does not support the current target architecture
这个错误的原因是用了32位的任务管理器抓的32位的dump文件。
需要用64位的任务管理器抓32位的dump文件（C:\Windows\SysWOW64\taskmgr.exe）

### 4、通过简单的几个命令学会分析 Dump 文件

```WinDbg
.loadby sos clr     //首先加载sos

// 或

.load  C:\Windows\Microsoft.NET\Framework64\v4.0.30319\SOS.dll

.chain  // 显示加载的扩展dll

!runaway // 查看线程运行时间

~22s // 进入线程 22

!clrstack  // 查看当前线程堆栈变量值的信息

!dso  // 把当前栈上所有的变量都显示出来
```

#### 调试dump步骤

将dump文件拖入windbg
执行.loadby sos clr或.loadby sos mscorwks加载模块
执行!analyze -v 进行异常分析

#### 调试exe文件步骤

Open Executeable..
执行 sxe ld:clrjit
执行 g
执行.loadby sos clr

### 5、常用命令

```WinDbg
.sympath  // 检查sympath是否正确

.sympath srv*c:\Symbols*http://msdl.microsoft.com/download/symbols

也可以在命令行执行
.symfix d:\symbols

!lmi truecrypt  // 查找相应的模块信息

!sym noisy    // 检查符号表加载详细情况

lm   // 列出模块
lmf  // 指令列出当前进程中加载的所有DLL文件和对应的路径
lmvm // 查看DLL/EXE文件信息，参数为某个dll文件名称
.loadby sos mscorwks // 指令用于加载.Net 3.5版本及以下模块
.loadby sos clr // 指令用于加载.Net 4.0版本及以上模块

!help sos // 指令帮助
!threads // 显示所有线程
!address  // 内存使用情况
!threadpool(!tp) // 显示程序池信息
!ProcInfo  // 显示进程信息
!address -summary // 内存概况

!heap  // 查看进程堆

!load wow64exts

!sw

.reload  // 重新加载

.loadby sos mscorwks

.symfix+ c:\symbols  //强制下载symbols

.reload /f;  // Force reloading symbols 强制加载符号文件

.cordll -ve -u –l   //重新加载调试DLL，这是加载DLL，不是符号
!dumpheap  // 显示托管堆的信息
!dumpheap -stat  //检查当前所有托管类型的统计信息
!dumpheap -type Person –stat  // 在堆中查找指定类型（person）对象，注意大小写敏感
!dumpheap -mt 00007ffdb9386948 -min 200 //查看200byte以上的string
!dumpobj(!do)  // 显示一个对象的内容
!DumpStackObjects(!dso) // 当前线程对象分配过程
!do 0000021bcbaf5158 // 使用!do命令查看一个对象的内容
!dumparray(!da) // 显示数组
!syncblk  // 显示同步块
!runaway  // 显示线程cpu时间
!gcroot // 跟踪对象内存引用
!gcroot 0000021bcbaf5158 //使用!gcroot 查看一个对象的gc根

!DumpObj /d 0000021975972b48  //查看第对象

.cordll -ve -u -l

.chain  // 显示加载的扩展dll

.unload C:\Windows\Microsoft.NET\Framework64\v4.0.30319\sos  // 卸载

!threads -live  //查看托管线程

!runaway  // 查看线程运行时间

.loadby sos clr

!eeversion

!pe // 打印异常
!ObjSize  // 查看对象大小　ObjSize 用于知道对象地址时，查看该对象的大小。
~#s

!clrstack //显示调用栈,只显示托管代码

kb // 显示当前线程的callstack,只显示非托管代码

~*e !clrstack // 所有线程的调用堆栈
.cls // 清屏

!EEStack -EE

```

参考：

[Windbg常用命令](https://www.jianshu.com/p/57f0ab8702b5)

[https://www.cnblogs.com/sheng-jie/p/9503650.html](https://www.cnblogs.com/sheng-jie/p/9503650.html)

[http://mp.weixin.qq.com/s/R6TrIlxqJVgApFP-V2r0GA](http://mp.weixin.qq.com/s/R6TrIlxqJVgApFP-V2r0GA)

[http://blog.csdn.net/beanjoy/article/details/39203259](http://blog.csdn.net/beanjoy/article/details/39203259)

[http://www.cnblogs.com/softfair/p/The_version_SOS_not_match_version_of_CLR_PDB_symbol_for_clr_dll_not_loaded.html](http://www.cnblogs.com/softfair/p/The_version_SOS_not_match_version_of_CLR_PDB_symbol_for_clr_dll_not_loaded.html)

[http://www.cnblogs.com/Clingingboy/archive/2013/03/26/2983166.html](http://www.cnblogs.com/Clingingboy/archive/2013/03/26/2983166.html)

[http://blog.csdn.net/bcbobo21cn/article/details/51683137](http://blog.csdn.net/bcbobo21cn/article/details/51683137)