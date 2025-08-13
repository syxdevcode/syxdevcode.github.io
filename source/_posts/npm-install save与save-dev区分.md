---
title: npm-install save与save-dev区分
date: 2021-09-01 15:26:36
tags:
- NPM
- Nodejs
- NVM
categories:
- NPM
---

```sh
# 安装模块到项目目录下，将依赖写入到dependencies中
npm install moduleName

# -save 将模块安装到项目目录下，并在package文件的dependencies节点写入依赖。
npm install -save moduleName

# -g 将模块安装到全局，具体安装到磁盘哪个位置，要看 npm config prefix 的位置。
npm install -g moduleName
 
# -save-dev 将模块安装到项目目录下，并在package文件的devDependencies节点写入依赖。
# -dev 主要是安装 开发用的库
npm install -save-dev moduleName
```

参考：

[NPM install -save 和 -save-dev 傻傻分不清](https://www.limitcode.com/detail/59a15b1a69e95702e0780249.html)
