---
title: Linux中PAM模块
date: 2020-08-04 14:37:39
tags:
- Linux
- CentOS7
- PAM
categories:
- PAM
---

## pam简介

**PAM：全称 Pluggable Authentication Modules，中文名 插入式认证模块。**

**Linux-PAM (Linux可插入认证模块)**：是一套适用于Linux的身份验证共享库系统，它为系统中的应用程序或服务提供动态身份验证模块支持。在Linux中，PAM是可动态配置的，本地系统管理员可以自由选择应用程序如何对用户进行身份验证。PAM应用在许多程序与服务上，比如登录程序(login、su)的PAM身份验证（口令认证、限制登录），passwd强制密码，用户进程实时管理，向用户分配系统资源等。

PAM的主要特征是认证的性质是可动态配置的。PAM的核心部分是库（libpam）和PAM模块的集合，它们是位于文件夹 `/lib/security/` 中的动态链接库(.so)文件，以及位于 `/etc/pam.d/` 目录中（或者是 `/etc/pam.conf` 配置文件，centos6之后，没有这个文件了）的各个PAM模块配置文件。

使用如下命令判断程序是否使用了PAM：

```sh
[root@host-192-125-30-59 ~]# ldd /usr/bin/passwd | grep libpam
	libpam.so.0 => /lib64/libpam.so.0 (0x00007fb53bd17000)
	libpam_misc.so.0 => /lib64/libpam_misc.so.0 (0x00007fb53bb13000)
```

如看到有类似的输出，说明该程序使用了PAM，没有输出，则没有使用。

## PAM的配置文件介绍

`/etc/pam.d/` 目录包含应用程序的PAM配置文件。例如，login程序将其程序/服务名称定义为login，与之对应的PAM配置文件为 `/etc/pam.d/login` 。

### PAM配置文件语法格式

每个PAM配置文件都包含一组指令，用于定义模块以及控制标志和参数。每条指令都有一个简单的语法，用于标识模块的目的（接口）和模块的配置设置，语法格式如下：

```sh
module_interface      control_flag      module_name  module_arguments
```

例如：

```sh
[root@host-192-125-30-59 ~]# vim /etc/pam.d/system-auth-ac
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
```

如在/etc/pam.d/password-auth-ac配置文件中(CentOS)，其中一行PAM模块接口定义如下
![password-auth-ac.png](/img/password-auth-ac.png)

### PAM的模块类型

PAM为认证任务提供四种类型可用的模块接口，它们分别提供不同的认证服务：

```sh
auth	- 认证模块接口，如验证用户身份、检查密码是否可以通过，并设置用户凭据
account	- 账户模块接口，检查指定账户是否满足当前验证条件，如用户是否有权访问所请求的服务，检查账户是否到期
password  - 密码模块接口，用于更改用户密码，以及强制使用强密码配置
session	- 会话模块接口，用于管理和配置用户会话。会话在用户成功认证之后启动生效
```

单个PAM库模块可以提供给任何或所有模块接口使用。例如，pam_unix.so提供给四个模块接口使用。

### PAM控制标志

所有的PAM模块被调用时都会返回成功或者失败的结果，每个PAM模块中由多个对应的控制标志决定结果是否通过或失败。每一个控制标志对应一个处理结果，PAM库将这些通过/失败的结果整合为一个整体的通过/失败结果，然后将结果返回给应用程序。模块可以按特定的顺序堆叠。控制标志是实现用户在对某一个特定的应用程序或服务身份验证的具体实现细节。该控制标志是PAM配置文件中的第二个字段，PAM控制标志如下：

```sh
required  - 模块结果必须成功才能继续认证，如果在此处测试失败，则继续测试引用在该模块接口的下一个模块，直到所有的模块测试完成，才将结果通知给用户。
requisite  - 模块结果必须成功才能继续认证，如果在此处测试失败，则会立即将失败结果通知给用户。
sufficient  - 模块结果如果测试失败，将被忽略。如果sufficient模块测试成功，并且之前的required模块没有发生故障，PAM会向应用程序返回通过的结果，不会再调用堆栈中其他模块。
optional  - 该模块返回的通过/失败结果被忽略。当没有其他模块被引用时，标记为optional模块并且成功验证时该模块才是必须的。该模块被调用来执行一些操作，并不影响模块堆栈的结果。
include	 - 与其他控制标志不同，include与模块结果的处理方式无关。该标志用于直接引用其他PAM模块的配置参数
```

### PAM配置方法

可以使用 `man` 命令查看配置，比如要查找某个程序支持PAM模块的配置，可以使用 `man` 加模块名(去掉.so)查找说明。

如 `man pam_unix`。(模块名可以在目录/lib/security/或/lib64/security/中找到。)

参考：

[Linux安全策略配置-pam_tally2身份验证模块](https://www.cnblogs.com/klb561/p/9236360.html)

[linux中pam模块](https://www.cnblogs.com/ilinuxer/p/5087447.html)

[深入 Linux PAM 体系结构](https://www.ibm.com/developerworks/cn/linux/l-cn-pam/index.html)