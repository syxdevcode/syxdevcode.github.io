---
title: Nodejs离线安装-CentOS7.4
date: 2020-05-19 13:30:07
tags:
- Linux
- CentOS7
- 安装部署
- Nodejs
categories: 
- Nodejs
---

## 下载解压

离线包：[https://nodejs.org/en/download/](https://nodejs.org/en/download/)

```sh
xz -d node-v14.2.0-linux-x64.tar.xz
tar -xvf node-v14.2.0-linux-x64.tar
```

## 安装

```sh
# 移动到/usr/local
mv node-v14.2.0-linux-x64 /usr/local/nodejs-v14.2

# 添加环境变量

# 进入 /etc/profile 文件
vim /etc/profile
# 在文件末尾添加
export PATH=$PATH:'/usr/local/nodejs-v14.2/bin'

# 使环境变量生效
source /etc/profile

# 验证
node -v
npm -v
```

参考：

[CentOS 7.3 离线二进制安装 nodeJS](https://blog.csdn.net/weixin_41474364/article/details/102906970)
