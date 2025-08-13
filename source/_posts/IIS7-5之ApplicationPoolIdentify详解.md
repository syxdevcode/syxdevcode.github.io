---
title: IIS7.5之ApplicationPoolIdentify详解
date: 2019-04-22 16:06:28
tags:
- ASP.NET/IIS请求处理管道
- ApplicationPoolIdentify
categories: ApplicationPoolIdentify
---
# IIS7.5之ApplicationPoolIdentify详解

`ApplicationPoolIdentify` 在 Windows Server 2008 和 Windows Vista 的 Service Pack 2（SP2）和更高版本的Windows引入的强大的新隔离功能，IIS7.5中首次引用。其它三种包括 `LocalService` , `LocalSystem`,`NetWorkService`。

`ApplicationPoolIdentify` 允许在唯一帐户下运行应用程序池，而无需创建和管理域或本地帐户。应用程序池帐户的名称对应于应用程序池的名称。

下图显示了作为DefaultAppPool标识运行的IIS工作进程（W3wp.exe）。

![image3.jpg](/img/image3.jpg)

Windows操作系统提供称为“虚拟帐户”的功能，允许IIS为其每个应用程序池创建唯一标识。


## 配置IIS ApplicationPoolIdentify

如果在Windows Server 2008 R2或更高版本的IIS上运行IIS 7.5，则无需执行任何操作即可使用新标识。创建的每个应用程序池，默认情况下，新应用程序池的Identity属性设置为ApplicationPoolIdentity。IIS管理进程（WAS）将创建一个具有新应用程序池名称的虚拟帐户，并默认在此帐户下运行应用程序池的工作进程。

要在Windows Server 2008上运行IIS 7.0时使用此虚拟帐户，必须将创建的应用程序池的Identity属性更改为`ApplicationPoolIdentity`。

方法:

1，打开IIS管理控制台（INETMGR.MSC）
2，打开计算机节点下的“应用程序池”节点。选择要更改为在自动生成的应用程序池标识下运行的应用程序池。
3，右键单击应用程序池，然后选择高级设置。

![image6.jpg](/img/image6.jpg)

4，选择标识列表项，然后单击省略号（带有三个点的按钮）。
5，出现以下对话框

![image8.jpg](/img/image8.jpg)

6,选择内置帐户按钮，然后从组合框中选择标识类型ApplicationPoolIdentity。

要使用命令行执行相同的步骤，可以通过以下方式调用appcmd命令行工具：

```ps
%windir%\system32\inetsrv\appcmd.exe set AppPool <your AppPool> -processModel.identityType:ApplicationPoolIdentity
```

## 确保资源安全

每当创建新的应用程序池时，IIS管理进程都会创建一个安全标识符（SID），该标识符表示应用程序池本身的名称。例如，如果创建名为“MyNewAppPool”的应用程序池，则会在Windows安全系统中创建名为“MyNewAppPool”的安全标识符。从现在开始，可以使用此标识来保护资源。但是，身份不是真实的用户帐户; 它不会在Windows用户管理控制台中显示为用户。

我们可以通过在Windows资源管理器中选择一个文件并将“DefaultAppPool”标识添加到文件的访问控制列表（ACL）来尝试此操作。


1，打开Windows资源管理器
2，选择文件或目录。
3，右键单击该文件并选择"属性"
4，选择"安全"选项卡
5，单击编辑按钮，然后单击添加按钮
6，单击"位置"按钮，确保选择计算机。

![image11.jpg](/img/image11.jpg)

7，在输入要选择的对象名称：文本框中输入 `IIS AppPool\DefaultAppPool`。
8，单击"检查名称"按钮，然后单击“ 确定”。








参考：

[Application Pool Identities](https://docs.microsoft.com/en-us/iis/manage/configuring-security/application-pool-identities)

[IIS ApplicationPoolIdentity](http://www.cnblogs.com/jfzhu/p/4067297.html)

[IIS7.5中神秘的ApplicationPoolIdentity ](http://www.cnblogs.com/yjmyzz/archive/2009/10/26/1590033.html)