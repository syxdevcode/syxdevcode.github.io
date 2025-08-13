---
title: NPM常用命令
date: 2020-05-19 15:12:08
tags:
- Linux
- CentOS7
- 安装部署
- Nodejs
- NPM
categories: 
- Nodejs
---

## NPM常用命令

```sh
# 查看版本
npm -v

# 查看安装的包
npm list
npm -global list

# 查看某个模块的全部信息
npm info name

# 查看单个信息
npm info name version
npm info name homepage

# install支持多种手段，包名，git路径，包括本地路径
# 注：离线安装存在包依赖，可能会出错
sudo npm install -global [package name]
npm install git://github.com/package/path.git
npm install git://github.com/package/path.git#0.1.0
npm install package_name@version
npm install path/to/somedir  //本地路径

# 清除缓存
npm cache clean

# 查看配置项
npm config list
npm config list -l

```

参考

[npm用法及离线安装方法](https://www.cnblogs.com/laozhbook/p/npm_help.html)