---
title: github添加ssh
date: 2020-02-29 14:20:23
tags:
- Git
- hexo
- SSH
categories: Git
---
## 配置环境变量

Git根目录 --> 变量名：`Git_Home` 变量值： `C:\Program Files (x86)\Git`                             

TortoiseGit根目录 --> 变量名：`TortoiseGit_Home` 变量值：`C:\Program Files\TortoiseGit`

## git生成公钥和私钥

快捷键Win+R然后输入cmd即可进入:

```cs
-- 默认生成文件id_rsa和id_rsa.pub，文件目录：C:\Users\当前用户\.ssh
ssh-keygen –t rsa –C xxxx@xxx.com
```

## 配置ssh密钥

打开 `id_rsa.pub`,将内容拷贝到github，如下图：

在github设置页面配置：

![QQ截图20200302113117.png](/img/QQ截图20200302113117.png)

## 配置TortoiseGit

### 生成 .ppk文件

直接在cmd中打开puttygen(也可以到TortoiseGit的安装路径下找到C:\Program Files\TortoiseGit\bin\puttygen.exe)

点击下图中规定load加载私钥id_rsa，然后点击Save private key生成TortoiseGit需要使用的ppk文件（id_rsa.ppk）

### 配置仓库

![QQ截图20200302114025.png](/img/QQ截图20200302114025.png)

![QQ截图20200302113903.png](/img/QQ截图20200302113903.png)

参考：

[Git-TortoiseGit完整配置流程](https://www.cnblogs.com/popfisher/p/5466174.html)
