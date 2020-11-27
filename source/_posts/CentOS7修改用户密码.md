---
title: CentOS7修改用户密码
date: 2020-11-27 14:31:37
tags:
- Linux
- CentOS7
categories: 
- CentOS7
---

## passwd命令

```sh
[root@host-192-1-1-82 ~]# passwd
更改用户 root 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd: 鉴定令牌操作错误
```

## 解决

```sh
lsattr /etc/passwd  # 可选 /etc/shadow /etc/group
----i----------- /etc/shadow
```

i：文件的隐藏属性，这里需要将i去掉；

```sh
# 去掉 i 属性
chattr -i /etc/shadow
```

重新修改密码即可。
