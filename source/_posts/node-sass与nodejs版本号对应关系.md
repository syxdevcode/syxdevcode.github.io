---
title: node-sass与nodejs版本号对应关系
date: 2021-09-01 15:18:51
tags:
- NPM
- Nodejs
- NVM
categories:
- NPM
---

对应关系：

[https://github.com/sass/node-sass](https://github.com/sass/node-sass)

NodeJS  | Supported node-sass version | Node Module
--------|-----------------------------|------------
Node 16 | 6.0+                        | 93
Node 15 | 5.0+                        | 88
Node 14 | 4.14+                       | 83
Node 13 | 4.13+, <5.0                 | 79
Node 12 | 4.12+                       | 72
Node 11 | 4.10+, <5.0                 | 67
Node 10 | 4.9+, <6.0                  | 64
Node 8  | 4.5.3+, <5.0                | 57
Node <8 | <5.0                        | <57

```ps1
# 卸载
npm uninstall node-sass

# 安装
npm install node-sass@4.12 --save-dev
```