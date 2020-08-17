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

## 下载

下载离线包：

[https://dl.min.io/server/minio/release/linux-amd64/minio](https://dl.min.io/server/minio/release/linux-amd64/minio)

或者使用命令下载（需要接入互联网）：

```sh
wget https://dl.min.io/server/minio/release/linux-amd64/minio
```

添加执行权限：

```sh
chmod +x minio
```

## 安装

进入到软件包所在目录：

```sh
cd /ldjc/rj

export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=test
./minio server /ldjc/minio/data --address :9010
```

后台运行

```sh
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=test
nohup ./minio server /ldjc/minio/data --address 192.125.30.71:9010
nohup ./minio server /ldjc/minio/data --address :9010
```

```sh
#查看端口：
netstat -tunpl | grep 9010

#查看进程
ps aux | grep minio
```

## TLS 

详细信息请参考官网：[How to secure access to MinIO server with TLS](https://docs.min.io/docs/how-to-secure-access-to-minio-server-with-tls.html)

### 生成域名证书

```sh
openssl genrsa -out private-pkcs8-key.key 2048

# 或者使用下面命令给证书加上密码
openssl genrsa -aes256 -passout pass:123456 -out private-pkcs8-key.key 2048

# 如果设置密码 启动minio的时候，需要设置证书密码
export MINIO_CERT_PASSWD=<123456>

# 私有加密密钥的默认 OpenSSL 格式为 PKCS-8，但 MinIO 仅支持 PKCS-1
# pkcs8 转 pkcs1
openssl rsa -in private-pkcs8-key.key -aes256 -passout pass:123456 -out private.key

# 生成自签名证书
# 注：domain.com，需要替换成自己的域名
openssl req -new -x509 -days 3650 -key private.key -out public.crt -subj "/C=US/ST=state/L=location/O=organization/CN=<domain.com>"

# 或者使用下面的命令生成自签名通配符证书，该证书该所有子域名都有效。
openssl req -new -x509 -days 3650 -key private.key -out public.crt -subj "/C=US/ST=state/L=location/O=organization/CN=<*.domain.com>"
```

### 生成IP证书

生成配置文件 openssl.conf

```sh
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = VA
L = Somewhere
O = MyOrg
OU = MyOU
CN = MyServerName

[v3_req]
subjectAltName = @alt_names

[alt_names]
IP.1 = 127.0.0.1
```

使用配置文件生成证书：

```sh
openssl req -x509 -nodes -days 730 -newkey rsa:2048 -keyout private.key -out public.crt -config openssl.conf
```

参考：

[Minio 文件服务（1）—— Minio部署使用及存储机制分析](https://www.jianshu.com/p/3e81b87d5b0b)

[MinIO Server Config Guide ](https://docs.min.io/docs/minio-server-configuration-guide.html)