---
title: Jenkins发布Nginx站点
date: 2023-08-21 09:23:27
tags:
  - Nginx
  - Jenkins
  - Windows Server
categories:
  - Nginx
---

## PowerShell脚本

脚本中使用到了 yarn 工具，需要提前在系统中安装。

```ps1
# vue nginx 站点发布
$baseDir="D:\Git\workspace\mywebsite\Web"
& cd $baseDir
& yarn build
$sourceFolder=Join-Path -Path $baseDir -ChildPath "dist" # 源文件
$webSiteFolder="D:\WebSite\mywebsite" # 站点根目录
$webSiteFolder
$destFolder=Join-Path -Path $webSiteFolder -ChildPath "web" # 手动创建web目录
$destFolder
$backFolder=Join-Path -Path $webSiteFolder -ChildPath "web_back" # 手动创建web_back目录
$backFolder
$now=Get-Date -Format "yyyy-MM-dd_HHmmss"
$backFileName=-join ($now,".zip")
$fullFolder=Join-Path -Path $backFolder -ChildPath $backFileName
Compress-Archive -Path $destFolder -DestinationPath $fullFolder -Force  # 1,备份成压缩包
$removeFolder=-join($destFolder,"\*")
Remove-Item $removeFolder -Recurse -Force # 2,删除 原站点 所有文件/文件夹
$fullSourceFolder=-join ($sourceFolder,"\*")
Copy-Item -Path $fullSourceFolder -Destination $destFolder -Recurse # 3，复制新文件
```

参考：

[使用PowerShell压缩和解压ZIP包](https://www.cnblogs.com/cqpanda/p/16153860.html)
