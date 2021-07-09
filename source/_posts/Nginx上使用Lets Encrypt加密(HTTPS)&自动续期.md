---
title: Nginx上使用Lets Encrypt加密(HTTPS)&自动续期
date: 2018-04-25 15:36:54
tags:
- Linux
- Ubuntu
- AWS
- Nginx
- Lets Encrypt
- https
categories:
- Lets Encrypt
---
# Nginx上使用Let’s Encrypt加密(HTTPS)&自动续期

[官网](https://letsencrypt.org/ "官网")

## 获取证书生成工具 certbot

``` linux
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
```

## 获取证书

``` linux
./certbot-auto certonly -d *.域名 --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

按照提示依次填写邮箱地址，同意服务条款，绑定IP就行了。

面这一步很关键，需要你配置你的域名TXT记录，以校验域名所有权，也就是判断证书申请者是不是该域名的拥有者。

``` linux
Please deploy a DNS TXT record under the name
_acme-challenge.shiyx.top with the following value:

l1VX8xfEnhrJ02QB4o-M6ntb9JEsItRISb1YjOEw3KY

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
Press Enter to Continue
```

在添加txt记录到你的域名之前，切勿点击回车键。

登陆域名解析控制台（阿里云），添加一条TXT记录：

![添加解析记录](/img/aliyun解析.png)

执行命令：`dig -t txt _acme-challenge.shiyx.top`

``` liunx
ubuntu@ip-172-31-16-20:~$ dig -t txt _acme-challenge.shiyx.top

; <<>> DiG 9.10.3-P4-Ubuntu <<>> -t txt _acme-challenge.shiyx.top
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54437
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;_acme-challenge.shiyx.top.     IN      TXT

;; ANSWER SECTION:
_acme-challenge.shiyx.top. 60   IN      TXT     "l1VX8xfSnwrJ02QB4o-M6ntb9JEsItRISb1YjOEw3YY"

;; Query time: 2001 msec
;; SERVER: 172.31.0.2#53(172.31.0.2)
;; WHEN: Thu Apr 26 00:50:34 UTC 2018
;; MSG SIZE  rcvd: 110
```

确认生效后，按回车键继续

``` linux
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/shiyx.top/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/shiyx.top/privkey.pem
   Your cert will expire on 2018-07-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

证书申请完毕！证书和密钥保存在“/etc/letsencrypt/archive/”目录下

``` linux
ubuntu@ip-172-31-16-20:~$ sudo tree /etc/letsencrypt/archive
/etc/letsencrypt/archive
└── shiyx.top
    ├── cert1.pem
    ├── chain1.pem
    ├── fullchain1.pem
    └── privkey1.pem

1 directory, 4 files
```

## 续期HTTPS证书

官网：

[https://certbot.eff.org/docs/using.html?highlight=renew#renewing-certificates](https://certbot.eff.org/docs/using.html?highlight=renew#renewing-certificates, "参考官网")

### 续期HTTPS证书命令

``` linux-ubuntu
sudo certbot renew
```

### 自动续签HTTPS证书

Let's Encrypt 证书的有效期只有 90 天，因此我们需要定期的对他进行续签，我们使用linux自带的cron来设定计划任务

``` linux
crontab -e
```

注意，如果你是第一次运行 crontab 命令，它会问题使用哪一个编辑器，你可以根据自己的需要进行选择，我选择的是 vim-basic

``` linux
sudo crontab -e
```

添加配置：

``` cron
30 2 * * 1 sudo certbot renew
```

上面的执行时间为：每周一半夜2点30分执行renew任务。

你可以在命令行执行`sudo renew`看看是否执行正常。

## Nginx配置

官网：[http://nginx.org/en/docs/](http://nginx.org/en/docs/, "参考官网")

注：aws需要定义安全组，允许https入站规则

### Webroot配置模式

Webroot 的工作插件放置在一个特殊的文件/.well-known目录文档根目录下，它可以打开（通过Web服务器）内由让我们的加密服务进行验证。 根据配置的不同，你可能需要明确允许访问/.well-known目录。

为了确保该目录可供Let’s Encrypt进行验证，让我们快速更改我们的Nginx配置。编辑`sudo vim  /etc/nginx/sites-available/default`文件，并将下面代码添加进去:

``` nginx
location ~ /.well-known {
    allow all;
}
```

``` nginx
## 检查测试配置文件是否正确
sudo nginx -t -c /etc/nginx/nginx.conf

## 重新加载nginx
sudo service nginx reload
```

Nginx配置：

``` nginx
server {
        listen 80;
        server_name www.shiyx.top;
        return 301 https://www.shiyx.top$request_uri;
        server_tokens off;
}
server {
        listen 443 ssl http2;
        ssl on;
        server_name www.shiyx.top;
        #$host该变量的值等于请求头中Host的值。如果Host无效时，那么就是处理该请求的server的名称。
        #permanent: 永久性重定向。请求日志中的状态码为301
        #nginx 对文档检测比较严格，所以if ( $host != 'www.shiyx.top' ) 这些代码之间需要有空格隔开，不然会
        #报错：unknown directive “if($host!=”
        if ($host != 'www.shiyx.top' ){
                rewrite ^/(.*)$ https://www.shiyx.top/$1 permanent;
        }
        ssl_certificate /etc/letsencrypt/live/shiyx.top/fullchain.pem;

        ssl_certificate_key /etc/letsencrypt/live/shiyx.top/privkey.pem;

        #禁止在header中出现服务器版本，防止黑客利用版本漏洞攻击
        server_tokens off;
        # 设置ssl/tls会话缓存的类型和大小。如果设置了这个参数一般是shared，buildin可能会参数内存碎片，默认是none，
        #和off差不多，停用缓存。如shared:SSL:10m表示我>所有的nginx工作进程共享ssl会话缓存，
        #网介绍说1M可以存放约4000个sessions。
        ssl_session_cache    shared:SSL:1m;

        # 客户端可以重用会话缓存中ssl参数的过期时间，内网系统默认5分钟太短了，可以设成30m即30分钟甚至4h。
        ssl_session_timeout  5m;

        ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;

        # 选择加密套件，不同的浏览器所支持的套件（和顺序）可能会不同。
        # 这里指定的是OpenSSL库能够识别的写法，你可以通过 openssl -v cipher 'RC4:HIGH:!aNULL:!MD5'
        #（后面是你所指定的套件加密算法） 来看所支持算法。
        ssl_ciphers  HIGH:!aNULL:!MD5;

        # 设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件。
        ssl_prefer_server_ciphers  on;
        location / {
                root   html;
                index  index.html index.htm;
        }
        location ~ /.well-known{
                allow all;
        }
}

```

另附https检查网站

[hhttps://www.ssllabs.com/ssltest/analyze.html](https://www.ssllabs.com/ssltest/analyze.html, "参考1")

参考：

[https://www.lao-wang.com/?p=142](https://www.lao-wang.com/?p=142, "参考1")

[https://blog.guorenxi.com/43.html](https://blog.guorenxi.com/43.html, "参考1")

[https://segmentfault.com/a/1190000005797776](https://segmentfault.com/a/1190000005797776, "参考1")


[http://www.cnblogs.com/stulzq/p/8628163.html](http://www.cnblogs.com/stulzq/p/8628163.html, "参考2")

[https://www.appinn.com/use-letsencrypt-with-nginx/](https://www.appinn.com/use-letsencrypt-with-nginx/, "参考3")

[https://www.cnblogs.com/lzpong/p/6433189.html](https://www.cnblogs.com/lzpong/p/6433189.html, "参考4")