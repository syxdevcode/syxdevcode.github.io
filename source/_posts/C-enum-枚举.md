---
title: C-enum(枚举)
date: 2021-05-27 09:42:44
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

## 语法格式

```c
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
```

第一个枚举成员的默认值为整型的 0，后续枚举成员的值在前一个成员上加 1。

以在定义枚举类型时改变枚举元素的值：

```c
enum season {spring, summer=3, autumn, winter};
```

没有指定值的枚举元素，其值为前一元素加 1。也就说 spring 的值为 0，summer 的值为 3，autumn 的值为 4，winter 的值为 5

<!--more-->
## 枚举变量的定义

在C 语言中，枚举类型是被当做 `int` 或者 `unsigned int` 类型来处理的

**1、先定义枚举类型，再定义枚举变量**

```c
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
enum DAY day;
```

**2、定义枚举类型的同时定义枚举变量**

```
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
```

**3、省略枚举名称，直接定义枚举变量**

```c
enum
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
```

实例：

```c
#include <stdio.h>
 
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
 
int main()
{
    enum DAY day;
    day = WED;
    printf("%d",day);
    return 0;
}
```

### 枚举在 switch 中的使用

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    enum color { red=1, green, blue };
 
    enum  color favorite_color;
 
    /* 用户输入数字来选择颜色 */
    printf("请输入你喜欢的颜色: (1. red, 2. green, 3. blue): ");
    
    // scanf 是不安全的，因为他在读取字符串的时候不会检查边界，可能会造成内存泄露
    // scanf("%u", &favorite_color);
    scanf_s("%u", &favorite_color);
 
    /* 输出结果 */
    switch (favorite_color)
    {
    case red:
        printf("你喜欢的颜色是红色");
        break;
    case green:
        printf("你喜欢的颜色是绿色");
        break;
    case blue:
        printf("你喜欢的颜色是蓝色");
        break;
    default:
        printf("你没有选择你喜欢的颜色");
    }
 
    return 0;
}
```

### 将整数转换为枚举

```c
#include <stdio.h>
#include <stdlib.h>
 
int main()
{
 
    enum day
    {
        saturday,
        sunday,
        monday,
        tuesday,
        wednesday,
        thursday,
        friday
    } workday;
 
    int a = 1;
    enum day weekend;
    weekend = ( enum day ) a;  //类型转换
    //weekend = a; //错误
    printf("weekend:%d",weekend);
    return 0;
}
```

结果为：`weekend:1`

### 直接使用枚举

```c
#include <stdio.h>
#include <stdlib.h>

enum {
 Q,W,E=4,R
};

int main()
{
   printf("枚举值QWER分别是: %d , %d , %d , %d",Q,W,E,R);
      return 0;
}
```













