---
title: >-
  AttributeError: /lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol:
  EVP_CIPHER_CTX_cleanup
date: 2020-11-05 14:10:55
tags:
- Linux
- AWS
- Ubuntu
- Shadowsocks
- 安装部署
categories:
- Ubuntu
---

## 原因

由于在 openssl1.1.0 版本中，废弃了 EVP_CIPHER_CTX_cleanup 函数,要用这个函数 EVP_CIPHER_CTX_reset() 代替。

## 解决

使用 vim 编辑 /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py

```sh
vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
```

搜索 EVP_CIPHER_CTX_cleanup ，总共两处：52行，111行，将 cleanup 改为 reset 。

```py
libcrypto.EVP_CIPHER_CTX_cleanup
# 改为
libcrypto.EVP_CIPHER_CTX_reset
```

![微信截图_20201105141624.png](/img/微信截图_20201105141624.png)

参考：

[AWS-AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_](https://www.jianshu.com/p/3f874d5aac54)
