---
title: Linux lvdisplay命令
date: 2020-08-27 13:09:55
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

`lvdisplay` 命令用于显示 `LVM` 逻辑卷空间大小、读写状态和快照信息等属性。如果省略"逻辑卷"参数，则 `lvdisplay` 命令显示所有的逻辑卷属性。否则，仅显示指定的逻辑卷属性。

LVM 是 `Logical Volume Manager`（逻辑卷管理）的简写，它是Linux环境下对磁盘分区进行管理的一种机制。

**语法**

```
lvdisplay (参数)
```

**参数**

逻辑卷：指定要显示属性的逻辑卷对应的设备文件。

**实例**

使用lvdisplay命令显示指定逻辑卷的属性。在命令行中输入下面的命令：

```sh
#显示逻辑卷属性
lvdisplay /dev/mapper/vg_ren-LogVol06_u01
  --- Logical volume ---
  LV Path                /dev/vg_ren/LogVol06_u01
  LV Name                LogVol06_u01
  VG Name                vg_ren
  LV UUID                xMhOr0-M11J-X410-BLmE-0zVw-ojqJ-yiH1yd
  LV Write Access        read/write
  LV Creation host, time ren-sj0001, 2020-01-19 11:18:05 +0800
  LV Status              available
  # open                 1
  LV Size                300.00 GiB
  Current LE             76800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           251:3
```

参考：

[lvdisplay命令](https://man.linuxde.net/lvdisplay)