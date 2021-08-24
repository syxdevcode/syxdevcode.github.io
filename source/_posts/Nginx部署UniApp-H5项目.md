---
title: Nginx部署UniApp-H5项目
date: 2021-08-24 15:02:17
tags:
- Linux
- Windows
- CentOS7
- UniApp
- 跨域
- Vue
- H5
- Nginx
- 安装部署
categories:
- Nginx
---

## Nginx配置

```conf
server {
        listen       10005;
        server_name  localhost,127.0.0.1;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   E:\code\induction\unpackage\dist\build\h5; # 路径，不支持中文
            index  index.html index.htm;
        }
		
        # 重定向 prefix：匹配的字符串
		location ^~ /prefix/ {
            rewrite  ^/prefix/(.*) /$1 break;
            proxy_pass https://api.mokahr.com;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
```

<!--more-->

## UniApp 配置

解决跨域问题：配置devServer。

```js
// vue.config.js
module.exports = {  
  devServer:{  
    proxy:{  
      '/prefix': {    //将www.exaple.com印射为/apis  
        target: 'https://api.mokahr.com',  // 接口域名  
        secure: true,  // 如果是https接口，需要配置这个参数  
        changeOrigin: true,  //是否跨域  
        pathRewrite: {  
            '^/prefix': '/'   //需要rewrite的,  
        }                
      }  
    }  
  }  
}
```

## 页面请求

```js
// prefix ：需要匹配的字符串
let url={
	ehrUrl:"/prefix/api-platform/v1/data/ehrApplications"
}

//数据请求
uni.request({
	url: url.ehrUrl,
	method: 'GET',

	data: {
		stage: 'pending_checkin',
		phone: that.formData.mailphone
	},

	header: {
		'content-type': 'application/json',
		'cache-control': 'no-cache'
	},
	success: res => {
		//console.log(res)
		if (res.statusCode == '500') {
			uni.showToast({
				icon: 'none',
				title: res.data.errorMessage,
			});
			return;
		}
		console.log(res); 
	},
	fail: (res) => {
		uni.showToast({
			icon: 'none',
			title: res,
		});
	},
	complete: () => {
		//uni.hideLoading(); // 关闭转loading操作
	}
});
```