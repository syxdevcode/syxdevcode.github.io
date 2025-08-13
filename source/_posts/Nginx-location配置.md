---
title: Nginx-location配置
date: 2022-09-20 18:58:12
tags:
  - Nginx
  - Ubuntu
  - RockyLinux
  - Linux
categories:
  - Nginx
---

## location 介绍

location是Nginx中的块级指令(block directive)，location指令的功能是用来匹配不同的url请求，进而对请求做不同的处理和响应

location有两种匹配规则：

* 1, 匹配URL类型，有四种参数可选，当然也可以不带参数。

```sh
location [ = | ~ | ~* | ^~ ] uri { … }
```

* 2, 命名location，用@标识，类似于定于goto语句块。

```sh
location @name { … }
```

<!--more-->
## location匹配参数解释

* （1）`=` 精确匹配

内容要同表达式完全一致才匹配成功

```sh
location = /abc/ {
  .....
}

# 只匹配http://abc.com/abc
#http://abc.com/abc [匹配成功]
#http://abc.com/abc/index [匹配失败]
```

* （2）`~` 执行正则匹配，区分大小写。

```sh
location ~ /Abc/ {
  .....
}
#http://abc.com/Abc/ [匹配成功]
#http://abc.com/abc/ [匹配失败]
```

* （3）`~*` 执行正则匹配，忽略大小写

```sh
location ~* /Abc/ {
  .....
}
# 则会忽略 uri 部分的大小写
#http://abc.com/Abc/ [匹配成功]
#http://abc.com/abc/ [匹配成功]
```

* （4）`^~` 表示普通字符串匹配上以后不再进行正则匹配。

```sh
location ^~ /index/ {
  .....
}
#以 /index/ 开头的请求，都会匹配上
#http://abc.com/index/index.page  [匹配成功]
#http://abc.com/error/error.page [匹配失败]
```

* （5）不加任何规则时，默认是大小写敏感，前缀匹配，相当于加了 `~` 与 `^~`

```sh
location /index/ {
  ......
}
#http://abc.com/index  [匹配成功]
#http://abc.com/index/index.page  [匹配成功]
#http://abc.com/test/index  [匹配失败]
#http://abc.com/Index  [匹配失败]
# 匹配到所有uri
```

* （6）`@` nginx内部跳转

```sh
location /index/ {
  error_page 404 @index_error;
}
location @index_error {
  .....
}
#以 /index/ 开头的请求，如果链接的状态为 404。则会匹配到 @index_error 这条规则上。
```

## 3、location匹配顺序

`=` > `^~` > `~ | ~*` > `最长前缀匹配` > `/`

```sh
location =    # 精准匹配
# = 匹配优先级最高。一旦匹配成功，则不再查找其他匹配项。

location ^~   # 带参前缀匹配
# ^~类型表达式。一旦匹配成功，则不再查找其他匹配项。

location ~    # 正则匹配（区分大小写）
location ~*   # 正则匹配（不区分大小写）
location /a   # 普通前缀匹配，优先级低于带参数前缀匹配。
location /    # 任何没有匹配成功的，都会匹配这里处理
```

## location URI结尾带不带 /

如果 URI 结构是 https://domain.com/ 的形式，尾部有没有 / 都不会造成重定向。因为浏览器在发起请求的时候，默认加上了 / 。虽然很多浏览器在地址栏里也不会显示 / 。这一点，可以访问百度验证一下。
如果 URI 的结构是 https://domain.com/some-dir/ 。尾部如果缺少 / 将导致重定向。因为**根据约定，URL 尾部的 / 表示目录，没有 / 表示文件**。所以访问 /some-dir/ 时，服务器会自动去该目录下找对应的默认文件。如果访问 /some-dir 的话，服务器会先去找 some-dir 文件，找不到的话会将 some-dir 当成目录，重定向到 /some-dir/ ，去该目录下找默认文件。

参考:

[干货 | 一文彻底读懂nginx中的location指令](https://zhuanlan.zhihu.com/p/137042956)