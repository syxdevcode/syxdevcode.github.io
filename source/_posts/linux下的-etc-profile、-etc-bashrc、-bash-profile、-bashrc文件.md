---
title: linux下的/etc/profile、/etc/bashrc、~/.bash_profile、~/.bashrc文件
date: 2020-12-15 16:57:09
tags:
- Linux
- CentOS7
- Ubuntu
- Linux基础
categories: Linux基础
---

## /etc/profile

为系统的每个用户设置环境信息。当用户第一次登录时，该文件被执行。并从 /etc/profile.d 目录的配置文件中收集 shell 的设置。如果修改 /etc/profile ，需要执行 source /etc/profile 才会生效，此修改对每个用户都生效。

## /etc/bashrc（ubuntu 为 /etc/bash.bashrc）

为每一个运行 bash shell 的用户执行此文件。当 bash shell 被打开时，该文件被读取。如果你想对所有的使用 bash 的用户修改某个配置并在以后打开的 bash 都生效的话可以修改这个文件，修改这个文件不用重启，重新打开一个 bash 即可生效。
Ubuntu 与之对应的是 /ect/bash.bashrc。

## ~/.bash_profile（ubuntu为 ~/.profile）

每个用户都可使用该文件输入专用于自己使用的 shell 信息，当用户登录时，该文件仅仅执行一次！默认情况下,它设置一些环境变量，执行用户的 `~/.bashrc` 文件。 此文件类似于 /etc/profile，也是需要需要 source 才会生效，/etc/profile 对所有用户生效，~/.bash_profile 只对当前用户生效。

## ~/.bashrc

该文件包含专用于你的 bash shell 的 bash 信息，当登录时以及每次打开新的 shell 时，该文件被读取。（每个用户都有一个 ~/.bashrc 文件，在用户目录下） 此文件类似于 /etc/bashrc，不需要重启就可以生效，重新打开一个 bash 即可生效，/etc/bashrc 对所有用户新打开的 bash 都生效，但 ~/.bashrc 只对当前用户新打开的 bash 生效。
~/.bashrc 文件会在 bash shell 调用另一个 bash shell 时读取，也就是在 shell 中再键入 bash 命令启动一个新 shell 时就会去读该文件。这样可有效分离登录和子 shell 所需的环境。但一般 来说都会在 /.bash_profile 里调用 /.bashrc 脚本以便统一配置用户环境。

## ~/.bash_logout:

当每次退出系统(退出 bash shell)时，执行该文件。可把一些清理工作的命令放到这文件中。

## 区别和联系

* 在 /etc 目录是系统级（全局）的配置文件，当在用户主目录下找不到 ~/.bash_profile 和 ~/.bashrc 时，就会读取这两个文件。
* /etc/profile 中设定的变量(全局)的可以作用于任何用户，而 ~/.bashrc 中设定的变量(局部)只能继承 /etc/profile 中的变量，他们是“父子”关系。
* ~/.bash_profile 是交互式、login 方式进入 bash 运行的； ~/.bashrc 是交互式 non-login 方式进入 bash 运行的。通常二者设置大致相同，所以通常前者会调用后者。设置生效：可以重启生效，也可以使用命令：source。
* ~/.bash_history 是 bash shell 的历史记录文件，里面记录了你在 bash shell 中输入的所有命令。可通过 HISSIZE 环境变量设置在历史记录文件里保存记录的条数。

## 执行顺序

### 1,登录Linux

首先启动 /etc/profile 文件，
然后再启动用户目录下的 ~/.bash_profile、 ~/.bash_login 或 ~/.profile 文件中
的其中一个，执行的顺序为：~/.bash_profile、 ~/.bash_login、 ~/.profile
其中 ~/.bash_profile、 ~/.bash_login 和 ~/.profile 文件往往只存在一个，这与 Linux 的发行版本有关。centos中为 ~/.bash_profile，ubuntu 则为 ~/.profile。

以上两个文件会在用户登录时执行。

### 2,执行用户的bash设置

如果 ~/.bash_profile 文件存在的话，一般会以这样的方式执行用户的 ~/.bashrc 文件。

执行顺序为:

* /etc/profile
* ~/.bash_profile | ~/.bash_login | ~/.profile
* ~/.bashrc
* /etc/bashrc
* ~/.bash_logout

## 其他

图形模式登录时，顺序读取：/etc/profile 和 ~/.profile
图形模式登录后，打开终端时，顺序读取：/etc/bash.bashrc 和 ~/.bashrc
文本模式登录时，顺序读取：/etc/bash.bashrc，/etc/profile 和 ~/.bash_profile
从其它用户 su 到该用户，则分两种情况：
（1）如果带 -l 参数（或-参数，–login参数），如：su -l username，则 bash 是 lonin 的，它将顺序读取以下配置文件：/etc/bash.bashrc，/etc/profile和~ /.bash_profile。
（2）如果没有带-l参数，则bash是non-login的，它将顺序读取：/etc/bash.bashrc 和 ~/.bashrc
注销时，或退出su登录的用户，如果是 longin 方式，那么 bash 会读取：~/.bash_logout
执行自定义的 shell 文件时，若使用 bash -l a.sh 的方式，则 bash 会读取行：/etc/profile 和 ~/.bash_profile，若使用其它方式，如：bash a.sh， ./a.sh，sh a.sh（这个不属于bash shell），则不会读取上面的任何文件。

参考：

[linux下的/etc/profile、/etc/bashrc、~/.bash_profile、~/.bashrc文件](https://www.jianshu.com/p/6d32b166f47d)

[linux alias 永久别名 永久有效](https://www.jianshu.com/p/5a54aeaf2b9e)