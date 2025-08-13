---
title: Linux OOM机制简介
date: 2020-08-05 09:40:25
tags:
- Linux
- CentOS7
categories:
- Linux
---

## OOM简介

OOM，全称 Out-Of-Memory，是一种内存分配策略。

所谓overcommit就是操作系统分配给进程的总内存大小超过了实际可用的内存，对进程而言实为虚拟内存，一个进程占用的虚拟内存空间通常比物理空间要大，甚至可能大许多，这样做的原因是进程实际上使用的内存往往比申请的内存要少。比如有个进程申请了1G的内存，但实际上它只在一小段时间里加载了大量数据，需要使用较大的内存，而在运行过程的其他大部分时间里只用了100M的内存。这样其实有900多M的内存在大部分时间里是闲置的，完全可以分给其他进程，overcommit的机制就能充分利用这些闲置的内存。

内核参数 overcommit_memory，

可选值：0、1、2。
0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
2， 表示内核允许分配超过所有物理内存和交换空间总和的内存

有三种方式修改内核参数，但要有root权限：

（1）编辑 `/etc/sysctl.conf` ，改 `vm.overcommit_memory=1` ，然后 `sysctl -p` 使配置文件生效
（2）`sysctl vm.overcommit_memory=1`
（3）`echo 1 > /proc/sys/vm/overcommit_memory`

系统是否行使OOM，由 `/proc/sys/vm/panic_on_oom` 的值决定，当 `/proc/sys/vm/panic_on_oom` 取值为1时表示关闭OOM，取值0时表示启用OOM。

Linux对大部分申请内存的请求都回复"yes"，以便能跑更多更大的程序。因为申请内存后，并不会马上使用内存。
Linux内核为了提高内存的使用效率采用过度分配内存(over-commit memory)的办法，造成物理内存过度紧张进而触发OOM killer机制来杀死一些进程(用户态进程，不是内核线程)回收内存。

该机制会监控那些占用内存过大，尤其是瞬间很快消耗大量内存的进程，为了防止内存耗尽会把该进程杀掉。

Linux在内存分配路径上会对内存余量做检查:
（1）如果检查到内存不足，则触发OOM机制。
（2）OOM首先会对系统所有进程(出init和内核线程等特殊进程)进行打分，并选出最bad的进程；然后杀死该进程。
（3）同时会触发内核oom_reaper进行内存收割。
（4）同时内核还提供了sysfs接口系统OOM行为，以及进程OOM行为。

Linux下每个进程都有自己的OOM权重，在 `/proc/<pid>/oom_adj` 里面，范围是-17到+15，取值越高，越容易被杀掉。

如果将 `/proc/sys/vm/oom_kill_allocating_task` 的值设置为1，则OOM时直接KILL当前正在申请内存的进程，否则OOM根据进程的oom_adj和oom_score来决定。

`/proc/xxx/oom_adj`：表示进程被 OOM KILLER 杀死的权重，可读写，范围是-17~15。值越大被KILL的概率越高，当进程的oom_adj值为-17时，表示永远不会被OOM KILLER选中。

`/proc/xxx/oom_score_adj`，可读写，范围是-1000~1000。

什么是Overcommit？当已申请的内存（Committed_AS）大小超出CommitLimit值时即为Overcommit，执行命令 `cat /proc/meminfo |grep CommitLimit` 可查看CommitLimit的当前大小。

CommitLimit为系统当前可以申请的内存总大小，即内存分配上限，当 `overcommit_memory` 值为2时，其值等于：`SwapTotal + MemTotal * overcommit_ratio`。

而Committed_AS，表示所有进程已申请的内存总和。命令 `sar -r` 输出中的 `kbcommit` 对应 `/proc/meminfo` 中的 `Committed_AS` ，而 `%commit` 值等于 `Committed_AS/(MemTotal+SwapTotal)` 。

如果是大内存机器，可以考虑适当调大 `/proc/sys/vm/min_free_kbytes` 的值，但不能太大了，不然容易频繁触发内存回收，`min_free_kbytes` 是内核保留空闲内存最小值，作用是保障必要时有足够内存使用。
 
参考：

[理解LINUX的MEMORY OVERCOMMIT](http://linuxperf.com/?p=102)

[Linux内存管理 (21)OOM](https://www.cnblogs.com/arnoldlu/p/8567559.html)

[linux的vm.overcommit_memory的内存分配参数详解](http://blog.chinaunix.net/uid-30401178-id-5159439.html)

[Linux OOM一二三](https://www.cnblogs.com/aquester/p/11460372.html)

[如何理解Linux中的OOM(Out Of Memory Killer)机制？](https://www.zhihu.com/question/21972130)

[overcommit-accounting](https://www.kernel.org/doc/Documentation/vm/overcommit-accounting)