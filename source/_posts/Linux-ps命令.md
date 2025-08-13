---
title: Linux ps命令
date: 2018-05-10 11:16:26
tags:
- Linux
- Linux基础命令
categories:
- Linux基础命令
---
# Linux ps命令

Linux中的ps命令是Process Status的缩写。ps命令用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，就是执行ps命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用top命令。

要对进程进行监测和控制，首先必须要了解当前进程的情况，也就是需要查看当前进程，而 ps 命令就是最基本同时也是非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等。总之大部分信息都是可以通过执行该命令得到的。

ps 为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用 top 工具。

kill 命令用于杀死进程。

## linux上进程有5种状态

1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)

## ps工具标识进程的5种状态码

D 不可中断 uninterruptible sleep (usually IO)
R 运行 runnable (on run queue)
S 中断 sleeping
T 停止 traced or stopped
Z 僵死 a defunct (”zombie”) process

## ps命令

** 命令格式：**
ps [参数]

** 命令功能：**
用来显示当前进程的状态

** 命令参数：**

a  显示所有进程
-a 显示同一终端下的所有程序
-A 显示所有进程
c  显示进程的真实名称
-N 反向选择
-e 等于“-A”
e  显示环境变量
f  显示程序间的关系
-H 显示树状结构
r  显示当前终端的进程
T  显示当前终端的所有程序
u  指定用户的所有进程
-au 显示较详细的资讯
-aux 显示所有包含其他使用者的行程 
-C<命令> 列出指定命令的状况
--lines<行数> 每页显示的行数
--width<字符数> 每页显示的字符数
--help 显示帮助信息
--version 显示版本显示

** 使用实例 **

`ps -A`:显示所有进程信息

`ps -u root`:显示指定用户信息

`ps -ef|grep ssh`:ps 与grep 常用组合用法，查找特定进程

`ps -l`:将目前属于您自己这次登入的 PID 与相关信息列示出来

``` bash
[root@localhost Desktop]# ps -l
F S   UID    PID   PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0   2774   2722  0  80   0 - 47498 do_wai pts/0    00:00:00 su
4 S     0   2780   2774  0  80   0 - 29075 do_wai pts/0    00:00:00 bash
0 R     0  23118   2780  0  80   0 - 37235 -      pts/0    00:00:00 ps
```

** 说明：**

``` bash
F 代表这个程序的旗标 (flag)， 4 代表使用者为 super user
S 代表这个程序的状态 (STAT)，关于各 STAT 的意义将在内文介绍
UID 程序被该 UID 所拥有
PID 就是这个程序的 ID ！
PPID 则是其上级父程序的ID
C CPU 使用的资源百分比
PRI 这个是 Priority (优先执行序) 的缩写，详细后面介绍
NI 这个是 Nice 值，在下一小节我们会持续介绍
ADDR 这个是 kernel function，指出该程序在内存的那个部分。如果是个 running的程序，一般就是 "-"
SZ 使用掉的内存大小
WCHAN 目前这个程序是否正在运作当中，若为 - 表示正在运作
TTY 登入者的终端机位置
TIME 使用掉的 CPU 时间。
CMD 所下达的指令为何
在预设的情况下， ps 仅会列出与目前所在的 bash shell 有关的 PID 而已，所以， 当我使用 ps -l 的时候，只有三个 PID。
```

`ps -axjf`:列出类似程序树的程序显示

`ps aux | egrep '(cron|syslog)'`:找出与 cron 与 syslog 这两个服务有关的 PID 号码

`ps -aux |more`:用 | 管道和 more 连接起来分页查看

`ps -aux > ps001.txt`:把所有进程显示出来，并输出到ps001.txt文件

`ps -o pid,ppid,pgrp,session,tpgid,comm`:输出指定字段

## ps -ef和ps aux的区别

Linux下显示系统进程的命令ps，最常用的有ps -ef 和ps aux。这两个到底有什么区别呢？两者没太大差别，讨论这个问题，要追溯到Unix系统中的两种风格，System Ｖ风格和BSD 风格，ps aux最初用到Unix Style中，而ps -ef被用在System V Style中，两者输出略有不同。现在的大部分Linux系统都是可以同时使用这两种方式的。

** `ps -ef`:显示所有进程信息，连同命令行 **

ps -ef 是用标准的格式显示进程的

```bash
[root@localhost Desktop]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 09:40 ?        00:00:03 /usr/lib/systemd/systemd --swi
root          2      0  0 09:40 ?        00:00:00 [kthreadd]
root          3      2  0 09:40 ?        00:00:00 [ksoftirqd/0]
root          5      2  0 09:40 ?        00:00:00 [kworker/0:0H]
root          7      2  0 09:40 ?        00:00:00 [migration/0]
root          8      2  0 09:40 ?        00:00:00 [rcu_bh]
root          9      2  0 09:40 ?        00:00:02 [rcu_sched]
root         10      2  0 09:40 ?        00:00:00 [watchdog/0]
```

参数详解：

``` bash
UID    //用户ID、但输出的是用户名
PID    //进程的ID
PPID   //父进程ID
C      //进程占用CPU的百分比
STIME  //进程启动到现在的时间
TTY    //该进程在那个终端上运行，若与终端无关，则显示? 若为pts/0等，则表示由网络连接主机进程。
CMD    //命令的名称和参数
```

** `ps aux`:列出目前所有的正在内存当中的程序 ** 

ps aux 是用BSD的格式来显示

``` bash
root@localhost Desktop]# ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.4 193720  4236 ?        Ss   09:40   0:03 /usr/lib/syste
root          2  0.0  0.0      0     0 ?        S    09:40   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    09:40   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   09:40   0:00 [kworker/0:0H]
root          7  0.0  0.0      0     0 ?        S    09:40   0:00 [migration/0]
root          8  0.0  0.0      0     0 ?        S    09:40   0:00 [rcu_bh]
root          9  0.0  0.0      0     0 ?        R    09:40   0:02 [rcu_sched]
root         10  0.0  0.0      0     0 ?        S    09:40   0:00 [watchdog/0]
root         12  0.0  0.0      0     0 ?        S    09:40   0:00 [kdevtmpfs]
```

同ps -ef 不同的有列有

``` bash
USER      //用户名
%CPU      //进程占用的CPU百分比
%MEM      //占用内存的百分比
VSZ       //该进程使用的虚拟內存量（KB）
RSS       //该进程占用的固定內存量（KB）（驻留中页的数量）
STAT      //进程的状态
START     //该进程被触发启动时间
TIME      //该进程实际使用CPU运行的时间
```

其中STAT状态位常见的状态字符有

``` bash
D      //无法中断的休眠状态（通常 IO 的进程）；
R      //正在运行可中在队列中可过行的；
S      //处于休眠状态；
T      //停止或被追踪；
W      //进入内存交换 （从内核2.6开始无效）；
X      //死掉的进程 （基本很少见）；
Z      //僵尸进程；
<      //优先级高的进程
N      //优先级较低的进程
L      //有些页被锁进内存；
s      //进程的领导者（在它之下有子进程）；
l      //多线程，克隆线程（使用 CLONE_THREAD, 类似 NPTL pthreads）；
+      //位于后台的进程组；
```

参考：

[http://www.cnblogs.com/peida/archive/2012/12/19/2824418.html](http://www.cnblogs.com/peida/archive/2012/12/19/2824418.html)

[https://www.linuxidc.com/Linux/2016-07/133515.htm](https://www.linuxidc.com/Linux/2016-07/133515.htm)