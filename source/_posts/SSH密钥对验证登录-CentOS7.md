---
title: SSH密钥对验证登录-CentOS7
date: 2018-08-07 09:15:05
tags:
- Linux
- CentOS7
- SSH
categories: 
- SSH
---

# SSH密钥对验证登录

SSH为`Secure Shell`的缩写，SSH是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。Linux一般自带的是OpenSSH，可以用 `ssh -V` 查看当前版本。

```bash
[toor@localhost .ssh]$ ssh -V
OpenSSH_6.6.1p1, OpenSSL 1.0.1e-fips 11 Feb 2013
```

## 生成密钥对

```bash
ssh-keygen -t rsa
```

回车会在~/.ssh目录（用户所在家目录下的.ssh目录，如果没有请自行创建.ssh目录）生成id_rsa, id_rsa.pub文件
第一个是私有密钥 第二个是公共密钥。

将用户.ssh目录内的两个密钥下载到本地.

## 配置ssh使用密钥

```bash
cp id_rsa.pub authorized_keys
```

修改authorized_keys权限为700（必须修改为700，不能为其他）

```bash
chmod 700 authorized_keys
```

修改配置文件：

`vim /etc/ssh/sshd_config`

```bash
StrictModes no  
RSAAuthentication yes
PubkeyAuthentication yes
```

重启ssh服务

```bash
systemctl restart sshd.service
```

## win下使用puttygen生成.ppk文件

![QQ截图20180807163438.png](/img/QQ截图20180807163438.png)

![QQ截图20180807163528.png](/img/QQ截图20180807163528.png)

![QQ截图20180807163703.png](/img/QQ截图20180807163703.png)

![QQ截图20180807163742.png](/img/QQ截图20180807163742.png)

## 测试连接centos主机

使用pageant Key List管理.ppk文件，如下：

![QQ截图20180807164027.png](/img/QQ截图20180807164027.png)

![QQ截图20180807164414.png](/img/QQ截图20180807164414.png)

参考：

[linux使用密钥+密码登录ssh（centos7）](https://www.jianshu.com/p/b1eeb3f1a2c1)

[CentOS 7配置SSH基于密钥对验证登录](http://pcvc.net/blog/2015/08/10/centos-7-configuring-ssh-key-based-authentication-login/)