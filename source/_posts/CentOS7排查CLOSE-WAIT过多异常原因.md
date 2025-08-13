---
title: CentOS7排查CLOSE_WAIT过多异常原因
date: 2021-03-29 15:33:01
tags:
---

## 查看系统连接数

```sh
# 获取当前socket连接状态统计信息
cat /proc/net/sockstat

# 统计当前各种状态的连接的数量的命令
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

## 查看端口范围

```sh
# 允许系统打开的端口范围，用于向外链接的端口范围
cat /proc/sys/net/ipv4/ip_local_port_range
1024	65000
```
<!--more-->
## 查看文件打开数

```sh
# 查看当前打开文件数 
# 如果执行缓慢，那么执行结果显示系统当前打开文件数500w
lsof | wc -l

# 把当前打开文件列表保存
lsof>>/tmp/lsof.log  

# 查看 5-6万 行之间的数据
sed -n '50000,60000p' /tmp/lsof.log
```

结果，应用大量调用redis，且没有关闭：

![微信截图_20210329155921.png](/img/微信截图_20210329155921.png)

## 排查代码

找到如下问题：每次调用工具类，都 new 一个 csredis 客户端，导致连接数过多。

![微信截图_20210329160151.png](/img/微信截图_20210329160151.png)

修复：删除工具类，使用csredis静态类。