---
title: hexo添加categories和tags页面
date: 2020-09-28 10:40:24
tags:
- Hexo
categories:
- Hexo
---

默认是没有 `categories` 和 `tags` 的

```sh
hexo new page "tags"
hexo new page "categories"
```

编辑 `/tags/index.md`

添加：

```sh
type: tags
layout: tags
```

`/categories/index.md`

添加：

```sh
type: categories
layout: categories
```