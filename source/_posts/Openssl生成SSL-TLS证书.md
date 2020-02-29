---
title: Openssl生成SSL/TLS证书
date: 2018-07-06 08:42:38
tags:
- Openssl
- https
- 加密
- SSL/TLS
categories:
- Openssl
---
# openssl生成SSL/TLS证书

官网：[https://www.openssl.org/](https://www.openssl.org/)

github:[https://github.com/openssl/openssl](https://github.com/openssl/openssl)

文档：[https://www.openssl.org/docs/manmaster/man1/](https://www.openssl.org/docs/manmaster/man1/)

## 证书相关概念

### CA根证书

首先要有一个CA根证书，然后用CA根证书来签发用户证书。
用户进行证书申请：一般先生成一个私钥，然后用私钥生成证书请求(证书请求里应含有公钥信息)，再利用证书服务器的CA根证书来签发证书。

** 特别说明:**

* （1）自签名证书(一般用于顶级证书、根证书): 证书的名称和认证机构的名称相同.
* （2）根证书：根证书是CA认证中心给自己颁发的证书,是信任链的起始点。任何安装CA根证书的服务器都意味着对这个CA认证中心是信任的。

数字证书则是由证书认证机构（CA）对证书申请者真实身份验证之后，用CA的根证书对申请人的一些基本信息以及申请人的公钥进行签名（相当于加盖发证书机构的公章）后形成的一个数字文件。数字证书包含证书中所标识的实体的公钥（就是说你的证书里有你的公钥），由于证书将公钥与特定的个人匹配，并且该证书的真实性由颁发机构保证（就是说可以让大家相信你的证书是真的），因此，数字证书为如何找到用户的公钥并知道它是否有效这一问题提供了解决方案。

### PKCS的15个标准

PKCS 全称是 Public-Key Cryptography Standards ，是由 RSA 实验室与其它安全系统开发商为促进公钥密码的发展而制订的一系列标准。

* ** PKCS#1 **：RSA加密标准。PKCS#1定义了RSA公钥函数的基本格式标准，特别是数字签名。它定义了数字签名如何计算，包括待签名数据和签名本身的格式；它也定义了PSA公/私钥的语法。

* ** PKCS#2 **：涉及了RSA的消息摘要加密，这已被并入PKCS#1中。

* ** PKCS#3 **：Diffie-Hellman密钥协议标准。PKCS#3描述了一种实现Diffie- Hellman密钥协议的方法。

* ** PKCS#4 **：最初是规定RSA密钥语法的，现已经被包含进PKCS#1中。

* ** PKCS#5 **：基于口令的加密标准。PKCS#5描述了使用由口令生成的密钥来加密8位位组串并产生一个加密的8位位组串的方法。PKCS#5可以用于加密私钥，以便于密钥的安全传输（这在PKCS#8中描述）。

* ** PKCS#6 **：扩展证书语法标准。PKCS#6定义了提供附加实体信息的X.509证书属性扩展的语法（当PKCS#6第一次发布时，X.509还不支持扩展。这些扩展因此被包括在X.509中）。

* ** PKCS#7 **：密码消息语法标准。PKCS#7为使用密码算法的数据规定了通用语法，比如数字签名和数字信封。PKCS#7提供了许多格式选项，包括未加密或签名的格式化消息、已封装（加密）消息、已签名消息和既经过签名又经过加密的消息。

* ** PKCS#8 **：私钥信息语法标准。PKCS#8定义了私钥信息语法和加密私钥语法，其中私钥加密使用了PKCS#5标准。

* ** PKCS#9 **：可选属性类型。PKCS#9定义了PKCS#6扩展证书、PKCS#7数字签名消息、PKCS#8私钥信息和PKCS#10证书签名请求中要用到的可选属性类型。已定义的证书属性包括E-mail地址、无格式姓名、内容类型、消息摘要、签名时间、签名副本（counter signature）、质询口令字和扩展证书属性。

* ** PKCS#10 **：证书请求语法标准。PKCS#10定义了证书请求的语法。证书请求包含了一个唯一识别名、公钥和可选的一组属性，它们一起被请求证书的实体签名（证书管理协议中的PKIX证书请求消息就是一个PKCS#10）。

* ** PKCS#11 **：密码令牌接口标准。PKCS#11或“Cryptoki”为拥有密码信息（如加密密钥和证书）和执行密码学函数的单用户设备定义了一个应用程序接口（API）。智能卡就是实现Cryptoki的典型设备。注意：Cryptoki定义了密码函数接口，但并未指明设备具体如何实现这些函数。而且Cryptoki只说明了密码接口，并未定义对设备来说可能有用的其他接口，如访问设备的文件系统接口。

* ** PKCS#12 **：个人信息交换语法标准。PKCS#12定义了个人身份信息（包括私钥、证书、各种秘密和扩展字段）的格式。PKCS#12有助于传输证书及对应的私钥，于是用户可以在不同设备间移动他们的个人身份信息。

* ** PDCS#13 **：椭圆曲线密码标准。PKCS#13标准当前正在完善之中。它包括椭圆曲线参数的生成和验证、密钥生成和验证、数字签名和公钥加密，还有密钥协定，以及参数、密钥和方案标识的ASN.1语法。

* ** PKCS#14 **：伪随机数产生标准。PKCS#14标准当前正在完善之中。为什么随机数生成也需要建立自己的标准呢？PKI中用到的许多基本的密码学函数，如密钥生成和Diffie-Hellman共享密钥协商，都需要使用随机数。然而，如果“随机数”不是随机的，而是取自一个可预测的取值集合，那么密码学函数就不再是绝对安全了，因为它的取值被限于一个缩小了的值域中。因此，安全伪随机数的生成对于PKI的安全极为关键。

* ** PKCS#15 **：密码令牌信息语法标准。PKCS#15通过定义令牌上存储的密码对象的通用格式来增进密码令牌的互操作性。在实现PKCS#15的设备上存储的数据对于使用该设备的所有应用程序来说都是一样的，尽管实际上在内部实现时可能所用的格式不同。PKCS#15的实现扮演了翻译家的角色，它在卡的内部格式与应用程序支持的数据格式间进行转换。

** 注意：** net,ios中rsa加解密使用的是pkcs1，而java使用的是pkcs8

以上参考：[PKCS 发布的15 个标准](http://falchion.iteye.com/blog/1472453)

### x509证书

#### 介绍

X.509 是密码学里公钥证书的格式标准。 X.509 证书己应用在包括TLS/SSL（WWW万维网安全浏览的基石）在内的众多 Intenet协议里.同时它也用在很多非在线应用场景里，比如电子签名服务。X.509证书里含有公钥、身份信息(比如网络主机名，组织的名称或个体名称等)和签名信息（可以是证书签发机构CA的签名，也可以是自签名）。对于一份经由可信的证书签发机构签名或者可以通过其它方式验证的证书，证书的拥有者就可以用证书及相应的私钥来创建安全的通信，对文档进行数字签名.X.509还附带了证书吊销列表和用于从最终对证书进行签名的证书签发机构直到最终可信点为止的证书合法性验证算法。

当前使用的版本是X.509 V3，它加入了扩展字段支持，这极大地增进了证书的灵活性。X.509 V3证书包括一组按预定义顺序排列的强制字段，还有可选扩展字段，即使在强制字段中，X.509证书也允许很大的灵活性，因为它为大多数字段提供了多种编码方案.

x509证书一般会用到三类文件，key，csr，crt。
Key是私用密钥，openssl格式，通常是rsa算法。
csr是证书请求文件，用于申请证书。在制作csr文件的时候，必须使用自己的私钥来签署申请，还可以设定一个密钥。
crt是CA认证后的证书文件（windows下面的csr，其实是crt），签署人用自己的key给你签署的凭证。

#### 编码格式(也用作扩展)

* ** .PEM **- PEM扩展名用于不同类型的X.509v3文件,全称Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是ASCII(BASE64)编码.

    查看PEM格式证书的信息: `openssl x509 -in certificate.pem -text -noout`

    Apache和*NIX服务器偏向于使用这种编码格式.

* ** .DER** - DER扩展用于二进制DER编码证书,全称Distinguished Encoding Rules,打开看是二进制格式,不可读,这些文件也可能带有CER或CRT扩展名.

    查看DER格式证书的信息: `openssl x509 -in certificate.der -inform der -text -noout`

    Java和Windows服务器偏向于使用这种编码格式.

例如：

密钥pem格式：

```bash
-----BEGIN RSA PRIVATE KEY-----
-----END RSA PRIVATE KEY-----
```

证书pem格式：

```bash
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
```

#### 文件扩展名

* ** .key格式 **：通常用来存放一个公钥或者私钥,默认PKCS＃1,并非X.509证书,可能是PEM,也可能是DER

* ** .csr/.p10格式 **：证书签名请求（证书请求文件），含有公钥信息，certificate signing request的缩写,PKCS#10标准定义

* ** .crt格式 **：证书文件，certificate的缩写,CRT扩展名用于证书，只有公钥没有私钥。证书可以编码为二进制DER或ASCII PEM。CER和CRT扩展几乎是同义词

* ** .cer格式 **：证书文件,常见于Windows系统，certificate缩写，只有公钥没有私钥,可能是PEM编码,也可能是DER编码,大多数应该是DER编码,crt文件和cer文件只有在使用相同编码的时候才可以安全地相互替代。

* ** .crl格式 **：证书吊销列表，Certificate Revocation List的缩写

* ** .pem格式 **：用于导出，导入证书时候的证书的格式，用于不同类型的X.509v3文件，有证书开头，结尾的格式

* ** .p7b/.p7r格式 **： 以树状展示证书链(certificate chain)，同时也支持单个证书，不含私钥,PKCS#7标准定义

* ** .p7c/.p7m/.p7s格式 **：PKCS#7证书格式，仅仅包含证书和CRL列表信息，没有私钥。

* ** .pfx/p12格式 **：由Public Key Cryptography Standards #12，PKCS#12标准定义，包含了公钥和私钥的二进制格式的证书形式，以pfx作为证书文件后缀名。PFX（也称为PKCS＃12）支持证书，私钥和证书路径中的所有证书的安全存储。PKCS＃12格式是唯一可用于导出证书及其私钥的文件格式。

## 证书生成

### openssl基本命令

公钥负责加密，私钥负责解密

私钥负责签名，公钥负责验证

#### 修改openssl.cnf配置文件

首先需要定位openssl.cnf文件：

```bash
locate openssl.cnf
```

结果：

```bash
/etc/pki/tls/openssl.cnf
/usr/share/man/man5/openssl.cnf.5ssl.gz
```

修改配置文件，修改其中的dir变量,重新设置SSL的工作目录

```bash
vim /etc/pki/tls/openssl.cnf
```

#### 生成私钥

命令：`openssl genrsa`

查看帮助：`openssl genrsa -h`

```bash
 -des           生成的私钥采用DES算法加密
 -des3          生成的私钥采用DES3算法加密 (168 bit key)
 -seed          encrypt PEM output with cbc seed
 -aes128, -aes192, -aes256
 -out file       私钥输出位置
 -passout arg    输出文件的密码，如果我们指定了对称加密算法，也可以不带此参数，会有命令行提示你输入密码
```

生成一个私钥：
默认长度512,PKCS1格式

```bash
openssl genrsa -out ca.key 2048
## 或者
openssl genrsa -des3 -passout pass:123456 -out RSA.pem
```

查看：

```bash
cat ca.key
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQE..................
-----END RSA PRIVATE KEY-----
```

#### 生成公钥

命令：`openssl rsa`

查看帮助：`openssl rsa -h`

```bash
 -inform arg     输入文件编码格式，只有pem和der两种
 -outform arg    输出文件编码格式，只有pem和der两种
 -in arg         input file  输入文件
 -sgckey         Use IIS SGC key format
 -passin arg     如果输入文件被对称加密过，需要指定输入文件的密码
 -out arg        输出文件位置
 -passout arg    如果输出文件也需要被对称加密，需要指定输出文件的密码

 -des            对输出结果采用对称加密 des算法
 -des3           对输出结果采用对称加密 des3算法
 -seed
 -aes128, -aes192, -aes256

 以上几个都是对称加密算法的指定，生成私钥的时候一般会用到，我们不让私钥明文保存

 -text           以明文形式输出各个参数值
 -noout          不输出密钥到任何文件
 -modulus        输出模数值
 -check          检查输入密钥的正确性和一致性
 -pubin          指定输入文件是公钥
 -pubout         指定输出文件是公钥
 -engine e       指定三方加密库或者硬件
```

使用刚刚生成的私钥，以生成公钥：

```bash
openssl rsa -in ca.key -pubout -out ca_pub.key
```

```bash
## 为RSA密钥增加口令保护
openssl rsa -in RSA.pem -des3 -passout pass:123456 -out E_RSA.pem

##为RSA密钥去除口令保护
openssl rsa -in E_RSA.pem -passin pass:123456 -out P_RSA.pem

## 比较
diff RSA.pem P_RSA.pem

##修改加密算法为aes128，口令是123456
openssl rsa -in RSA.pem -passin pass:123456 -aes128 -passout pass:123456 -out E_RSA.pem

## 查看密钥对中的各个参数
openssl rsa -in RSA.pem -des -passin pass:123456 -text -noout

## 提取密钥中的公钥并打印模数值
## 提取公钥，用pubout参数指定输出为公钥
openssl rsa -in RSA.pem -passin pass:123456 -pubout -out pub.pem

##打印公钥中模数值
$ openssl rsa -in pub.pem -pubin -modulus -noout

Modulus=C2B..........3ED

##把pem格式转化成der格式，使用outform指定der格式
openssl rsa -in RSA.pem -passin pass:123456 -des -passout pass:123456 -outform der -out rsa.der

##把der格式转化成pem格式，使用inform指定der格式
openssl rsa -in rsa.der -inform der -passin pass:123456 -out rsa.pem
```

#### 加解密/签名验证

命令：`openssl rsautl`

注意：无论是使用公钥加密还是私钥加密，RSA每次能够加密的数据长度不能超过RSA密钥长度，并且根据具体的补齐方式不同输入的加密数据最大长度也不一样，而输出长度则总是跟RSA密钥长度相等。

![QQ截图20180706152021.png](/img/QQ截图20180706152021.png)

```bash
openssl rsautl -h
Usage: rsautl [options]
-in file        input file                                           //输入文件
-out file       output file                                          //输出文件
-inkey file     input key                                            //输入的密钥
-keyform arg    private key format - default PEM                     //指定密钥格式
-pubin          input is an RSA public                               //指定输入的是RSA公钥
-certin         input is a certificate carrying an RSA public key    //指定输入的是证书文件
-ssl            use SSL v2 padding                                   //使用SSLv23的填充方式
-raw            use no padding                                       //不进行填充
-pkcs           use PKCS#1 v1.5 padding (default)                    //使用V1.5的填充方式
-oaep           use PKCS#1 OAEP                                      //使用OAEP的填充方式
-sign           sign with private key                                //使用私钥做签名
-verify         verify with public key                               //使用公钥认证签名
-encrypt        encrypt with public key                              //使用公钥加密
-decrypt        decrypt with private key                             //使用私钥解密
-hexdump        hex dump output                                      //以16进制dump输出
-engine e       use engine e, possibly a hardware device.            //指定三方库或者硬件设备
-passin arg    pass phrase source                                    //指定输入的密码
```

##### 加密和解密

```bash
##生成RSA密钥
openssl genrsa -des3 -passout pass:123456 -out RSA.pem

##提取公钥
openssl rsa -in RSA.pem -passin pass:123456 -pubout -out pub.pem

##使用RSA作为密钥进行加密，实际上使用其中的公钥进行加密
openssl rsautl -encrypt -in plain.txt -inkey RSA.pem -passin pass:123456 -out enc.txt

##使用RSA作为密钥进行解密，实际上使用其中的私钥进行解密
openssl rsautl -decrypt -in enc.txt -inkey RSA.pem -passin pass:123456 -out replain.txt

##比较原始文件和解密后文件
diff plain.txt replain.txt

##使用公钥进行加密
openssl rsautl -encrypt -in plain.txt -inkey pub.pem -pubin -out enc1.txt

##私钥进行解密
openssl rsautl -decrypt -in enc1.txt -inkey RSA.pem -passin pass:123456 -out replain1.txt

##比较原始文件和解密后文件
diff plain.txt replain1.txt
```

##### 签名与验证

```bash
##使用RSA密钥进行签名，实际上使用私钥进行签名
openssl rsautl -sign -in plain.txt -inkey RSA.pem -passin pass:123456 -out sign.txt

##使使用RSA密钥进行验证，实际上使用公钥进行验证
openssl rsautl -verify -in sign.txt -inkey RSA.pem -passin pass:123456 -out replain.txt

##对比原始文件和签名解密后的文件
diff plain.txt replain.txt


##提取PCKS8格式的私钥
openssl pkcs8 -topk8 -in RSA.pem -passin pass:123456 -out pri.pem -nocrypt

##使用私钥进行签名
openssl rsautl -sign -in plain.txt -inkey pri.pem  -out sign1.txt

##使用公钥进行验证
openssl rsautl -verify -in sign1.txt -inkey pub.pem -pubin -out replain1.txt

##对比原始文件和签名解密后的文件
diff plain.txt replain1.txt
```

### CA证书生成

生成CA私钥（.key）-->生成CA证书请求（.csr）-->自签名得到根证书（.crt）（CA给自已颁发的证书）。

```bash
# Generate CA private key
openssl genrsa -out ca.key 2048
# Generate CSR
openssl req -new -key ca.key -out ca.csr ## 需要手动输入回车
# Generate Self Signed certificate（CA 根证书）
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt

##或者
# Generate CA private key
openssl genrsa -out ca.key 2048

openssl req -passin pass:1111 -new -x509 -days 365 -key ca.key -out ca.crt -subj "/C=CN/ST=JS/L=ZJ/O=zx/OU=test/CN=root"
```

### 用户证书

生成私钥（.key）-->生成证书请求（.csr）-->用CA根证书签名得到证书（.crt）

#### 服务器端证书

```bash
openssl genrsa -passout pass:1111 -des3 -out server.key 2048
openssl req -passin pass:1111 -new -key server.key -out server.csr -subj "/C=CN/ST=JS/L=ZJ/O=zx/OU=test/CN=www.shiyx.top:5673"
openssl x509 -req -passin pass:1111 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
openssl rsa -passin pass:1111 -in server.key -out server.key ## 移除密码
## 生成pfx格式证书，需要输入密码，或使用 -passin pass:1111
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt -certfile ca.crt
```

#### 客户端证书

```bash
openssl genrsa -passout pass:1111 -des3 -out client.key 2048
openssl req -passin pass:1111 -new -key client.key -out client.csr -subj "/C=CN/ST=JS/L=ZJ/O=zx/OU=test/CN=www.shiyx.top:5673"
openssl x509 -passin pass:1111 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt
openssl rsa -passin pass:1111 -in client.key -out client.key ## 移除密码
```

#### PKCS＃12(.pfx .p12)证书生成

```bash
##将PEM证书文件和私钥转换为PKCS＃12(.pfx .p12)
## 需要输入密码
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt -certfile ca.crt
```

#### 生成pem格式证书

有时需要用到pem格式的证书，可以用以下方式合并证书文件（crt）和私钥文件（key）来生成

```bash
$cat client.crt client.key> client.pem
$cat server.crt server.key > server.pem
$ cat server.crt server.key|tee server.pem # 将私钥和证书合并到一个文件中
```

tee：定向输出到server.pem

#### 证书转换

可以使用以下命令：

```bash
openssl x509 ## 必须为证书文件， .key文件非证书文件

openssl rsa  ## 必须为密钥文件。
```

依赖命令中参数：

```bash
-inform pem|der : 输入文件格式
-outform der|pem: 输出文件格式
```

例如：

```bash
## crt转换为pem
openssl x509 -in server.crt -out server.pem ## -outform der :从pem格式转成der格式

## pem转换为crt
openssl x509 -in server.pem -out server1.crt ## -outform der :从der格式转成pem格式

## 比较
diff server.crt server1.crt

## der格式转成pem格式
openssl x509 -in ca.crt -outform der -out ca.der
openssl rsa -in ca.key -outform der -out ca1.der

## pem格式转成crt格式 需要指定 -inform der
openssl x509 -in ca.der -inform der -outform pem -out ca.pem
diff ca.pem ca.crt

##将PEM证书文件和私钥转换为PKCS＃12(.pfx .p12)
## 需要输入密码
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt -certfile ca.crt

##将包含私钥和证书的PKCS＃12文件(.pfx .p12)转换为PEM
## 需要输入密码
openssl pkcs12 -in server.pfx -out server1.pem -nodes
diff server.pem server.crt

openssl pkcs12 -in server.pfx -out server.cer -nodes
```

#### pkcs1与pkcs8格式RSA私钥互相转换

```bash
openssl pkcs8 -topk8 -inform PEM -in server.key -outform pem -nocrypt -out pkcs8.pem
## openssl pkcs8 -in pkcs8.pem -nocrypt -out server2.key ## 失效
openssl rsa -in pkcs8.pem -out pkcs1.pem
diff server.key pkcs1.pem
```

参考：

[openssl生成SSL证书的流程](https://blog.csdn.net/liuchunming033/article/details/48470575)

[X.509](https://zh.wikipedia.org/wiki/X.509)

[那些证书相关的玩意儿(SSL,X.509,PEM,DER,CRT,CER,KEY,CSR,P12等)](https://www.cnblogs.com/guogangj/p/4118605.html)

[OpenSSL加解密使用](https://www.jianshu.com/p/15b1d935a44b)

[openssl ca(签署和自建CA)](http://www.cnblogs.com/f-ck-need-u/p/7115871.html)

[pkcs1与pkcs8格式RSA私钥互相转换](https://www.jianshu.com/p/08e41304edab)

[http://www.cnblogs.com/yaozhenfa/p/gRpc_with_ssl.html](http://www.cnblogs.com/yaozhenfa/p/gRpc_with_ssl.html)

[https://vimsky.com/article/3608.html](https://vimsky.com/article/3608.html)

[https://stackoverflow.com/questions/13732826/convert-pem-to-crt-and-key?answertab=votes](https://stackoverflow.com/questions/13732826/convert-pem-to-crt-and-key?answertab=votes)
