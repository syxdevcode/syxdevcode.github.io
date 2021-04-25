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

`datetime` 对象是包含来自 `date` 对象和 `time` 对象的所有信息的单一对象。

`date` 类表示一个由年、月、日组成的日期，格式为：`datetime.date(year, month, day)`。

* year 范围为：[1, 9999]
* month 范围为：[1, 12]
* day 范围为 [1, 给定年月对应的天数]。

### date 对象

**类方法**

* **date.today()** 返回当前的本地日期。等价于 date.fromtimestamp(time.time())。
* **date.fromtimestamp(timestamp)** 返回对应于 POSIX 时间戳的当地时间，例如 time.time() 返回的就是时间戳。
* 

```py
import datetime
import time

print(datetime.date.today())
print(datetime.date.fromtimestamp(time.time()))
print(datetime.date.min)
print(datetime.date.max)
```

**实例方法**

* **date.replace(year=self.year, month=self.month, day=self.day)** 返回一个具有同样值的日期，除非通过任何关键字参数给出了某些形参的新值。
* **date.timetuple()** 返回一个 time.struct_time，即 time.localtime() 所返回的类型。
* **date.weekday()** 返回一个整数代表星期几，星期一为 0，星期天为 6
* **isoweekday()**	返回一个整数代表星期几，星期一为 1，星期天为 7
* **isocalendar()**	返回格式为 (year，month，day) 的元组
* **isoformat()**	返回格式如 YYYY-MM-DD 的字符串
* **date.ctime()** 返回一个表示日期的字符串:
* **strftime(format)**	返回自定义格式的字符串

实例1：

```py
>>> import time
>>> from datetime import date
>>> today = date.today()
>>> today
datetime.date(2007, 12, 5)
>>> today == date.fromtimestamp(time.time())
True
>>> my_birthday = date(today.year, 6, 24)
>>> if my_birthday < today:
...     my_birthday = my_birthday.replace(year=today.year + 1)
>>> my_birthday
datetime.date(2008, 6, 24)
>>> time_to_birthday = abs(my_birthday - today)
>>> time_to_birthday.days
202
```

实例2：

```py
import datetime

td = datetime.date.today()
print(td.replace(year=1993, month=1, day=24))
print(td.timetuple())
print(td.weekday())
print(td.isoweekday())
print(td.isocalendar())
print(td.isoformat())
print(td.strftime('%Y %m %d %H:%M:%S %f'))
print(td.year)
print(td.month)
print(td.day)
```

### time 对象

一个 time 对象代表某日的（本地）时间，它独立于任何特定日期，并可通过 tzinfo 对象来调整。

class datetime.time(hour=0, minute=0, second=0, microsecond=0, tzinfo=None, *, fold=0)

所有参数都是可选的。 tzinfo 可以是 None，或者是一个 tzinfo 子类的实例。 其余的参数必须是在下面范围内的整数：

0 <= hour < 24,
0 <= minute < 60,
0 <= second < 60,
0 <= microsecond < 1000000,
fold in [0, 1].

实例方法:

* **isoformat()**	返回 HH:MM:SS 格式的字符串
* **replace(hour, minute, second, microsecond, tzinfo, * fold=0)**	创建一个新的时间对象，用参数指定的时、分、秒、微秒代替原有对象中的属性
* **strftime(format)**	返回自定义格式的字符串 

实例

```py
import datetime

t = datetime.time(10, 10, 10)
print(t.isoformat())
print(t.replace(hour=9, minute=9))
print(t.strftime('%I:%M:%S %p'))
print(t.hour)
print(t.minute)
print(t.second)
print(t.microsecond)
print(t.tzinfo)
```

### datetime 类

datetime 包括了 date 与 time 的所有信息，格式为：datetime(year, month, day, hour=0, minute=0, second=0, microsecond=0, tzinfo=None, *, fold=0)，参数范围值参考 date 类与 time 类。

类方法：

* **today()**	返回当地的当前时间
* **now(tz=None)**	类似于 today()，可选参数 tz 可指定时区
* **utcnow()**	返回当前 UTC 时间
* **fromtimestamp(timestamp, tz=None)**	根据时间戳返回对应时间
* **utcfromtimestamp(timestamp)**	根据时间戳返回对应 UTC 时间
* **combine(date, time)**	根据 date 和 time 返回对应时间

实例：

```py
import datetime
import time

print(datetime.datetime.today())
print(datetime.datetime.now())
print(datetime.datetime.utcnow())
print(datetime.datetime.fromtimestamp(time.time()))
print(datetime.datetime.utcfromtimestamp(time.time()))
print(datetime.datetime.combine(datetime.date(
    2019, 12, 1), datetime.time(10, 10, 10)))
print(datetime.datetime.min)
print(datetime.datetime.max)
```

实例方法：

* **date()**	返回具有同样 year,month,day 值的 date 对象
* **time()**	返回具有同样 hour, minute, second, microsecond 和 fold 值的 time 对象
replace(year, month, day=self.day, hour, minute, second, microsecond, tzinfo, * fold=0)	生成一个新的日期对象，用参数指定的年，月，日，时，分，秒…代替原有对象中的属性
* **weekday()**	返回一个整数代表星期几，星期一为 0，星期天为 6
* **isoweekday()**	返回一个整数代表星期几，星期一为 1，星期天为 7
* **isocalendar()**	返回格式为 (year，month，day) 的元组
* **isoformat()**	返回一个以 ISO 8601 格式表示日期和时间的字符串 YYYY-MM-DDTHH:MM:SS.ffffff
* **strftime(format)**	返回自定义格式的字符串

```py
import datetime

td = datetime.datetime.today()
print(td.date())
print(td.time())
print(td.replace(day=11, second=10))
print(td.weekday())
print(td.isoweekday())
print(td.isocalendar())
print(td.isoformat())
print(td.strftime('%Y-%m-%d %H:%M:%S .%f'))
print(td.year)
print(td.month)
print(td.month)
print(td.hour)
print(td.minute)
print(td.second)
print(td.microsecond)
print(td.tzinfo)
```

## calendar 模块

参考：

[calendar --- 日历相关函数](https://docs.python.org/zh-cn/3.10/library/calendar.html?highlight=calendar#module-calendar)

### calendar类

常用函数

* **setfirstweekday(weekday)**	设置每一周的开始(0 表示星期一，6 表示星期天)
* **firstweekday()**	返回当前设置的每星期的第一天的数值
* **isleap(year)**	如果 year 是闰年则返回 True ,否则返回 False
* **leapdays(y1, y2)**	返回 y1 至 y2 （包含 y1 和 y2 ）之间的闰年的数量
* **weekday(year, month, day)**	返回指定日期的星期值
* **monthrange(year, month)**	返回指定年份的指定月份第一天是星期几和这个月的天数
* **month(theyear, themonth, w=0, l=0)**	返回月份日历
* **prcal(year, w=0, l=0, c=6, m=3)**	返回年份日历
* **iterweekdays()**	返回一个迭代器，迭代器的内容为一星期的数字
* **itermonthdates(year, month)**	返回一个迭代器，迭代器的内容为年、月的日期
* **itermonthdays(year, month)** 返回一个迭代器，迭代器的内容与 itermonthdates() 类似，为 year 年 month 月的日期，但不受 datetime.date 范围限制。

实例：

```py
import calendar

calendar.setfirstweekday(1)
print(calendar.firstweekday())
print(calendar.isleap(2021))
print(calendar.leapdays(1993, 2021))
print(calendar.weekday(2021, 11, 11))
print(calendar.monthrange(2021, 5))
print(calendar.month(2021, 5))
print(calendar.prcal(2021))
```

实例2：

```py
from calendar import Calendar

c = Calendar()
print(list(c.iterweekdays()))
for i in c.itermonthdates(2021, 4):
    print(i)
```

### TextCalendar 类

TextCalendar 为 Calendar子类，用来生成纯文本日历。

实例方法：
* **formatmonth(theyear, themonth, w=0, l=0)**	返回一个多行字符串来表示指定年、月的日历
* **formatyear(theyear, w=2, l=1, c=6, m=3)**	返回一个 m 列日历，可选参数 w, l, 和 c 分别表示日期列数， 周的行数，和月之间的间隔
* **prmonth(theyear, themonth, w=0, l=0)** 与 formatmonth() 方法一样，返回一个月的日历。
* **pryear(theyear, w=2, l=1, c=6, m=3)** 与 formatyear() 方法一样，返回一整年的日历。

```py
from calendar import TextCalendar

tc = TextCalendar()
print(tc.formatmonth(2021, 5))
print(tc.formatyear(2021))
```

### HTMLCalendar类

`HTMLCalendar` 类可以生成 HTML 日历。

实例方法：

* **formatmonth(theyear, themonth, withyear=True)**	返回一个 HTML 表格作为指定年、月的日历
* **formatyear(theyear, width=3)**	返回一个 HTML 表格作为指定年份的日历
* **formatyearpage(theyear, width=3, css=‘calendar.css’, encoding=None)**	返回一个完整的 HTML 页面作为指定年份的日历 

```py
from calendar import HTMLCalendar

hc = HTMLCalendar()
print(hc.formatmonth(2021, 5))
print(hc.formatyear(2021))
print(hc.formatyearpage(2021))
```