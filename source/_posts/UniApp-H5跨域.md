---
title: UniApp-H5跨域
date: 2021-08-23 09:49:18
tags:
- UniApp
- H5
- Vue
- IIS
categories:
- UniApp
---

`uni-app` 中 `manifest.json->h5->devServer`配置：

![微信截图_20210823095017.png](/img/微信截图_20210823095017.png)

代码：

```js
"h5": {
		"devServer": {
			"port": 8080, // 端口
			"disableHostCheck": true,
			"proxy": {
				"/apis": {
					"target": "https://api.mokahr.com",
					"changeOrigin": true, //是否跨域
					"secure": false // 设置支持https
					,"pathRewrite": {
						"^/apis": ""
					}
				}
			},
			"https": true
		}
	}
```
<!--more-->

请求页面：

![微信截图_20210823095129.png](/img/微信截图_20210823095129.png)

代码：

```js
uni.request({ //数据请求
	url: '/apis/api-platform/v1/data/ehrApplications',
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
		if (res.statusCode == '500') {
			uni.showToast({
				icon: 'none',
				title: res.data.errorMessage,
			});
			//this.resetForm();
			return;
		} 
	},
	fail: (res) => {
		uni.showToast({
			icon: 'none',
			title: res,
		});
	},
	complete: () => {
		//uni.hideLoading(); // 关闭loading
	}
});
```

