---
title: Nignx部署Vue项目刷新报404错误解决
date: 2021-09-08 14:11:37
tags:
- Windows
- CentOS7
- Vue
- H5
- Nginx
- 安装部署
categories:
- Nginx
---

```sh
location / {
    root   D:\项目部署\web;
    index  index.html index.htm;
    ## 添加以下代码
    try_files $uri $uri/ /index.html;
}
```
