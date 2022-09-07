---
title: 扩大ubuntu的ubuntu--vg-ubuntu--lv空间
date: 2022-09-07 16:33:36
tags:
  - Ubuntu
  - Linux
  - 计算机基础
  - 硬盘
  - LVM
  - 计算机科学
categories:
  - Linux LVM
---

## 检查使用量

检查使用量，可以看到所属卷组。

```shell
# df /var
Filesystem                        1K-blocks     Used Available Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv 102626232 13556520  83810448  14% /
```

<!--more-->

## 命令

```shell
lvextend -L 10G /dev/mapper/ubuntu--vg-ubuntu--lv      //增大或减小至19G
lvextend -L +10G /dev/mapper/ubuntu--vg-ubuntu--lv     //增加10G
lvreduce -L -10G /dev/mapper/ubuntu--vg-ubuntu--lv     //减小10G
lvresize -l  +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv   //按百分比扩

# 显示当前系统里指定的 VG 或所有 VG 的信息
vgdisplay

# 查看磁盘空间占用情况
df -h
```

参考:

[如何扩大 ubuntu 的 ubuntu--vg-ubuntu--lv 空间](https://zhuanlan.zhihu.com/p/359959580)
