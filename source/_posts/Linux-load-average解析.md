---
title: Linux load average解析
date: 2021-04-01 14:33:15
tags:
- Linux
- CentOS7
- Linux基础命令
- Linux优化
categories:
- Linux优化
---

`load average` 分别代表最近1分钟、5分钟、15分钟内系统的平均负荷。

可以使用 `uptime` ，`w` 和 `top` 命令查看。`load average` 的值越低，比如等于0.2或0.3，就说明电脑的工作量越小，系统负荷比较轻。

## 系统负荷的经验法则

* 当系统负荷持续大于0.7，你必须开始调查了，问题出在哪里，防止情况恶化。
* 当系统负荷持续大于1.0，你必须动手寻找解决办法，把这个值降下来。
* 当系统负荷达到5.0，就表明你的系统有很严重的问题，长时间没有响应，或者接近死机了。

## 多处理器

`系统负荷=电脑CPU个数`。如 `n` 个CPU的电脑，可接受的系统负荷最大为 `n.0`。

查看物理CPU的个数:

```sh
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

## 多核处理器

一个CPU内部，包含多个CPU核心，这被称为多核CPU。

在系统负荷方面，多核CPU与多CPU效果类似，所以考虑系统负荷的时候，必须考虑这台电脑有几个CPU、每个CPU有几个核心。然后，把 `系统负荷 除以 总的核心数`，只要每个核心的负荷不超过1.0，就表明电脑正常运行。

```sh
# 直接返回CPU的总核心数
grep -c 'model name' /proc/cpuinfo

# 或
cat /proc/cpuinfo | grep "processor" | wc -l
```

参考：

[理解Linux系统负荷](http://www.ruanyifeng.com/blog/2011/07/linux_load_average_explained.html)