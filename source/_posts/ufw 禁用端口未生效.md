---
title: ufw 禁用端口未生效
date: 2022-09-08 14:36:07
tags:
  - Ubuntu
  - Linux基础
  - Docker
  - Docker Compose
  - Linux
  - ufw
  - iptables
  - 安全策略
categories:
  - 防火墙
---

iptables-save 查看防火墙状态(节选):

```shell
-A DOCKER ! -i br-fe67cfba7476 -p tcp -m tcp --dport 15000 -j DNAT --to-destination 192.168.224.2:5000
-A DOCKER ! -i br-fe67cfba7476 -p tcp -m tcp --dport 15002 -j DNAT --to-destination 192.168.224.3:5000
-A DOCKER ! -i br-fe67cfba7476 -p tcp -m tcp --dport 15001 -j DNAT --to-destination 192.168.224.4:5000
COMMIT
# Completed on Thu Sep  8 06:38:44 2022
```

在 docker 中只要有容器映射了端口 docker 就会自动加 iptables，所以根本就没走到 ufw 端口就被放行了。

参考：

[ufw 禁用端口未生效](https://www.cnblogs.com/hujingnb/p/15579099.html)
