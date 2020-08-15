---
title: MinIo-CentOS7离线安装
date: 2020-08-15 08:49:05
tags:
- Linux
- CentOS7
- MinIo
- 安装部署
categories:
- MinIo
---
## 



进入到软件包所在目录：

```sh
cd /ldjc/rj

./minio server /ldjc/minio/data --address :9010
```

后台运行

```sh
nohup ./minio server /ldjc/minio/data --address :9010
```



