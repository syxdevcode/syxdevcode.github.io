---
title: TortoiseGit和Git使用相同的秘钥
date: 2023-10-08 15:53:21
tags:
  - Git
  - TortoiseGit
categories:
  - Git
---

## 方法1

![Snipaste_2023-10-08_15-32-57.png](/img1/Snipaste_2023-10-08_15-32-57.png)

加载 PuTTYgen生成的私钥，导出openssh的秘钥：

![Snipaste_2023-10-08_15-34-10.png](/img1/Snipaste_2023-10-08_15-34-10.png)

保存文件名为`id_rsa`的文件，不需要拓展名（保存到 `c:\用户\用户名\.ssh`，即Git默认的ssh存储地址）

保存公钥：将配置 `TortoiseGit` 密钥时的公钥，复制一份，保存到`c:\用户\用户名\.ssh`，命名为`id_rsa.pub`。当然也可以就着上一步私钥转换的时候，直接保存公钥到默认文件夹。

这样，相当于配置了一回 Git 的 SSH (不通过 ssh-keygen -t rsa-C “邮箱” 命令配置，而是通过 PuTTYgen 将 TortoiseGit 的密钥转换成合适的格式) ，让 openSSH 与 PuTTY 使用相同的密钥，Git Bash 使用 openSSH 连接，而 TortoiseGit 使用 PuTTY 连接，互相不干扰。

## 方法2

TortoiseGit--> Settings，将Network（网络）中的SSH client（SSH客户端）改为Git目录下的ssh.exe（C:\Program Files\Git\usr\bin）。

![Snipaste_2023-10-08_15-50-28.png](/img1/Snipaste_2023-10-08_15-50-28.png)

## 方法3

重新运行向导

![C:\Program Files\Git\usr\bin](/img1/Snipaste_2023-10-08_15-49-06.png)

## 配置多个ssh

找到 `C:\Program Files\Git\etc\ssh`目录，找到 `ssh_config`，在文件最后一行添加:

```cfg
Host github.com
    User syxdevcode  
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_rsa
    ServerAliveInterval 300
    ServerAliveCountMax 10
```

## 错误排查

no matching host key type found. Their offer: ssh-rsa,ssh-dss

找到 `C:\Program Files\Git\etc\ssh`目录，找到 `ssh_config`，在文件最后一行添加

```t
Host *
    KexAlgorithms +diffie-hellman-group1-sha1
    HostkeyAlgorithms +ssh-dss,ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-dss,ssh-rsa
```
