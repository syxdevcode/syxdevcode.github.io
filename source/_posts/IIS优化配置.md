---
title: IIS优化配置
date: 2022-01-26 16:16:19
tags:
- IIS
categories:
- IIS
---

```ps1
<!-- 调大应用程序池的请求队列 -->
c:\windows\system32\inetsrv\appcmd.exe set apppool 审核Api -queueLength:65535

<!-- 取消固定时间间隔的回收 -->
c:\windows\system32\inetsrv\appcmd.exe set apppool 审核Api -recycling.periodicRestart.time:00:00:00

<!-- 设置指定时间点的回收 -->
c:\windows\system32\inetsrv\appcmd.exe set apppool 审核Api /+recycling.periodicRestart.schedule.[value='04:15:00']

<!-- 取消空闲超时的进程关闭 -->
c:\windows\system32\inetsrv\appcmd.exe set apppool 审核Api -processModel.idleTimeout:00:00:00

<!-- 开启所有回收事件的日志 -->
c:\windows\system32\inetsrv\appcmd.exe set apppool 审核Api -recycling.logEventOnRecycle:"Time, Requests, Schedule, Memory, IsapiUnhealthy, OnDemand, ConfigChange, PrivateMemory"

<!-- IIS并发请求数 -->
c:\windows\system32\inetsrv\appcmd.exe set config /section:serverRuntime /appConcurrentRequestLimit:100000

// 注册表 将最大连接数设置为10万
reg add HKLM\System\CurrentControlSet\Services\HTTP\Parameters /v MaxConnections /t REG_DWORD /d 100000
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\HTTP\Parameters /v MaxFieldLength /t REG_DWORD /d 32768
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\HTTP\Parameters /v MaxRequestBytes /t REG_DWORD /d 32768 

// 修改TCP MaxUserPort限制（由默认5000改为65534）
reg add HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters /v MaxUserPort /t REG_DWORD /d 65534 

// 重启
net stop http  & net start http & iisreset
```
