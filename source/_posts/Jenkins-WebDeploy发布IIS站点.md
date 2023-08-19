---
title: Jenkins WebDeploy发布IIS站点
date: 2023-08-19 13:57:28
tags:
  - IIS
  - Jenkins
  - Windows Server
categories:
  - IIS
---

## 配置Windows Server服务器

### 安装.NET SDK

在服务器上下载安装 .NET SDK。

下载地址： [下载 .NET 6.0](https://dotnet.microsoft.com/zh-cn/download/dotnet/6.0)

### 安装WebDeploy

下载地址：[web-deploy下载](https://www.iis.net/downloads/microsoft/web-deploy)

通过选择“自定义”选项来激活“IIS 部署处理程序”功能。

![Snipaste_2023-08-19_14-09-17.png](/img1/Snipaste_2023-08-19_14-09-17.png)

WebDeploy默认监听8172端口，可以按照以下操作自定义端口号：

![Snipaste_2023-08-19_14-11-13.png](/img1/Snipaste_2023-08-19_14-11-13.png)

先停止服务，然后在修改端口，之前启动即可。

![Snipaste_2023-08-19_14-11-59.png](/img1/Snipaste_2023-08-19_14-11-59.png)

<!--more-->
参考：

[WebDeploy 不监听端口 8172](https://www.coder.work/article/6683200)

### 添加IIS发布用户

![Snipaste_2023-08-19_14-15-24.png](/img1/Snipaste_2023-08-19_14-15-24.png)

![Snipaste_2023-08-19_14-15-58.png](/img1/Snipaste_2023-08-19_14-15-58.png)

选择需要配置的站点，点击：

![Snipaste_2023-08-19_14-17-05.png](/img1/Snipaste_2023-08-19_14-17-05.png)

选择在 IIS管理器用户 中添加的用户。

![Snipaste_2023-08-19_14-17-44.png](/img1/Snipaste_2023-08-19_14-17-44.png)

## 发布

### PowerShell命令

文件管理器地址栏输入以下命令，确定msdeploy.exe文件目录：

`%programfiles%\IIS\Microsoft Web Deploy V3`

通常所在目录为：C:\Program Files\IIS\Microsoft Web Deploy V3

**msdeploy -verb 对应的命令：**

* dump                         显示所指定源对象的详细信息。
* sync                         使目标对象与源对象同步。
* delete                       删除指定的目标对象。
* getDependencies              检索给定对象的依赖关系
* getParameters                返回对象支持的参数
* getSystemInfo                检索与给定对象关联的系统信息

`%programfiles%\IIS\Microsoft Web Deploy V3\scripts\BackupScripts.ps1` 找到用于在服务器级别配置备份功能的 PowerShell 脚本。

```ps1
# 执行脚本
. .\BackupScripts.ps1 # 注意两个点之间的空格
```

可以在Windows Server服务器上使用以下代码测试，之后再配置到Jenkins中：
注意：需要删除掉空格，注释。

```ps1
$website="二方审核api-新版" # 站点名称
$sourceFolder='D:\Git\publish\Pony-ErFangAudit' # 发布的源文件路径
$url="10.10.0.37"
$port="8500"
$username="webDeploy" #需要在IIS管理用户中进行创建
$password="W#Edc/@WsxPony"
$webDeployFolder='C:\Program Files\IIS\Microsoft Web Deploy V3'
$msdeploy=Join-Path -Path $webDeployFolder -ChildPath "msdeploy.exe"
$scriptFolder=Join-Path -Path $webDeployFolder -ChildPath "Scripts"
cd "$scriptFolder"
. .\BackupScripts.ps1
# *********配置备份-开始*********
# Turns on all backup functionality
TurnOn-Backups -On $true   # 启用备份
# Turns off all backup functionality
# TurnOn-Backups -On $false  # 关闭备份
# Changes default global backup behavior to enabled
Configure-Backups -Enabled $true # 启用全局备份
# Changes default backup behavior for site "二方审核api-新版" to enabled
Configure-Backups -SiteName "$website" -Enabled $true
# Changes the path of where backups are stored to a sibling directory named "siteName_snapshots".  
# For more information about path variables, see the "backupPath" attribute in the section 
# "Configuring  Backup Settings on the Server for Global usage manually in IIS Config"
Configure-Backups -BackupPath "{SitePathParent}\{siteName}_snapshots"
# 最大备份文件数 Configures default backup limit to 5 backups
Configure-Backups -NumberOfBackups 8
# Configures sync behavior to fail if a sync fails for any reason
Configure-Backups -ContinueSyncOnBackupFailure $false  #如果备份失败则不继续同步
# Adds providers to skip when performing a backup
Configure-Backups -AddExcludedProviders @("dbmysql","dbfullsql")
# *********配置备份-结束*********

# ***********操作IIS站点-开始**********
# 备份站点,发布文件的时候，会自动备份，不需要手动执行备份命令
#& $msdeploy -verb:sync -allowUntrusted -source:backupManager -dest:backupManager="$website",computername="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic" -skip:objectName=dirPath,absolutePath='wwwroot' -skip:objectName=dirPath,absolutePath='logs'

# 停止应用程序池 StopAppPool:
& $msdeploy -verb:sync -allowUntrusted -source:recycleApp -dest:recycleApp="$website",recycleMode="StopAppPool",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic"

# 同步整个文件夹至根目录，默认会删除其他文件
# & $msdeploy -verb:sync -allowUntrusted -source:contentPath=$sourceFolder -dest:contentPath="$website/",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic" -skip:objectName=dirPath,absolutePath='Configuration' -skip:objectName=dirPath,absolutePath='logs'

# 同步整个文件夹至根目录，但不要删除目标的其它文件:
& $msdeploy -verb:sync -allowUntrusted -source:contentPath=$sourceFolder -dest:contentPath="$website/",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic" -enableRule:DoNotDeleteRule

# 启动应用程序池 StartAppPool:
& $msdeploy -verb:sync -allowUntrusted -source:recycleApp -dest:recycleApp="$website",recycleMode="StartAppPool",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic"
```

**Power Shell 完整命令：**

```ps1
cd "D:\Git\workspace\二方审核\Admin.NET"
dotnet publish -p:PublishDir=D:\Git\publish\Pony-ErFangAudit --arch x64
$website="二方审核api-新版" # 站点名称
$sourceFolder='D:\Git\publish\Pony-ErFangAudit' # 发布的源文件路径
$url="10.10.0.37"
$port="8500"
$username="webDeploy" #需要在IIS管理用户中进行创建
$password="W#Edc/@WsxPony"
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
# Changes default backup behavior for site "二方审核api-新版" to enabled
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
```

Jenkins中配置如下：

![Snipaste_2023-08-19_16-16-36.png](/img1/Snipaste_2023-08-19_16-16-36.png)

### 备份，还原

```ps1
#查看所有的备份
& $msdeploy -verb:dump -source:backupManager=$website

# 用最近的一份备份还原
# 注意需要设置 enableRule:DoNotDeleteRule 防止删除其他文件
& $msdeploy -verb:sync -allowUntrusted -source:backupManager -dest:backupManager=$website,useLatest=true,computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic" -skip:objectName=dirPath,absolutePath='wwwroot'  -enableRule:DoNotDeleteRule
```

### 更新文件

```ps1
# 更新单个文件 Upload Single File
$file="C:\Text\Admin认证.txt"
& $msdeploy -verb:sync -allowUntrusted -source:contentPath=$file -dest:contentPath="$website/$([System.IO.Path]::GetFileName($file))",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic"
```

### 刪除文件/文件夹

```ps1
# 删除-跳过某些文件或文件夹
& $msdeploy -verb:delete -allowUntrusted -dest:contentPath="$website/$file",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic" -skip:absolutePath=web.config -skip:objectName=dirPath,absolutePath='Logs' -skip:objectName=dirPath,absolutePath='Temp'

# 删除单个文件
$file="Admin认证.txt"
& $msdeploy -verb:delete -allowUntrusted -dest:contentPath="$website/$file",computerName="https://${url}:${port}/msdeploy.axd?site=$website",username="$username",password="$password",AuthType="Basic"
```

参考：

[Web 部署自动备份](https://learn.microsoft.com/zh-cn/iis/publish/using-web-deploy/web-deploy-automatic-backups)

[Web 部署错误代码](https://learn.microsoft.com/zh-cn/troubleshoot/developer/webapps/iis/deployment-migration/web-deploy-error-codes)

[使用WebDeploy部署到IIS](https://blog.csdn.net/ChenXi0803/article/details/128906674)