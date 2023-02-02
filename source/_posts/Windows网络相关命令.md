---
title: Windows网络相关命令
date: 2023-02-02 15:16:15
tags:
  - Windows
  - Windows10
categories:
  - Windows10
---

## 网络命令

```ps1
ipconfig /flushdns
ipconfig /displaydns
ipconfig /release 
ipconfig /renew
```

netsh winsock reset 命令和作用是重置 Winsock 目录。如果电脑的Winsock协议配置有问题，则会导致网络连接出现问题，就需要使用 netsh winsock reset命令来重置Winsock目录以恢复网络。
该命令可以重新初始化网络环境，以解决由于软件冲突、病毒等原因造成的参数错误问题。

* 1 清除ARP缓存，输入命令 `arp -d *`代替执行。
* 2 清除NETBT，输入命令`nbtstat -R`代替执行。
* 3 清除DNS缓存，输入命令`ipconfig/flushdns`代替执行。
