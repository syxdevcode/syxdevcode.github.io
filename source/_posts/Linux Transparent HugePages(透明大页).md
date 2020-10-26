---
title: Linux Transparent HugePages(透明大页)
date: 2020-08-05 14:40:31
tags:
- Linux
- CentOS7
categories:
- Linux
---

什么是 Transparent Huge Pages？为提升性能，通过大内存页来替代传统的4K页，使用得管理虚拟地址数变少，加快从虚拟地址到物理地址的映射，以及摒弃内存页面的换入换出以提高内存的整体性能。内核 Kernel 将程序缓存内存中，每页内存以2M为单位。相应的系统进程为 khugepaged。
 
在Linux中，有两种方式使用 Huge Pages，一种是2.6内核引入的 HugeTLBFS，另一种是2.6.36内核引入的THP。HugeTLBFS 主要用于数据库，THP广泛应用于应用程序。

一般可以在 rc.local 或 /etc/default/grub 中对 Huge Pages 进行设置。

Linux下的大页分为两种类型：标准大页（Huge Pages）和透明大页（Transparent Huge Pages）。Huge Pages有时候也翻译成大页/标准大页/传统大页，它们都是Huge Pages的不同中文翻译名而已。目的是使用更大的内存页面（memory page size） 以适应越来越大的系统内存，让操作系统可以支持现代硬件架构的大页面容量功能。透明大页（Transparent Huge Pages）缩写为THP，这个是RHEL 6（其它分支版本SUSE Linux Enterprise Server 11, and Oracle Linux 6 with earlier releases of Oracle Linux Unbreakable Enterprise Kernel 2 (UEK2)）开始引入的一个功能。具体可以参考官方文档。这两者的区别在于大页的分配机制，标准大页管理是预分配的方式，而透明大页管理则是动态分配的方式。


## CentOS7 禁用Transparent Huge Pages

自CentOS6版本开始引入了Transparent Huge Pages(THP)，从 CentOS7 版本开始，该特性默认就会启用。尽管THP的本意是为提升内存的性能，不过某些数据库厂商还是建议直接关闭THP(比如说 ORACLE、MariaDB、MongoDB 等)，否则可能会导致性能出现下降。

首先检查THP的启用状态：

```sh
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
[always] madvise never
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
```
这个状态就说明都是启用的。

我们这个时候当然可以逐个修改上述两文件，来禁用THP，但要想一劳永逸的令其永久生效，还是参考下列的步骤。

编辑 rc.local 文件：

```s
[root@localhost ~]# vim /etc/rc.local
```

增加下列内容：

```sh
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```

保存退出，然后赋予 rc.local 文件执行权限：

```sh
[root@localhost ~]# chmod +x /etc/rc.local
```

最后重启系统，以后再检查THP应该就是被禁用了

```sh
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
always madvise [never]
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag 
always madvise [never]
```

参考：

[Linux传统Huge Pages与Transparent Huge Pages再次学习总结](https://www.cnblogs.com/kerrycode/p/7760026.html)

[Huge pages (标准大页)和 Transparent Huge pages(透明大页)](http://blog.itpub.net/26736162/viewspace-2214374/)

[CentOS7 禁用Transparent Huge Pages的实现方法](https://www.jb51.net/article/98191.htm)