---
title: IIS配置导出与导入
date: 2022-03-29 17:11:01
tags:
- IIS
categories:
- IIS
---

## 站点

### 所有站点

```ps1
# 导出所有站点
%windir%\system32\inetsrv\appcmd list site /config /xml > c:\sites.xml

# 导入所有站点
%windir%\system32\inetsrv\appcmd add site /in < c:\sites.xml
```

### 单个站点

```ps1
# 导出单独站点
%windir%\system32\inetsrv\appcmd list site "站点名称" /config /xml > c:\mywebsite.xml

# 导入单独站点
%windir%\system32\inetsrv\appcmd add site /in < c:\mywebsite.xml
```

## 应用程序池

### 所有程序池

```ps1
# 导出所有应用程序池
%windir%\system32\inetsrv\appcmd list apppool /config /xml > c:\apppools.xml

# 导入所有应用程序池
%windir%\system32\inetsrv\appcmd add apppool /in < c:\apppools.xml
```

### 单个程序池

```ps1
# 导出单独的应用程序池
%windir%\system32\inetsrv\appcmd list apppool "应用程序池名称" /config /xml > c:\myapppool.xml

# 导入单独的应用程序池
%windir%\system32\inetsrv\appcmd add apppool /in < c:\myapppool.xml
```

参考：

[IIS配置导入导出](https://www.cnblogs.com/dingzp/p/10785100.html)