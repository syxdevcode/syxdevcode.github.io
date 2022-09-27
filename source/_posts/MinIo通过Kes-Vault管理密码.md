---
title: MinIo通过Kes-Vault管理密码
date: 2022-09-26 14:43:03
tags:
  - RockyLinux
  - Ubuntu
  - MinIo
  - Hashicorp Vault
  - Linux
categories:
  - MinIo
---

## 自签名证书

参考：{% post_link 'OpenSSL生成CA，二级CA，服务器证书' %}

## Kes配置文件

```yml
address: 0.0.0.0:7373 # Listen on all network interfaces on port 7373

admin:
  identity: disabled  # We disable the admin identity since we don't need it in this guide 
   
tls:
  #key: /opt/certs/agent/key/cakey.pem    # The KES server TLS private key
  #cert: /opt/certs/agent/key/cacert.crt       # The KES server TLS certificate  
  key: /opt/certs/25519/server.key    # The KES server TLS private key
  cert: /opt/certs/25519/server.crt       # The KES server TLS certificate
   
policy:
  my-app: 
    allow:
    - /v1/key/create/my-key*
    - /v1/key/generate/my-key*
    - /v1/key/decrypt/my-key*
    identities:
    - 0a3b5c174894c5b782889775a6a586c1dc8c9e03f8cf1b41be099a017ec25ec4 # Use the identity of your client.crt
   
keystore:
   vault:
     endpoint: https://127.0.0.1:8200
     version:  v1 # The K/V engine version - either "v1" or "v2".
     approle:
       id:     "a54e9ae2-a4e7-87bf-3fda-1fa30f65c3c5" # Your AppRole ID
       secret: "47ef5ebb-ebef-0ae1-001a-80dc74c8c638" # Your AppRole Secret
       retry:  15s
     status:
       ping: 10s
     tls:
       ca: /opt/certs/vault/vault.crt # Manually trust the vault certificate since we use self-signed certificates
```

-k 跳过证书校验。

```sh
kes key dek my-key-1 -k
```

参考：

[Hashicorp Vault Keystore](https://github.com/minio/kes/wiki/Hashicorp-Vault-Keystore#kes-cli-access)