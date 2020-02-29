---
title: LRU缓存淘汰算法
date: 2019-07-27 13:37:59
tags:
---
# LRU缓存淘汰算法

## 简介

LRU （英文：Least Recently Used）, 意为最近最少使用，这个算法的精髓在于如果一块数据最近被访问，那么它将来被访问的几率也很高，根据数据的历史访问来淘汰长时间未使用的数据。

![v2-71b21233c615b1ce899cd4bd3122cbab_hd.jpg](/img/v2-71b21233c615b1ce899cd4bd3122cbab_hd.jpg)

参考：

[缓存淘汰算法--LRU算法](https://zhuanlan.zhihu.com/p/34989978)

[Default memory cache with LRU policy](https://stackoverflow.com/questions/9653696/default-memory-cache-with-lru-policy)

[MemoryCache does not obey memory limits in configuration](https://stackoverflow.com/questions/6895956/memorycache-does-not-obey-memory-limits-in-configuration)