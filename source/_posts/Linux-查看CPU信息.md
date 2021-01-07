---
title: Linux 查看CPU信息
date: 2021-01-07 16:04:08
tags:
- Linux
- CentOS7
- Linux优化
categories:
- Linux
---

## 查看物理CPU的个数

```sh
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

## 查看物理CPU内核的个数

```sh
cat /proc/cpuinfo | grep "cpu cores" | uniq
```

## 查看所有逻辑CPU的个数

```sh
cat /proc/cpuinfo | grep "processor" | wc -l
```

## 查看每个物理CPU中逻辑CPU的个数

```sh
cat /proc/cpuinfo | grep 'siblings' | uniq
```

## 查询CPU是否启用超线程

```sh
cat /proc/cpuinfo | grep -e "cpu cores" -e "siblings" | sort | uniq
```

如果cpu cores数量是siblings数量一半，说明启动了超线程。
如果cpu cores数量和siblings数量一致，则没有启用超线程。

## 查看CPU的主频

```sh
cat /proc/cpuinfo |grep MHz|uniq 
```

## 查看CPU信息（型号）

```sh
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
```

## 查看内存信息

```sh
cat /proc/meminfo
```

## 查看机器型号 

```sh
dmidecode | grep "Product Name"  
```

## CPU运行位数

```sh
getconf LONG_BIT
```