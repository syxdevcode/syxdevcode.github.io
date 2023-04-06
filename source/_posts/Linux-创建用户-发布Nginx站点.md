---
title: Linux-创建用户-发布Nginx站点
date: 2022-11-16 15:56:43
tags:
  - RockyLinux
  - MinIo
  - Linux
  - Nginx
categories:
  - Nginx
---

```sh
# 新建用户
useradd lims

# 给用户设置密码
passwd lims

# 设置相应的文件夹的用户名和用户组
chown -R lims:lims /opt/nginx/www

# 给文件夹赋权
chmod -R 775 /opt/nginx/www

# 查看目录的所属者和操作权限
ls -la /opt/nginx/www

# 到 sudoers 文件中增加新的用户，否则无法使用 sudo 命令
vim /etc/sudoers ====> 增加一行与 ROOT 相同的配置，然后输入 wq! 强制强制退出
```
