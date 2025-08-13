---
title: PowerShell：因为在此系统上禁止运行脚本，解决方法
date: 2021-09-04 13:58:20
tags:
- Windows
- PowerShell
- Windows10
categories: PowerShell
---

## 报错详情

```ps1
PS E:\code> hexo server
hexo : 无法加载文件 C:\Users\Administrator\AppData\Roaming\npm\hexo.ps1，
因为在此系统上禁止运行脚本。有关详细信息，请参阅 
https:/go.microsoft.com/fwlink/?LinkID=135170 中的about_Execution_Policies。
所在位置 行:1 字符: 1
+ hexo new "PowerShell：因为在此系统上禁止运行脚本，解决方法"
+
    + CategoryInfo          : SecurityError: (:) []，PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

计算机上启动 `Windows PowerShell` 时，执行策略很可能是 `Restricted`（默认设置）。

* `Restricted` 执行策略不允许任何脚本运行。  
* `AllSigned` 和 `RemoteSigned` 执行策略可防止 `Windows PowerShell` 运行没有数字签名的脚本。

计算机上的现用执行策略，打开 `PowerShell` 然后输入 ：`get-executionpolicy`

```ps1
PS C:\WINDOWS\system32> get-executionpolicy
Restricted
```

## 设置

以管理员身份打开 `PowerShell` 输入 `set-executionpolicy remotesigned`，并输入Y。

```ps1
PS C:\WINDOWS\system32> set-executionpolicy remotesigned

执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170
中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): y
PS C:\WINDOWS\system32> get-executionpolicy
RemoteSigned
```

参考：

[PowerShell：因为在此系统上禁止运行脚本，解决方法](https://www.jianshu.com/p/4eaad2163567)
