---
title: win10安装linux子系统
date: 2020-01-21 16:25:13
tags:
- Linux
- Windows子系统
- WSL
- Linux基础命令
categories:
- WSL
---

## 简介

**WSL1和WSL2**
相比于 WSL1，WSL2 通过虚拟机的方式带来了更完整的 Linux 内核，但这种方式也引入了一些问题，微软给出了下面的图表来展示这些不同：

![v2-bb7b8a23b362ba5329d66517f81fbca8_720w.jpg](/img/v2-bb7b8a23b362ba5329d66517f81fbca8_720w.jpg)

## 安装

```ps1
# 完全未安装 WSL 时才有效
wsl --install

# 以查看可用发行版列表并运行
wsl --list --online
# 或
wsl -l -o

# 安装发行版
wsl --install -d <DistroName>
```

`--install` 命令执行以下操作：

* 启用可选的 WSL 和虚拟机平台组件
* 下载并安装最新 Linux 内核
* 将 WSL 2 设置为默认值
* 下载并安装 Ubuntu Linux 发行版（可能需要重新启动)

## 网络互通

Windows 访问 WSL2 启动的网络服务，可以直接使用 localhost，但是 Linux 访问 Windows 启动的网络服务这种方式就不行了，可以使用如下脚本获取 Windows 的 IP，并使用 IP 访问 Windows：

```ps1
ip route | grep default | awk '{print $3}'
```

## 文件系统互通

WSL2 访问 Windows 文件系统依然通过挂载分区的方式，Windows 下的磁盘会被挂载在 `/mnt` 下，例如 `/mnt/c`。

相比于 WSL1，这次增加了 Windows 访问 Linux 分区的能力，可以在资源管理器中输入 \\wsl$\<子系统名> 访问对应的子系统分区，为了方便也可以在资源管理器中把 Linux 分区挂载成一个磁盘。

更加方便的一个方式是，在 Terminal 中，使用 explorer.exe . 可以直接调用资源管理器打开当前目录，有点类似 Mac 下的 `open .`。

## cmder

cmder下载： [http://cmder.net/](http://cmder.net/)

## 修改源

```bash
sudo vim /etc/apt/sources.list
# 使用vim命令删除:直接连着按下d键

# 按下i键，拷贝下面的命令修改为阿里云的源

deb http://mirrors.163.com/ubuntu/ zesty main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ zesty-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ zesty-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ zesty-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ zesty-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ zesty main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ zesty-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ zesty-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ zesty-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ zesty-backports main restricted universe multiverse

# 按下 : vim 命令 wq 保存退出

sudo apt-get update
sudo apt-get upgrade
```

## ubuntu命令

```bash
// 系统
# uname -a               # 查看内核/操作系统/CPU信息
# head -n 1 /etc/issue   # 查看操作系统版本
# cat /proc/cpuinfo      # 查看CPU信息
# hostname               # 查看计算机名
# lspci -tv              # 列出所有PCI设备
# lsusb -tv              # 列出所有USB设备
# lsmod                  # 列出加载的内核模块
# env                    # 查看环境变量

// 资源
# free -m                # 查看内存使用量和交换区使用量
# df -h                  # 查看各分区使用情况
# du -sh <目录名>         # 查看指定目录的大小
# grep MemTotal /proc/meminfo   # 查看内存总量
# grep MemFree /proc/meminfo    # 查看空闲内存量
# uptime                 # 查看系统运行时间、用户数、负载
# cat /proc/loadavg      # 查看系统负载

// 磁盘和分区
# mount | column -t      # 查看挂接的分区状态
# fdisk -l               # 查看所有分区
# swapon -s              # 查看所有交换分区
# hdparm -i /dev/hda     # 查看磁盘参数(仅适用于IDE设备)
# dmesg | grep IDE       # 查看启动时IDE设备检测状况

// 网络
# ifconfig               # 查看所有网络接口的属性
# iptables -L            # 查看防火墙设置
# route -n               # 查看路由表
# netstat -lntp          # 查看所有监听端口
# netstat -antp          # 查看所有已经建立的连接
# netstat -s             # 查看网络统计信息

// 进程
# ps -ef                 # 查看所有进程
# top                    # 实时显示进程状态

// 用户
# w                      # 查看活动用户
# id <用户名>             # 查看指定用户信息
# last                    # 查看用户登录日志
# cut -d: -f1 /etc/passwd   # 查看系统所有用户
# cut -d: -f1 /etc/group    # 查看系统所有组
# crontab -l             # 查看当前用户的计划任务

// 服务
# chkconfig --list       # 列出所有系统服务
# chkconfig --list | grep on    # 列出所有启动的系统服务
```














参考：

[安装 WSL](https://docs.microsoft.com/zh-cn/windows/wsl/install)

[面向开发者的 WSL2 安装指南](https://zhuanlan.zhihu.com/p/145488247)

[Ubuntu修改apt-get源](https://www.cnblogs.com/TechSnail/p/7754969.html)
