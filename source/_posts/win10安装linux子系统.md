---
title: win10安装linux子系统
date: 2020-01-21 16:25:13
tags:
- Linux
- Windows子系统
- Linux基础命令
categories:
- Windows子系统
---

## 安装

![QQ截图20200121163159.png](/img/QQ截图20200121163159.png)

![QQ截图20200121163433.png](/img/QQ截图20200121163433.png)

![QQ截图20200121163508.png](/img/QQ截图20200121163508.png)

Microsoft Store应用商店获取子系统

![QQ截图20200121163729.png](/img/QQ截图20200121163729.png)

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

[win10上linux子系统的开启、升级及使用](https://blog.csdn.net/helaisun/article/details/80712287)

[Ubuntu修改apt-get源](https://www.cnblogs.com/TechSnail/p/7754969.html)
