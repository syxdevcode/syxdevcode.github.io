---
title: Esxi8常用命令总结
date: 2023-04-26 17:39:14
tags:
  - Linux
  - Esxi8
  - 安装部署
categories: 
  - Esxi8
---

Alt + F1

## 命令

```sh
# 查看
esxcli storage

# 列出ESXi主机的所有LUN（逻辑单元）的路径信息
esxcli storage core path list

# 列出连接到ESXi的LUN列表
esxcli storage core device list

# 若要查询LUN的详细路径信息，可使用如下命令列出某个卷的信息。其中-d参数加上该lun的wwn号。
esxcli storage core path list -d naa.600b342a17eacb1d6e6bde340d0000d0

# 查询active总路径或单独某个lun的路径数量
esxcli storage core path list | grep “State: active” | wc -l

# 查询单独某个LUN的路径的数量，以LUN：2为例
esxcli storage core path list | grep “LUN: 2” | wc -l

#  查看系统版本
vmware -v

# 查看系统版本包括patch等信息
esxcli system version get

# 查看系统时间
esxcli system time get

#  ESXi主机进入/退出，维护模式
esxcli system maintenanceMode set --enable true/false

# 系统重启/关机（必须处于维护模式，否则命令不生效）
esxcli system shutdown reboot/poweroff

# 查看接口ipv4地址
esxcli network ip interface ipv4 get

# 查看路由表
esxcli network ip route ipv4 list

# 查看ESXi主机网卡列表（nic）或up-link列表
esxcli network nic list

# 关闭/打开vmnic1接口
esxcli network nic down/up -n=vmnic1

```
<!--more-->

## ESXi的shell命令

1、`services.sh` 是 Linux服务通常使用services命令管理，管理ESXi服务是通过使用 `services.sh` 命令实现的。`services.sh`命令支持的参数包括`stop、start、restart`，通过这三个参数可以停止、启动或重启所有的ESXi服务。

2、`services.sh restart` ： 重启所有的ESXi服务

3、`/etc/init.d` ： 执行位于 `/etc/init.d` 目录下的脚本可以启动或停止对应的服务。如果只想重启vCenter Server Agent（vpax服务），可以运行 `/etc/init.d/vpxa restart` 命令。而 `services.sh restart` 将重启所有服务。

4、`/etc/init.d/vpxa restart` ：重启主机上的 vCenter Agent

5、`cat /etc/chkconfig.db` ： 查看所有ESXi服务的运行状态

6、`vmkping` ：是我们都熟悉ping命令的用法及功能。Vmkping命令更进一步，允许使用Vmkernel的IP堆栈通过特定的接口发送ICMP数据包。这意味着你可以通过vMotion网络而非管理网络发送ping包。

```sh
# 通过vmkl接口向10.10.10.1发送ICMP请求 【-I：大写的i】
vmkping –I vmk1 10.10.10.1
```

7、`nc` ：组合使用vmkping、nc命令（netcat），可以确认ESXi主机与特定IP之间的网络连通性。尽管vmkping命令通过ICMP确认连通性，但有时我们想确认是否可以访问特定的TCP端口(例如iSCSI的TCP端口是3260)。

```sh
nc -z 10.10.10.10 3260
nc -z 10.128.0.160 22

# UDP端口检查
nc -uz 10.128.0.160 22 # UDP端口22不通
nc -uz 10.128.16.9 123 # UDP端口123通
Connection to 10.128.16.9 123 port [udp/ntp] succeeded!
```

8、`vmkfstools`：如果需要通过命令行管理VMFS数据卷以及虚拟磁盘，那么vmkfstools命令就派上用场了。使用vmkfstools命令可以创建、克隆、扩展、重命名并删除VMDK文件。除了虚拟磁盘选项，你还可以使用vmkfstools命令创建、扩展、增大、回收文件系统的数据块。

```sh
vmkfstools –i test.vmdk testclone.vmdk # 将test.vmdk克隆为testclone.vmdk
```

9、`esxtop` ：对ESXi主机进行性能监控以及故障诊断时，很少有工具能够提供和esxtop同样多的信息。除提供和Linux top命令类似的功能外，esxtop还可以收集很多VMware专有的指标，包括中断、内存、网络、磁盘适配器、磁盘设备以及电源管理。

10、`vscsiStats` ：需要进一步监控存储I/O的性能时，vscsiStats命令就能够派上用场了。vscsiStats命令能够帮助你收集与虚拟机磁盘I/O负载相关的性能数据。进行容量规划或者迁移后端存储时，使用vscsiStats命令收集到的数据可谓价值连城。

11、`vim-cmd` ：vim-cmd是构建在hostd进程之上的命令空间，允许最终用户调用几乎所有的vSphere API。Vim-cmd提供了一些ESXi子命令管理不同的虚拟基础设施，而且和vimsh相比，更容易使用。

12、`dcui` ：登录到ESXi主机时，VMware直接用户控制台接口（DCUI）提供了基于菜单的主机管理功能。DCUI提供了很多不同的功能，比如root密码维护、网络维护。有时你可能只能通过SSH访问主机，但幸运的是，在命令行下执行dcui命令就可以进入基于菜单的DCUI系统。【图形系统】

13、`vm-support` ：曾经想过收集ESXi主机所有的支持及日志信息吗？Vm-support命令恰好能够满足你的要求。如果之前与VMware的技术支持热线联系过，那么很可能用过这个命令。

## Linux shell命令

```sh
find/cat/grep –在试图查找指定的文件或者在某个文件中查找字符串时这三个命令非常重要。find命令可以基于文件名或者模式查找指定的文件，cat命令用于显示文件内容，grep用于在单个或多个文件内查找指定的字符串。

find / -name "*iso"

find /path/to/vm/folder –i name "delta" – 列出虚拟机所有的增量磁盘。

cat hostd.log | grep error –在hostd.log中查找所有的"error"字符串

head/tail –查看文件内容时这两个命令非常有用。尽管可以使用cat命令显示文件完整的内容，但head以及tail命令可以用于显示文件开头或结尾的部分，忽略了文件的中间内容。进行故障诊断时tail命令尤为有用，尤其是可以使用-f标记实时监控日志文件发生的变化。

tail -f /var/log/vmkernel.log – 实时查看vmkernel日志发生的变化

less –显示大文件内容时less命令非常有用。通过在cat命令的输出内容之后输入“|”less,可以分页显示输出结果，而且可以向前或向后滚动浏览。

cat /var/log/vpxa.log | less –在屏幕上分页显示vpxa.log文件的内容。

df/vdf –这两个命令显示文件系统的可用空间。df命令显示本地文件系统以及数据存储的容量、已用空间以及可用空间。为查看ESXi主机不同随机磁盘的使用情况，必须使用vdf命令。这两个命令都可以用于发现由于可用空间不足而可能导致的任何问题。

ps/kill –这两个命令分别用于查找ESXi主机内部运行的服务、强制终止这些服务。ps命令包括很多命令行开关，但最常用的是检索正在运行的进程的ID，然后就可以使用Kill命令终止相应的服务。

vi – 如果之前不熟悉vi命令，那么在学习时大多会遇到麻烦。Vi是一个文本编辑器，用于修改文件内容—vSphere管理员通过命令行shell进行故障诊断时必须要具备该技能。
```

## esxcli命令探究

esxcli命令用途广泛，我们不能简单地将其归为单个命令。esxcli包括许多不同的命名空间，允许你控制ESXi提供的几乎所有设备。下面列出了使用最频繁（肯定不是所有）的命名空间：

esxcli hardware – 想获取ESXi主机的硬件及配置信息时，esxcli硬件命名空间就能够派上用场了。

esxcli hardware cpu list – 获取CPU信息（系列、型号以及缓存）

esxcli hardware memory get – 获取内存信息（可用内存以及非一致内存访问）

esxcli iscsi – iscsi命名空间可以被用于监控并管理硬件iSCSI及软件iSCSI设置。

esxcli iscsi software –用于启用/禁用软件iSCSI initiator。

esxcli iscsi adapter –用于设置软硬件iSCSI适配器的发现、CHAP以及其他设置

esxcli iscsi sessions – 用于列出主机上已建立的iSCSI会话。

esxcli network –需要监控vSphere网络并对如下网络组件进行调整时，包括虚拟交换机、VMkernel网络接口、防火墙以及物理网卡等，esxcli网络命名空间就派上用场了。

esxcli network nic –列出并修改网卡信息，比如名字、唤醒网卡以及速度。

esxcli network vm list – 列出有一个活动网络端口的虚拟机的网络信息。

esxcli network vswitch –检索并管理VMware的标准交换机以及分布式虚拟交换机。

esxcli network ip – 管理VMkernel端口，包括管理、vMotion以及FT网络。还可以修改主机的所有IP栈，包括DNS、IPsec以及路由信息。

esxcli software – 软件命名空间可以用于检索ESXi主机已安装的软件及驱动并可以安装新组件。

esxcli software vib list – 列出ESXi主机上已经安装的软件及驱动。

esxcli storage – 可能是最常用的esxcli命令命名空间之一，包括了管理连接到vSphere的存储的所有信息。

esxcli storage core device list – 列出当前存储设备

esxcli storage core device vaai status get –获得存储设备支持的VAAI的当前状态。

esxcli system – 通过该命令使你能够控制ESXi的高级选项，比如设置syslog并管理主机状态。

esxcli system maintenanceMode set –enabled yes/no – 将主机设置为维护模式

查看并更改ESXi高级设置（提示：使用esxcli system settings

advanced list –d 命令查看非默认设置）

esxcli system syslog –查看 Syslog 及配置信息

esxcli vm – ESXi的虚拟机命名空间用于列出运行在主机上的虚拟机的各种信息，如果需要可以强制关闭这些虚拟机。

esxcli vm process list –列出已启动的虚拟机的进程信息。

esxcli vm process kill – 停止正在运行的虚拟机的进程，关闭虚拟机或者强制关闭虚拟机电源。

esxcli vsan – ESXi的VSAN命名空间包括配置并维护VSAN的很多命令，包括数据存储、网络、默认域名以及策略配置。

esxcli vsan storage – 配置VSAN使用的本地存储，包括增加、删除物理存储并修改自动声明。

esxcli vsan cluster – 本地主机脱离/加入VSAN集群。

esxcli esxcli – esxcli命令包括一个称为esxcli的命名空间，通过使用esxcli命名空间，你可以获得更多信息。

esxcli esxcli command list – 列出所有的esxcli命令及其提供的功能

将出问题的前端的业务注册到另一个前端 双机架构
find /vmfs/volumes/ | egrep "win2003|win2008|centos"|grep .vmx$|while read i ; do vmware-cmd "$i" register; done

## 基础信息查看

esxcli --help ：帮助命令
reboot ：重起主机
poweroff ：关闭主机
vmware -v ：显示你的esxi版本
esxcli system version get ：查看ESXI版本号和build号
esxcfg-info -a ：显示所有ESXI相关信息
esxcfg-info -w ：显示EXSI上的硬件信息
esxcfg-vmknic -l ：查看ESXI主机IP
esxcli hardware cpu list ：查看CPU信息（Brand,Core Speed）
esxcl hardware cpu global get ：查看CPU Cores信息
esxcli hardware memory get ：查看物理内存信息
esxcli hardware clockget ：查看ESXI宿主当前时间
vim-cmd hostsvc/hostsummary ：查看宿主机摘要信息
esxcli software vib install -d /vmfs/volumes/datastore/patches/xxx.zip ：为ESXi主机安装更新补丁和驱动
esxcli network vm list ：列出虚拟机的网路信息
vim-cmd vmsvc/getallvms ：列出所有虚拟机VMID
vim-cmd vmsvc/power.getstate VMID ：查看指定VMID虚拟机状态
vim-cmd vmsvc/power.shutdown VMID ：关闭虚拟机
vim-cmd vmsvc/power.on VMID ：开起虚拟机
vim-cmd vmsvc/power.off VMID ：如果虚拟机强制关机
vim-cmd vmsvc/get.config VMID ：查看虚拟机配置信息
vim-cmd hostsvc/maintenance_mode_enter ：主机进入维护模式（虚拟要挂起或关机才能执行）
vim-cmd hostsvc/maintenance_mode_exit ：主机退出维护模式

参考：

[使用 VMware ESXi/ESX 的命令行查询磁盘信息及路径等](https://blog.csdn.net/u011775882/article/details/119681912)

[ESXI常用命令](https://www.cnblogs.com/slqt/p/10001909.html)

[【VMware虚拟化】ESXi常用命令总结（文字版本）](https://blog.csdn.net/weixin_42742658/article/details/122138194)