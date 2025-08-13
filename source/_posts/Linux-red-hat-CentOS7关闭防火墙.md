---
title: Linux Red hat/CentOS7关闭防火墙
date: 2020-09-23 11:06:19
tags:
- Red Hat
- Linux
- CentOS7
categories: 
- Linux
---

```sh
# 查看防火状态
# CentOS7
systemctl status firewalld
# Red hat
service iptables status

# 暂时关闭防火墙
# CentOS7
systemctl stop firewalld
# Red hat
service iptables stop

# 重启防火墙
# CentOS7
systemctl enable firewalld
# Red hat
service iptables restart

# 永久关闭防火墙
# CentOS7
systemctl disable firewalld
# Red hat
chkconfig iptables off
```