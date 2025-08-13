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
# 或 将卷组 100% 分配到 逻辑卷unicomvol
lvextend -l 100%VG /dev/mapper/centos-root

# 确认文件系统是xfs
cat /etc/fstab | grep centos-root

# xfs_growfs: 调整一个 xfs 文件系统大小（只能扩展)
xfs_growfs /dev/mapper/centos-root

# 可选，resize2fs
# resize2fs -p /dev/mapper/centos-root

# 查看
df -hT
```

xfs 相关：

```sh
xfs_admin: 调整 xfs 文件系统的各种参数
xfs_copy: 拷贝 xfs 文件系统的内容到一个或多个目标系统（并行方式） 
xfs_db: 调试或检测 xfs 文件系统（查看文件系统碎片等） 
xfs_check: 检测 xfs 文件系统的完整性 
xfs_bmap: 查看一个文件的块映射 
xfs_repair: 尝试修复受损的 xfs 文件系统 
xfs_fsr: 碎片整理 
xfs_quota: 管理 xfs 文件系统的磁盘配额 
xfs_metadump: 将 xfs 文件系统的元数据 (metadata) 拷贝到一个文件中 
xfs_mdrestore: 从一个文件中将元数据 (metadata) 恢复到 xfs 文件系统 
xfs_growfs: 调整一个 xfs 文件系统大小（只能扩展） 
xfs_freeze    暂停（-f）和恢复（-u）xfs 文件系统
xfs_logprint: 打印xfs文件系统的日志 
xfs_mkfile: 创建xfs文件系统 
xfs_info: 查询文件系统详细信息 
xfs_ncheck: generate pathnames from i-numbers for XFS 
xfs_rtcp: XFS实时拷贝命令 
xfs_io: 调试xfs I/O路径
```

resize2fs 相关

此命令的适用范围：`RedHat、RHEL、Ubuntu、CentOS、SUSE、openSUSE、Fedora`。

调整 `ext2\ext3\ext4` 文件系统的大小，它可以放大或者缩小没有挂载的文件系统的大小。如果文件系统已经挂载，它可以扩大文件系统的大小，前提是内核支持在线调整大小。

`size` 参数指定所请求的文件系统的新大小。如果没有指定任何单元，那么 `size` 参数的单位应该是文件系统的文件系统块大小。`size` 参数可以由下列单位编号之一后缀：`s`、`K`、`M` 或 `G`，分别用于512字节扇区、千字节、兆字节或千兆字节。文件系统的大小可能永远不会大于分区的大小。如果未指定 `Size` 参数，则它将默认为分区的大小。

`resize2fs` 程序不操作分区的大小。如果希望扩大文件系统，必须首先确保可以扩展基础分区的大小。如果您使用逻辑卷管理器 `LVM(8)` ，可以使用 `fdisk(8)` 删除分区并以更大的大小重新创建它，或者使用 `lvexport(8)` 。在重新创建分区时，请确保使用与以前相同的启动磁盘圆柱来创建分区！否则，调整大小操作肯定无法工作，您可能会丢失整个文件系统。运行 `fdisk(8)` 后，运行 `resize2fs` 来调整 `ext 2`文件系统的大小，以使用新扩大的分区中的所有空间。

如果希望缩小 `ext2` 分区，请首先使用 `resize2fs` 缩小文件系统的大小。然后可以使用 `fdisk(8)` 缩小分区的大小。缩小分区大小时，请确保不使其小于 `ext2` 文件系统的新大小。

```sh
# 语法相关
resize2fs  [选项]  device  [size]
resize2fs  [ -fFpPM ]  [ -d debug-flags ]  [ -S RAID-stride ]  device  [ size ]

# 选项列表
-d debug-flags
    打开各种resize2fs调试特性，如果它们已经编译成二进制文件的话。调试标志应该通过从以下列表中添加所需功能的数量来计算：
    2，调试块重定位。
    4，调试iNode重定位。
    8，调试移动inode表。
-f  强制执行，覆盖一些通常强制执行的安全检查。
-F  执行之前，刷新文件系统的缓冲区
-M  将文件系统缩小到最小值
-p  显示已经完成任务的百分比
-P  显示文件系统的最小值
-S RAID-stride
```

`resize2fs` 程序将启发式地确定在创建文件系统时指定的 `RAID` 步长。此选项允许用户显式地指定 `RAID` 步长设置，以便由 `resize2fs` 代替。