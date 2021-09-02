---
title: Hexo显示文章字数与时长
date: 2021-09-02 18:07:25
tags:
- Hexo
categories:
- Hexo
---

安装npm 包：

```sh
npm install hexo-symbols-count-time
```

在博客根目录修改：`_config.yml`：

```yml
symbols_count_time:
  symbols: true
  time: true
  total_symbols: true
  total_time: true
  exclude_codeblock: false
  awl: 4
  wpm: 275
  suffix: "mins."
```

在 Next 配置文件中 `\source\_data\next.yml` 或 `config`：
（默认会添加，切记不能重复添加）

```yml
# Post wordcount display settings
# Dependencies: https://github.com/theme-next/hexo-symbols-count-time
symbols_count_time:
  separated_meta: true
  item_text_post: true
  item_text_total: false
```

重新启动：

```sh
npx hexo s
```

参考：

[Hexo博客NexT主题下添加字数统计和阅读时长](https://blog.csdn.net/mqdxiaoxiao/article/details/93670772)

[https://github.com/theme-next/hexo-symbols-count-time](https://github.com/theme-next/hexo-symbols-count-time)