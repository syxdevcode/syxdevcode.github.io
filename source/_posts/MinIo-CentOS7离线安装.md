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
export MINIO_ACCESS_KEY=ldjcadmin
export MINIO_SECRET_KEY=Perfect1
nohup ./minio server /ldjc/minio/data --address 192.125.30.71:9010
nohup ./minio server /ldjc/minio/data --address 0.0.0.0:9010
```


## TLS 

openssl genrsa -out private.key 2048
openssl genrsa -aes256 -out private.key 2048 -passout pass:123456
export MINIO_CERT_PASSWD=<Perfect1>
openssl rsa -in private-pkcs8-key.key -aes256 -passout pass:Perfect1 -out private.key






[Minio 文件服务（1）—— Minio部署使用及存储机制分析](https://www.jianshu.com/p/3e81b87d5b0b)