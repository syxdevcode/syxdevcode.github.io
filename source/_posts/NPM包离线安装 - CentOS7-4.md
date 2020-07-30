---
title: NPM包离线安装 - CentOS7-4
date: 2020-05-19 15:08:17
tags:
- Linux
- CentOS7
- 安装部署
- Nodejs
- NPM
categories: 
- Nodejs
---

## 联网机器安装NPM包

以 `http-server` 为例：

```sh
# 首先查看 node npm 版本
C:\Users\DELL>npm -v
6.13.4

C:\Users\DELL>node -v
v12.16.1

# 安装 http-server
npm install http-server -g
```

```sh
# 查看配置，找到安装目录
npm config list

C:\Users\DELL>npm config list
; cli configs
metrics-registry = "https://registry.npmjs.org/"
scope = ""
user-agent = "npm/6.13.4 node/v12.16.1 win32 x64"

; builtin config undefined
prefix = "C:\\Users\\DELL\\AppData\\Roaming\\npm"

; node bin location = C:\Program Files\nodejs\node.exe
; cwd = C:\Users\DELL
; HOME = C:\Users\DELL
; "npm config ls -l" to show all defaults.
```

查看 `C:\\Users\\DELL\\AppData\\Roaming\\npm` 目录，截图如下：

![npm-path.png](/img/npm-path.png)
![npm-http-server](/img/npm-http-server.png)

## 通过ftp工具拷贝到未联网服务器

```sh
# 本地文件夹：C:\\Users\\DELL\\AppData\\Roaming\\npm 下的文件，
# 拷贝到 服务器上 /root/.npm 下 (不存在，使用mkdir命令创建)
http-server
http-server.cmd
```

如图：

![微信截图_20200520102421.png](/img/微信截图_20200520102421.png)


```sh
# 本地文件夹：C:\Users\DELL\AppData\Roaming\npm\node_modules 下的文件夹，
# 拷贝到 服务器上 /root/.npm/node_modules 下
http-server
```

如图：

![微信截图_20200520102513.png](/img/微信截图_20200520102513.png)


## 安装启动

```sh
# 安装
npm install /root/.npm/node_modules/http-server -g

# 启动
http-server /test/webapp -p 8013

# 查看监听的端口
netstat -lnpt

# curl 命令验证
curl  -o /dev/null -s -w %{http_code} -X GET "192.125.30.82:8013/pages/tsjb/lzsp.html" -H "accept: text/html"
curl -X GET "192.125.30.82:8013/pages/tsjb/lzsp.html" -H "accept: text/html"
```

参考：

[https://stackoverflow.com/questions/43064107/how-to-install-npm-package-while-offline/58744517#58744517](https://stackoverflow.com/questions/43064107/how-to-install-npm-package-while-offline/58744517#58744517)

[npm install](https://docs.npmjs.com/cli/install)

[npm 模块安装机制简介](http://www.ruanyifeng.com/blog/2016/01/npm-install.html)

[NPM离线包](https://www.zybuluo.com/lxjwlt/note/297879)

[npmbox](https://github.com/arei/npmbox)