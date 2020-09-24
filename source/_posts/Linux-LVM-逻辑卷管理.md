---
title: Linux LVM(逻辑卷管理)
date: 2020-08-27 17:24:24
tags:
- 计算机基础
- 硬盘
- Linux
- LVM
- 计算机科学
categories: Linux LVM
---

## 简介

LVM是逻辑盘卷管理( `Logical Volume Manager`)的简称，它是Linux环境下对磁盘分区进行管理的一种机制，LVM是建立在硬盘和 分区之上的一个逻辑层，来提高磁盘分区管理的灵活性。

通过LVM系统管理员可以轻松管理磁盘分区，如:将若干个磁盘分区连接为一个整块的卷组 (`volume group`)，形成一个存储池。

管理员可以在卷组上随意创建逻辑卷组(`logical volumes`)，并进一步在逻辑卷组上创建文件系 统。管理员通过LVM可以方便的调整存储卷组的大小，并且可以对磁盘存储按照组的方式进行命名、管理和分配。

例如按照使用用途进行定义:`development` 和 `sales`，而不是使用物理磁盘名 `sda` 和`sdb`。而且当系统添加了新的磁盘，通过LVM管理员就不必将磁盘的 文件移动到新的磁盘上以充分利用新的存储空间，而是直接扩展文件系统跨越磁盘即可。

![58846-20160825131201304-610664422.jpg](/img/58846-20160825131201304-610664422.jpg)

## LVM相关名词

* **PV(physical volume)**：`物理卷` 在逻辑卷管理系统最底层，可为整个物理硬盘或实际物理硬盘上的分区。
* **VG(volume group)**：`卷组` 建立在物理卷上，一卷组中至少要包括一物理卷，卷组建立后可动态的添加卷到卷组中，一个逻辑卷管理系统工程中可有多个卷组。
* **LV(logical volume)**：`逻辑卷` 建立在卷组基础上，卷组中未分配空间可用于建立新的逻辑卷，逻辑卷建立后可以动态扩展和缩小空间。
* **PE(physical extent)**：`物理区域` 是物理卷中可用于分配的最小存储单元，物理区域大小在建立卷组时指定，一旦确定不能更改，同一卷组所有物理卷的物理区域大小需一致，新的pv加入到vg后，pe的大小自动更改为vg中定义的pe大小。
* **LE(logical extent)**：`逻辑区域` 是逻辑卷中可用于分配的最小存储单元，逻辑区域的大小取决于逻辑卷所在卷组中的物理区域的大小。
卷组描述区域：卷组描述区域存在于每个物理卷中，用于描述物理卷本身、物理卷所属卷组、卷组中逻辑卷、逻辑卷中物理区域的分配等所有信息，它是在使用pvcreate建立物理卷时建立的。

**关于PV命令**

`pvcreate` 命令用于将物理硬盘分区初始化为物理卷，以便LVM使用。

```sh
pvcreate：将物理分区制作成 物理卷 PV
pvscan：搜索当前系统里所有的具有 PV 属性的磁盘
pvdisplay：显示当前系统上的 PV 状态
pvremove：将 PV 属性删除，让其不具有 PV 属性，成为一般的分区
```

**关于VG下命令**

```sh
vgcreate：建立 VG 。它的参数比较多，一会儿详细介绍
vgscan：搜索当前系统里有多少个卷组 VG 存在
vgdisplay：显示当前系统里指定的 VG 或所有 VG 的信息
vgextend：在卷组 VG 中增加额外的物理卷 PV
vgreduce：在卷组 VG 中删除 物理卷 PV
vgchange：设置 VG 是否启动（active）
vgremove：删除一个卷组 VG 本身
```

## 创建LVM过程

创建 LVM 过程 ：

分区成LVM格式(8e)---PV创建--VG创建---LV创建---格式化分区---MOUNT分区----e2fsadm调整LV大小

在LVM发行包中有一个称为 `e2fsadm` 的工具，它同时包含了 `lvextend` 与 `resize2fs` 的功能。

### 创建分区

使用分区工具(如: `fdisk` 等)创建LVM分区，方法和创建其他一般分区的方式是一样的，区别仅仅是LVM的分区类型为8e。

```sh
fdisk /dev/vdb

# 选项
# p n w

# 重启（可选）
reboot
```

### 创建物理卷

创建物理卷的命令为 `pvcreate`，利用该命令将希望添加到卷组的所有分区或者磁盘创建为物理卷:

```sh
# 格式化
mkfs.ext4 /dev/vdb

# 将整个磁盘创建为物理卷
pvcreate /dev/vdb

# 或 将单个分区创建为物理卷
pvcreate /dev/vda1

# 或  将6-9分区转成pv
pvcreate /dev/vda{6,7,8,9}

# -v显示创建的全部过程
pvcreate [-v] /dev/vda2 /dev/vdb2

# 显示系统上的 PV 状态
pvdisplay
```

### 创建卷组

创建卷组的命令为 `vgcreate`，将使用 `pvcreate` 建立的物理卷创建为一个完整的卷组:

```sh
# unicomvg：指定该卷组的逻辑名:
# 后面参数指定希望添加到该卷组的所有分区和磁盘
vgcreate unicomvg /dev/vdb
```

`vgcreate` 在创建卷组 `unicomvg` 以外，还设置使用大小为4MB的PE(默认为4MB)，这表示卷组上创建的所有逻辑卷都以4MB为增量单位来进行扩充 或缩减。

由于内核原因，PE大小决定了逻辑卷的最大大小，4MB的PE决定了单个逻辑卷最大容量为256GB，若希望使用大于256G的逻辑卷则创建卷组时指定更大的PE。

PE大小范围为8KB到512MB，并且必须总是2的倍数(使用-s指定，具体请参考 `manvgcreate` )。(centos 6.2系统已发现没有这种限制)

例如，如果希望使用 64MB 的PE创建卷组，这样逻辑卷最大容量就可以为 `4TB`，命令如下: 

```sh
vgcreate -64MB lvmdisk /dev/vdb1 /dev/vdc1
```

### 激活卷组

为了立即使用卷组而不是重新启动系统，可以使用 `vgchange` 来激活卷组:

```sh
vgchange -ay unicomvg
```

### 添加新的物理卷到卷组中

当系统安装了新的磁盘并创建了新的物理卷，而要将其添加到已有卷组时，就需要使用 `vgextend` 命令:

```sh
vgextend unicomvg /dev/vdb1
```

这里`/dev/vdb1`是新的物理卷。

### 从卷组中删除一个物理卷

要从一个卷组中删除一个物理卷，首先要确认要删除的物理卷没有被任何逻辑卷正在使用，就要使用 `pvdisplay` 命令察看一个该物理卷信息:

如果某个物理卷正在被逻辑卷所使用，就需要将该物理卷的数据备份到其他地方，然后再删除。

删除物理卷的命令为 `vgreduce`:

```sh
vgreduce unicomvg /dev/sda1
```

### 创建逻辑卷

逻辑卷（Logical Volumes）简称 LV，是在卷组中划分的一个逻辑区域，类似于非LVM系统中的硬盘分区。 
创建逻辑卷的命令为lvcreate，通过下面的命令。

该命令就在卷组 `unicomvg` 上创建名字为 `unicomvol` ，大小为15000M（15G）的逻辑卷，并且设备入口为 `/dev/unicomvg/unicomvol` ( `unicomvg` 为卷组名，`unicomvol` 为逻辑卷名)。

创建逻辑卷的命令为 `lvcreate`:

* -L：指定逻辑卷的大小，单位为“kKmMgGtT”字节；
* -l：指定逻辑卷的大小（LE数）。

```sh
lvcreate -L 15000 -n unicomvol unicomvg

# 或 指定100%的LE 到逻辑卷
lvcreate -l 100%VG  -n unicomvol unicomvg

# 也可以指定大小为 15G
lvcreate -L 15G -n unicomvol unicomvg
```

如果希望创建一个使用全部卷组的逻辑卷，则需要首先察看该卷组的PE数，然后在创建逻辑卷时指定:

```sh
vgdisplay unicomvg | grep "TotalPE"

# 显示
Total PE 45230

lvcreate -l 45230 unicomvg -n unicomvol
```

同卷组一样，逻辑卷在创建的过程中也被分成了一块一块的空间，这些空间称为 `LE（Logical Extents）`，在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

### 创建文件系统

创建了文件系统以后，就可以加载并使用它:

```sh
# 创建目录
mkdir /data/wwwroot

# 挂载
mount /dev/unicomvg/unicomvol /data/wwwroot

# 如果希望系统启动时自动加载文件系统，
# 则还需要在 /etc/fstab 中添加内容:
/dev/unicomvg/unicomvol /data/wwwroot ext4  defaults 0 0

# 或者使用
echo  "/dev/mapper/unicomvg-unicomvol /unicom ext4 defaults 0 0 " >>/etc/fstab
```

`/etc/fstab`文件的每一行都遵循以下格式：

```sh
<device>   <dir>    <type>   <options>       <dump> <pass>
```

* **device**：指定加载的磁盘分区或移动文件系统，除了指定设备文件外，也可以使用 `UUID、LABEL`来指定分区；
* **dir**：指定挂载点的路径；
* **type**：指定文件系统的类型，如 `ext3`，`ext4`等；
* **options**：指定挂载的选项，默认为defaults，其他可用选项包括`acl`，`noauto`，`ro`等；
* **dump**：表示该挂载后的文件系统能否被dump备份命令作用；0表示不能，1表示每天都进行dump备份，2表示不定期进行dump操作。
* **pass**：表示开机过程中是否校验扇区；0表示不要校验，1表示优先校验（一般为根目录），2表示为在1级别校验完后再进行校验；

## 删除一个逻辑卷

删除逻辑卷以前首先需要将其卸载，然后删除:

```sh
# 卸载
umount /dev/unicomvg/unicomvol

# 删除
lvremove /dev/unicomvg/unicomvol
```

## 扩展逻辑卷大小

LVM提供了方便调整逻辑卷大小的能力，扩展逻辑卷大小的命令是 `lvextend` :

```sh
# 将逻辑卷unicomvol的大小扩招为12G
lvextend -L12G /dev/unicomvg/unicomvol

# 将逻辑卷unicomvol的大小增加1G
lvextend -L +1G /dev/unicomvg/unicomvol、

# 将卷组 100% 分配到 逻辑卷unicomvol
lvextend -l 100%VG /dev/mapper/centos-root
```

**调整文件系统大小**

resize2fs 命令是用来增大或者收缩未加载的 `ext2/ext3/ext4` 文件系统的大小。

参数：

* -d  打开调试特性
* -p  打印已完成的百分比进度条
* -f  强制执行调整大小操作，覆盖掉安全检查操作
* -F  开始执行调整大小前，刷新文件系统设备的缓冲区
* -M: 将文件系统缩小到最小值
* -P: 显示文件系统的最小值

```sh
# 建议最好将文件系统卸载，调整大小，然后再加载
# 卸载
umount /dev/unicomvg/unicomvol

# 调整
resize2fs -p /dev/unicomvg/unicomvol

# 挂载
mount  /dev/unicomvg/unicomvol /data/wwwroot
```

## 减少逻辑卷大小

`lvreduce`命令用于减少LVM逻辑卷占用的空间大小。使用 `lvreduce` 命令收缩逻辑卷的空间大小有可能会删除逻辑卷上已有的数据，所以在操作前必须进行确认。

* -L：指定逻辑卷的大小，单位为“kKmMgGtT”字节；
* -l：指定逻辑卷的大小（LE数）。

需要首先将文件系统卸载:

```sh
umount /data/wwwroot

# 减小 2G
lvreduce -L -2G /dev/unicomvg/unicomvol

mount /dev/unicomvg/unicomvol /data/wwwroot
````

参考：

[LVM](https://baike.so.com/doc/5462643-5700974.html)

[Linux-centos磁盘扩容](https://www.cnblogs.com/wangyongqiang/articles/12713877.html)

[LVM原理及PV、VG、LV、PE、LE关系图](https://www.cnblogs.com/stragon/p/5806388.html)
