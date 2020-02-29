---
title: Hexo 部署到 Github Pages 文件夹大小写问题
date: 2018-12-11 15:15:52
tags:
- hexo
categories: 
- hexo
---
# Hexo 部署到 Github Pages 文件夹大小写问题

解决办法:

* 1,进入到博客项目中 .deploy_git文件夹，修改 .git 下的 config 文件，将 ignorecase=true 改为 ignorecase=false

可以使用以下命令行：

```code
cd .deploy_git
vim .git/config
```

* 2,删除博客项目中 .deploy_git 文件夹下的所有文件，并 push 到 Github 上, 这一步是清空你的 github.io 项目中所有文件。

```code
git rm -rf *
git commit -m 'clean all file'
git push
```

* 3,使用 Hexo 再次生成及部署

```code
cd ..
hexo clean
hexo deploy -generate
```


参考：

[Hexo 部署到 Github Pages 文件夹大小写问题](http://1mhz.me/2015/hexo-deploy-case-sensitive/)