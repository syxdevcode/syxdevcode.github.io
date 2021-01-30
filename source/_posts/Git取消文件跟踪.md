---
title: Git取消文件跟踪
date: 2021-01-18 10:05:02
tags:
- Git
categories: 
- Git
---

```sh
git rm -r --cached .  # 不删除本地文件
git rm -r --f . 　　  # 删除本地文件
git rm --cached 1.txt  # 删除readme1.txt的跟踪，并保留在本地。
git rm --f 1.txt    # 删除readme1.txt的跟踪，并且删除本地文件。
```

已经被纳入了版本管理中，则修改 `.gitignore` 是无效的。

解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

```sh
git rm -r --cached .
git add .

# 提交暂存区到本地仓库中
git commit -m 'update .gitignore'
```
