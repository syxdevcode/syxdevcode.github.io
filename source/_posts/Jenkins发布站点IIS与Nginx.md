---
title: Jenkins发布站点IIS与Nginx
date: 2023-08-22 13:55:18
tags:
  - Ubuntu
  - Jenkins
  - Docker Compose
  - Docker
  - DotNetCore
  - Windows Server
categories:
  - Jenkins
---

## Powershell脚本

在服务器创建 `deploy.ps1`  powershell脚本，具体代码如下：
注意：前端代码的备份文件夹的上级目录需要手动创建一次。

```ps1
## IIS站点发布
cd "D:\Git\workspace\mywebsite\Admin.NET"
dotnet publish -p:PublishDir=D:\Git\publish\mywebsite --arch x64
$website="mywebsiteApiNew" # 站点名称
$sourceFolder='D:\Git\publish\mywebsite' # 发布的源文件路径
$url="10.10.0.3"
$port="8100"
$username="test" #需要在IIS管理用户中进行创建
$password="123123"
$webDeployFolder='C:\Program Files\IIS\Microsoft Web Deploy V3'
$msdeploy=Join-Path -Path $webDeployFolder -ChildPath "msdeploy.exe"
$scriptFolder=Join-Path -Path $webDeployFolder -ChildPath "Scripts"
cd "$scriptFolder"
. .\BackupScripts.ps1
# Turns on all backup functionality
TurnOn-Backups -On $true   # 启用备份
# Turns off all backup functionality
# TurnOn-Backups -On $false  # 关闭备份
# Changes default global backup behavior to enabled
Configure-Backups -Enabled $true
# Changes default backup behavior for site to enabled
Configure-Backups -SiteName "$website" -Enabled $true
# Changes the path of where backups are stored to a sibling directory named "siteName_snapshots".  
# For more information about path variables, see the "backupPath" attribute in the section 
# "Configuring  Backup Settings on the Server for Global usage manually in IIS Config"
Configure-Backups -BackupPath "{SitePathParent}\{siteName}_snapshots"
# 最大备份文件数 Configures default backup limit to 5 backups
Configure-Backups -NumberOfBackups 8
# Configures sync behavior to fail if a sync fails for any reason
Configure-Backups -ContinueSyncOnBackupFailure $false
# Adds providers to skip when performing a backup
Configure-Backups -AddExcludedProviders @("dbmysql","dbfullsql")
# StopAppPool:
& $msdeploy -verb:sync -allowUntrusted -source:recycleApp -dest:recycleApp="$website",recycleMode="StopAppPool",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic"
# sync
& $msdeploy -verb:sync -allowUntrusted -source:contentPath=$sourceFolder -dest:contentPath="$website/",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic" -skip:objectName=dirPath,absolutePath='Configuration' -skip:objectName=dirPath,absolutePath='logs'
# StartAppPool:
& $msdeploy -verb:sync -allowUntrusted -source:recycleApp -dest:recycleApp="$website",recycleMode="StartAppPool",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic"
# 前端站点发布
# vue nginx 站点发布
$baseDir="D:\Git\workspace\mywebsite\Web"
& cd $baseDir
& yarn build
$sourceFolder=Join-Path -Path $baseDir -ChildPath "dist" # 源文件
$webSiteFolder="D:\WebSite\mywebsiteNew" # 站点根目录
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

在 Jenkins => Build Steps 中配置 PowerShell脚本如下：

```ps1
$webSiteFolder="D:\Git\workspace\mywebsite" # 站点根目录
$webSiteFolder
& cd $webSiteFolder
. .\deploy.ps1
```
