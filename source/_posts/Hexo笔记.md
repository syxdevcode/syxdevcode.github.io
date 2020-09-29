---
title: Hexo笔记
date: 2017-04-28 09:47:46
tags: Hexo
categories: Hexo
---
# Hexo 笔迹

## hexo文档

* 官网：[https://hexo.io/](https://hexo.io/ "hexo官网")
* 参考教程1：[http://www.cnblogs.com/zhcncn/p/4097881.html](http://www.cnblogs.com/zhcncn/p/4097881.html "参考教程1")
* 主题：[https://github.com/hexojs/hexo/wiki/Themes](https://github.com/hexojs/hexo/wiki/Themes "主题")
* 更多主题：[http://theme-next.iissnan.com/theme-settings.html](http://theme-next.iissnan.com/theme-settings.html "更多主题")

### 命令

* hexo：查看帮助
* hexo server ：本地启动命令，可以在本地浏览器浏览
  * Ctrl+C 停止Server
* hexo new "My New Post" : 新建文档
* hexo clean ：清除旧文档
* hexo generate ：生成
* hexo deploy ：部署

### 部署配置

#### 生成 SSH KEY

``` git
git config --global user.name " GitHub 用户名 "
git config --global user.email " GitHub 邮箱 "
ssh-keygen -t rsa -C " 邮箱地址 "
```

1，SSH KEY 生成之后会默认保存在 C:/Users/电脑名用户名/.ssh 目录中，打开这个目录，打开 id_rsa.pub 文件，复制全部内容，即复制密钥

2，打开 GitHub ，依次点击 头像-->Settings-->SSH and GPG keys-->Add SSH key，将复制的密钥粘贴到 key 输入框，最后点击 Add Key ，SSH KEY 配置成功

### 添加图片

* 使用本地路径：在hexo/source目录下新建一个img文件夹，将图片放入该文件夹下，插入图片时链接即为/img/图片名称
  * 例如：`![配置](/img/mysql-workbench.png)`