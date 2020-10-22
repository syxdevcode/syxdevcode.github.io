---
title: 在VirtualBox下给CentOS7挂载镜像
date: 2020-10-22 15:08:21
tags:
- Linux
- CentOS7
- 安装部署
- VirtualBox
categories:
- VirtualBox
---

## 在VirtualBox挂载镜像

![微信截图_20201022150956.png](/img/微信截图_20201022150956.png)

## 在CentOS7挂载镜像

```sh
# 创建文件
mkdir /media/cdrom

# 挂载
mount -o loop /dev/cdrom /media/cdrom

##查看挂载状态
df -hT

# 重新挂载系统分区
mount -a

# 卸载
umount /media/cdrom
```