---
title: Windows Server 2012/2012R2配置MSDTC
date: 2021-05-08 15:29:56
tags:
- Windows Server 2012/2012R2
- MSDTC
- 分布式事务
categories: MSDTC
---

## 配置MSDTC

点击打开 `开始`—>`管理工具`—>`组件服务` （或者 Win+R 输入：`Dcomcnfg.exe`）

`组件服务`—>`计算机`—>`我的电脑`—>`Distributed Transaction Coordinator`，右击`本地DTC`，选择属性，按下图进行设置，设置完成后点击确定。

<!--more-->
![微信截图_20210508153523.png](/img/微信截图_20210508153523.png)

![微信截图_20210508154146.png](/img/微信截图_20210508154146.png)

## 配置注册表

运行 ：`Regedt32.exe`，找到 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\MSDTC`，添加 `DWord (32位)`，输入 `ServerTcpPort`，选择 十进制，值输入：5000。

![微信截图_20210521163327.png](/img/微信截图_20210521163327.png)

![微信截图_20210508154445.png](/img/微信截图_20210508154445.png)

![微信截图_20210508154527.png](/img/微信截图_20210508154527.png)

## 135端口

打开双方 `135` 端口 `MSDTC` 服务依赖于 `RPC`（`Remote Procedure Call (RPC)`）服务，`RPC` 使用 `135` 端口，保证 `RPC` 服务启动，如果服务器有防火墙，保证 `135` 端口不被防火墙挡住。

## 验证

停止，启动 MSDTC。

```sh
# 启动MSDTC服务：
net start msdtc

# 停止MSDTC服务：
net stop msdtc

# 卸载MSDTC服务：
msdtc -uninstall

# 重新安装MSDTC服务：
mstdc -install
```

确认 `MSDTC` 使用了正确的端口：

打开管理命令提示符，以管理员方式运行 `Netstat -ano` 来获取端口和进程标识符（PID）
启动 `Task Manager`，找到 `MSDTC.exe` 并获取 PID
浏览输出并显示其是 `MSDTC`

![微信截图_20210508154947.png](/img/微信截图_20210508154947.png)

参考：

[Windows Server 2012/2012R2 中配置 MSDTC，令其使用特定端口](https://blog.csdn.net/xiaoye1202/article/details/110938912)