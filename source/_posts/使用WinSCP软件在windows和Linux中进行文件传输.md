---
title: 使用WinSCP软件在windows和Linux中进行文件传输
date: 2017-05-24 14:24:14
tags:
- Linux
- WinSCP
- 同步软件
categories:
- WinSCP
---
# 使用WinSCP软件在windows和Linux中进行文件传输

官方的解释：WinSCP 是一个 Windows 环境下使用 SSH 的开源图形化 SFTP 客户端。同时支持 SCP 协议。它的主要功能就是在本地与远程计算机间安全的复制文件等。

官网网站：[http://winscp.net/eng/docs/lang:chs](http://winscp.net/eng/docs/lang:chs "官网")

下载地址：[http://winscp.net/eng/download.php](http://winscp.net/eng/download.php "下载")

Portable executables是绿色版，不需要安装。
下载完成之后打开可执行文件，填写登录信息，选择协议之后，就可以进行图形化管理了。

![配置](/img/winscp-配置.png)

我们只需要填写3个地方：1. host name 2.user name 3.password。hostname是虚拟机的IP地址。
最好是填写root用户时的用户名和密码。点击登陆就进入到Linux系统了
界面中，左边属于windows操作系统的目录，右边属于Linux（CentOS）操作系统的目录。可以用鼠标直接把文件拖过来拖过去的。

参考：

[http://www.cnblogs.com/shanyou/archive/2012/08/25/2656753.html](http://www.cnblogs.com/shanyou/archive/2012/08/25/2656753.html "winscp配置")