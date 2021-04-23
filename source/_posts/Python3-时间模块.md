---
title: Python3-时间模块
date: 2021-04-23 15:08:18
tags:
- Python3
- Python
categories: Python3
---

## time

参考：

[time --- 时间的访问和转换](https://docs.python.org/zh-cn/3.10/library/time.html)

`epoch` 是时间开始的点，其值取决于平台。对于 Unix， `epoch` 是1970年1月1日00:00:00（UTC）。`time.gmtime(0)` 获取给定平台上的 `epoch` 。

### time.struct_time

| 索引	|   属性   |	 值    |
| :-----| :---- | :---- |
| 0	  |tm_year（年）|	如：1993|
|1	|tm_mon（月）	|1 ~ 12|
|2	|tm_mday（日）	|1 ~ 31|
|3	|tm_hour（时）	|0 ~ 23|
|4	|tm_min（分）	|0 ~ 59|
|5	|tm_sec（秒）	|0 ~ 61|
|6	|tm_wday（周）	|0 ~ 6|
|7	|tm_yday（一年内第几天）|	1 ~ 366|
|8	|tm_isdst（夏时令）|	-1(未知)、0(不生效)、1(生效)|

### 常用函数

* strftime(format[, t]) 转换一个元组或 struct_time 表示的由 gmtime() 或 localtime() 返回的时间到由 format 参数指定的字符串。如果未提供 t ，则使用由 localtime() 返回的当前时间。 format 必须是一个字符串。如果 t 中的任何字段超出允许范围，则引发 ValueError 。
* strptime(string[, format]) 根据格式解析表示时间的字符串。 返回值为一个被 gmtime() 或 localtime() 返回的 struct_time 。
* time()	以浮点数表示从 `epoch` 开始的秒数的时间值
* gmtime([secs])	将时间戳转换为UTC下的 `struct_time，可选参数` secs 表示从 `epoch` 到现在的秒数，默认为当前时间
* localtime([secs])	与 gmtime() 相似，返回当地时间下的 `struct_time`
* mktime(t)	localtime() 的反函数,返回一个浮点数，与 time() 兼容
* asctime([t])	接收一个 `struct_time` 表示的时间，返回形式为：Sun Jun 20 23:21:05 1993 的字符串
* ctime([secs])	ctime(secs) 相当于 asctime(localtime(secs))
* strftime(format[, t])	格式化日期，接收一个 struct_time 表示的时间，并返回以可读字符串表示的当地时间
* sleep(secs)	暂停执行调用线程指定的秒数
* altzone	本地 DST 时区的偏移量，以 UTC 为单位的秒数
* timezone	本地（非 DST）时区的偏移量，UTC 以西的秒数（西欧大部分地区为负，美国为正，英国为零）
* tzname	两个字符串的元组：第一个是本地非 DST 时区的名称，第二个是本地 DST 时区的名称

### strftime 函数日期格式化符号说明

|  strftime 符号	|描述|
| :-----| :---- |
|%a	|本地化的缩写星期中每日的名称|
|%A	|本地化的星期中每日的完整名称|
|%b	|本地化的月缩写名称|
|%B	|本地化的月完整名称|
|%c	|本地化的适当日期和时间表示|
|%d	|十进制数 [01,31] 表示的月中日|
|%H	|十进制数 [00,23] 表示的小时（24小时制）|
|%I	|十进制数 [01,12] 表示的小时（12小时制）|
|%j	|十进制数 [001,366] 表示的年中日|
|%m	|十进制数 [01,12] 表示的月|
|%M	|十进制数 [00,59] 表示的分钟|
|%p	|本地化的 AM 或 PM|
|%S	|十进制数 [00,61] 表示的秒|
|%U	|十进制数 [00,53] 表示的一年中的周数（星期日作为一周的第一天）|
|%w	|十进制数 [0(星期日),6] 表示的周中日|
|%W	|十进制数 [00,53] 表示的一年中的周数（星期一作为一周的第一天）|
|%x	|本地化的适当日期表示|
|%X	|本地化的适当时间表示|
|%y	|十进制数 [00,99] 表示的没有世纪的年份|
|%Y	|十进制数表示的带世纪的年份|
|%z	|时区偏移以格式 +HHMM 或 -HHMM 形式的 UTC/GMT 的正或负时差指示，其中 H 表示十进制小时数字，M 表示小数分钟数字 [-23:59, +23:59] |
|%Z	|时区名称|
|%%	|字面的 ‘%’ 字符 |

注释:

* 当与 strptime() 函数一起使用时，如果使用 %I 指令来解析小时， %p 指令只影响输出小时字段。
* 范围真的是 0 到 61 ；值 60 在表示 leap seconds 的时间戳中有效，并且由于历史原因支持值 61 。
* 当与 strptime() 函数一起使用时， %U 和 %W 仅用于指定星期几和年份的计算。

### 实例

**strftime**

```py
from time import gmtime, strftime
strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())
# Fri, 23 Apr 2021 09:38:33 +0000
```

**strptime**

```py
import time

print(time.strptime("30 Nov 00", "%d %b %y"))
# time.struct_time(tm_year=2000, tm_mon=11, tm_mday=30, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=3, tm_yday=335, tm_isdst=-1)
```

```py
import time

print(time.time())
print(time.gmtime())
print(time.localtime())
print(time.asctime(time.localtime()))
print(time.tzname)
```

## datetime

参考：

[datetime --- 基本的日期和时间类型](https://docs.python.org/zh-cn/3.10/library/datetime.html?highlight=datetime#module-datetime)

datatime 模块重新封装了 time 模块，提供了更多接口，变得更加直观和易于调用。

date 类表示一个由年、月、日组成的日期，格式为：`datetime.date(year, month, day)`。

* year 范围为：[1, 9999]
* month 范围为：[1, 12]
* day 范围为 [1, 给定年月对应的天数]。



## calendar

参考：

[calendar --- 日历相关函数](https://docs.python.org/zh-cn/3.10/library/calendar.html?highlight=calendar#module-calendar)