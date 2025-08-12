---
title: Git gpg认证
date: 2025-08-12 14:35:50
tags:
- Git
- Gpg
categories: 
- Git
---

## 安装gpg

### windows

 https://www.gpg4win.org/

## 生成密钥

```
gpg --gen-key
```

## 查看密钥

```
gpg --list-keys
```

## 导出公钥

```
gpg --export --armor 邮箱 > 公钥文件名
```

## 导入公钥

```
gpg --import 公钥文件名
```

## 签名

```
gpg --sign --armor 文件名
```

## 验证

```
gpg --verify 文件名.asc
```

## 配置git


```
git config --global user.signingkey 密钥ID
git config --global commit.gpgsign true

# 配置gpg程序路径
git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe"

# 配置git提交签名
git config commit.gpgsign true
```

密钥ID为上一步生成的密钥ID的后8位

在路径
`C:\Users\Administrator` 下找到 `.gitconfig`，复制一份，重命名为 `.gitconfig-gitlab`，

`.gitconfig` 内容如下：



```
[user]
	name = xxxxxx
	email = xxxxxx@qq.com
	
# The contents of this file are included only for GitLab.com URLs
[includeIf "hasconfig:remote.*.url:https://gitlab.xxxxx.com/**"]

# Edit this line to point to your alternative configuration file
path = ~/.gitconfig-gitlab

[commit]
	gpgsign = false
[gpg]
	program = C:\\Program Files (x86)\\GnuPG\\bin\\gpg.exe

```

`.gitconfig-gitlab` 内容如下：

```
[user]
	name = xx
	email = xxx@xxx.com
	signingkey = 密钥ID
[commit]
	gpgsign = true

```

## 配置github

1. 复制公钥
2. 登录github
3.  Settings -> SSH and GPG keys -> New GPG key
4. 粘贴公钥
5. 测试
    ```
    git commit -S -m "测试"
    ```
6. 推送
    ```
    git push
    ```
7. 查看
    ```
    git log --show-signature
    ```