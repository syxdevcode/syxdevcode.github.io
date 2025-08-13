---
title: 解决windows系统80端口被占用问题
date: 2023-04-06 11:03:23
tags:
  - Windows
  - Windows10
  - Windows服务
categories:
  - Windows10
---

## 查找80端口占用

```ps1
# 80端口占用
netstat -ano|findstr 80

# 任务列表
tasklist 
```

<!--more-->
## 取消80端口占用

系统占用的端口一般都是微软官方的产品占用的。所以这个时候主要考虑到几个服务：

* SQL Server导致。其中很有可能是 `SQL Server Reporting Services (MSSQLSERVER)`。
* IIS 服务。如果你电脑安装了这个，很有可能它在运行着，那么它就占用着80端口

### 在服务面板关闭

管理员方式运行，输入 `services.msc` 启动服务窗口，选择对应的服务关闭。

### 使用CMD命令关闭

* 1，使用管理员身份运行 cmd
* 2，`net stop http` //停止系统http服务
* 3，`sc config http start= disabled` //禁用服务的自动启动，此处注意等号后面的空格不可少

参考：

[解决windows系统80端口被占用问题](https://blog.csdn.net/the_liang/article/details/81914920)
