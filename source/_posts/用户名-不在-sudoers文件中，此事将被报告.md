---
title: 用户名 不在 sudoers文件中，此事将被报告
date: 2018-08-02 11:22:49
tags:
- Linux
- CentOS7
categories: 
- CentOS7
---

root账户执行以下命令，或者使用`sudo`,否则提示权限不足。

```bash
vim /etc/sudoers
```

找到root用户：

```bash
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
toor    ALL=(ALL)       ALL
```

:!wq
退出保存

sudoers的权限是0440，即只有root才能读。在你用root或sudo后强行保存（wq!）即可。