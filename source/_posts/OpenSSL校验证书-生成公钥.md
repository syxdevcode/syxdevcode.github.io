---
title: OpenSSL校验证书+生成公钥
date: 2024-08-30 17:34:56
tags:
- OpenSSL
- https
- 加密
- SSL/TLS
categories:
- OpenSSL
---

## RSA

```sh
# 查看md5签名
openssl x509 -in 1.crt -modulus -noout | openssl md5
openssl rsa -in 1.key -modulus -noout | openssl md5

# 生成公钥
openssl x509 -pubkey -in 1.crt -noout > pub1.crt
openssl rsa -in 1.key -pubout -out pub1.key

# 比较公钥
diff pub1.crt pub1.key
```

## ECDSA

```sh
openssl x509 -noout -modulus -in test.com-cert.pem | openssl md5
openssl ec -noout -modulus -in test.com-key.pem | openssl md5

# 生成公钥
openssl pkey -in hdvr-key.key -pubout -out pubkey1.pem |
openssl x509 -pubkey -noout -in hdvr-cert.pem > cert_pubkey1.pem

# 比较公钥
diff cert_pubkey1.pem pubkey1.pem
```
