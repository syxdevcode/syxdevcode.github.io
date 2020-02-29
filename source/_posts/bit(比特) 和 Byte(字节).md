---
title: bit(比特) 和 Byte(字节)
date: 2019-11-16 10:57:11
tags:
- 计算机基础
categories: 
- 计算机基础
---
## bit（比特）

bit 是信息量的单位，是表示信息的最小单位，只有两种状态：0和1。二进制的一位，就叫做 1 bit。
它的简写为小写字母 “b”。

## Byte (字节)

表示一个 8 位无符号整数。

即，8 bit（比特位）= 1 Byte（字节）；

## bit（比特）与 Byte (字节) 的关系

```cs
1MB = 1024KB = 1024 * 1024B = 1048576B；
1024Byte = 1KB；
1024KB = 1MB;
1024MB = 1GB;
1024GB = 1TB;
```

![QQ截图20191116112021.png](/img/QQ截图20191116112021.png)

例如：

根据 一字节 等于 8 比特的 换算方法，就可以得出以下结论。

下载速度从理论上来说，应该是 带宽的 八分之一。

2M 宽带理论下载速度是 256 KB

10M 宽带理论下载速度是 1280 KB

总结：

存储单位和网速的单位，不管是 B 还是 b，代表的都是 字节 Byte。
带宽的单位，不管是 B 还是 b，代表的都是 比特 bit 。

参考：

[bit ( 比特 )和 Byte（字节）的关系？](https://zhuanlan.zhihu.com/p/46040087)

[Byte 结构](https://docs.microsoft.com/zh-cn/dotnet/api/system.byte?view=netframework-4.8)