---
title: 解决ssh-keygen不是内部或外部命令
date: 2019-04-11 16:08:08
tags: 
- Git
- hexo
- SSH
categories: Git
---
# 解决ssh-keygen不是内部或外部命令

使用Git生成SSH密钥时，cmd报错：ssh-keygen不是内部或外部命令。

解决：

1，设置环境变量（推荐方法）

找到系统->高级系统设置->找到高级-> 环境变量-> 找到Path变量->选择编辑-添加ssh-keygen.exe路径，如下图：

![微信截图_20190411163954.png](/img/微信截图_20190411163954.png)


2，找到Git/usr/bin目录下的ssh-keygen.exe

参考：

[ssh-keygen不是内部或外部命令](https://blog.csdn.net/zy_281870667/article/details/50443403)