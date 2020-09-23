---
title: Linux-CentOS7磁盘扩充
date: 2020-09-23 17:52:59
tags:
- 计算机基础
- 硬盘
- Linux
- LVM
- 计算机科学
categories: Linux LVM
---

```sh
# 创建物理卷
pvcreate  /dev/vdb

# 扩展卷组
vgextend centos /dev/vdb

# 扩展逻辑卷大小
lvextend -L +500G /dev/mapper/centos-root

# xfs_growfs: 调整一个 xfs 文件系统大小（只能扩展)
xfs_growfs /dev/mapper/centos-root

# 查看
df -hT
```