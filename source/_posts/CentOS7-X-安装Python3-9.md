---
title: CentOS7.X 安装Python3.9
date: 2020-11-03 10:43:20
tags:
- Linux
- CentOS7
- Python3
- AWS
- Shadowsocks
- 安装部署
categories:
- Python3
---

## 安装依赖

```sh
yum -y install wget gcc automake autoconf libtool make
```

## 安装

```sh
mkdir /shadowsocks
cd /shadowsocks
wget https://www.python.org/ftp/python/3.9.0/Python-3.9.0.tgz

tar -zxvf Python-3.9.0.tgz
cd Python-3.9.0
./configure
make && make install

# 检查版本
python3 -V
```