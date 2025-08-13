---
title: MongoDb-Windows平台安装记录
date: 2024-08-06 18:51:08
tags:
  - MongoDB
  - Windows
  - Windows Server
  - 安装部署
categories:
  - MongoDB
---

## 安装

![Snipaste_2024-08-07_08-39-02.png](/img1/Snipaste_2024-08-07_08-39-02.png)

![Snipaste_2024-08-07_08-39-32.png](/img1/Snipaste_2024-08-07_08-39-32.png)

![Snipaste_2024-08-07_08-39-56.png](/img1/Snipaste_2024-08-07_08-39-56.png)

## 配置

### 新建用户

执行一下命令，创建 `Admin` 用户：

```shell
use admin
db.createUser(
  {
    user: "Admin",
    pwd: passwordPrompt(), // or cleartext password
    roles: [
      { role: "userAdminAnyDatabase", db: "admin" },
      { role: "readWriteAnyDatabase", db: "admin" }
    ]
  }
)
```

在 `C:\Program Files\MongoDB\Server\7.0\bin` 目录下，找到 `mongod.cfg` 文件，添加以下内容，启用身份验证：

```shell
security:
  authorization: "enabled"
```

之后重启 MongoDB 服务。

![Snipaste_2024-08-07_08-47-43.png](/img1/Snipaste_2024-08-07_08-47-43.png)

## 测试连接

```shell
# 验证用户
db.auth("Admin", passwordPrompt())
```

![Snipaste_2024-08-07_08-52-29.png](/img1/Snipaste_2024-08-07_08-52-29.png)
