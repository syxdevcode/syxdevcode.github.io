---
title: Nvm管理Nodejs版本
date: 2021-09-01 15:42:25
tags:
---

## 安装配置

nvm下载地址：[https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)

在安装目录下配置：`C:\Users\Administrator\AppData\Roaming\nvm`

输入以下命令设置nodejs路径(相当于`setting.txt`中的`path`:)：

```ps1
nvm node_mirror https://npm.taobao.org/mirrors/node/
nvm npm_mirror https://npm.taobao.org/mirrors/npm/
```

`settings.txt` 文件内容：

```txt
root: C:\Users\Administrator\AppData\Roaming\nvm
arch: 64
proxy: none
originalpath: .
originalversion: 
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

## 命令

```sh
# 查看已经安装的版本
nvm list

# 查看已经安装的版本
nvm list installed

# 查看网络可以安装的版本
nvm list available

# 查看当前系统的位数和当前nodejs的位数
nvm arch 

# 安装制定版本的node 并且可以指定平台 version 版本号 arch 平台
nvm install [arch]

nvm on  # 打开nodejs版本控制
nvm off # 关闭nodejs版本控制
nvm proxy [url]  # 查看和设置代理

# 设置或者查看setting.txt中的node_mirror，
# 如果不设置的默认是 https://nodejs.org/dist/
nvm node_mirror [url]

#  设置或者查看setting.txt中的npm_mirror,
# 如果不设置的话默认的是：https://github.com/npm/npm/archive/.
nvm npm_mirror [url]

nvm uninstall  # 卸载制定的版本
nvm use [version] [arch]   # 切换制定的node版本和位数
nvm root [path] # 设置和查看root路径
nvm version    # 查看当前的版本
```

参考：

[nvm的下载，安装与使用](https://blog.csdn.net/qq_44401643/article/details/90400626)