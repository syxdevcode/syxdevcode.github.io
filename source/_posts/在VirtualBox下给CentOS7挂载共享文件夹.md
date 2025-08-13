---
title: 在VirtualBox下给CentOS7挂载共享文件夹
date: 2020-10-22 15:17:23
tags:
---

## 安装依赖

如果虚机没有联网，需要挂载系统镜像安装。

```sh
yum install gcc kernel-devel kernel-headers dkms make bzip2
```

## 设置VirtualBox

![微信截图_20201022172443.png](/img/微信截图_20201022172443.png)

## 挂载VBoxGuestAdditons镜像

在安装目录下：C:\Program Files\Oracle\VirtualBox，找到 VBoxGuestAdditons.iso 镜像，并挂载。

![微信截图_20201022172219.png](/img/微信截图_20201022172219.png)

## 虚机设置

启动虚机，执行 lsscsi 查看 IDE , /dev/sr0 就是 VBoxGuestAdditons 镜像挂载之后的路径。

![微信截图_20201022172727.png](/img/微信截图_20201022172727.png)

```sh
# 创建文件夹
mkdir /media/cdrom

# 挂载
mount /dev/sr0 /media/cdrom

cd /media/cdrom

ls

# 安装
sh ./VBoxLinuxAdditons.run
```

进入 /media 目录出现 sf_share，前面出现的 sf 代表成功。

![微信截图_20201022173219.png](/img/微信截图_20201022173219.png)

还可以通过 mount 命令把 Centos7 的目录下文件夹变成共享文件夹。

```sh
mount -t vboxsf share  /home/share
```

参考：

[VirtualBox使用Centos7与主机共享文件夹](https://www.cnblogs.com/uqing/p/8160318.html)