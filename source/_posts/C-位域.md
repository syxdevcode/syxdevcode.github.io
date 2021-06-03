---
title: C-位域
date: 2021-06-03 22:48:14
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

如果程序的结构中包含多个开关量，只有 `TRUE/FALSE` 变量，如下：

```c
struct
{
  unsigned int widthValidated;
  unsigned int heightValidated;
} status;
```

这种结构需要 8 字节的内存空间，但在实际上，在每个变量中，只存储 0 或 1。

在这种情况下，C 语言提供了一种更好的利用内存空间的方式。如果在结构内使用这样的变量，可以定义变量的宽度来告诉编译器，将只使用这些字节。例如，上面的结构可以重写成：

```c
struct
{
  unsigned int widthValidated : 1;
  unsigned int heightValidated : 1;
} status;
```

现在，上面的结构中，`status` 变量将占用 4 个字节的内存空间，但是只有 2 位被用来存储值。如果您用了 32 个变量，每一个变量宽度为 1 位，那么 `status` 结构将使用 4 个字节，但只要您再多用一个变量，如果使用了 33 个变量，那么它将分配内存的下一段来存储第 33 个变量，这个时候就开始使用 8 个字节。

实例

```c
#include <stdio.h>
#include <string.h>
 
/* 定义简单的结构 */
struct
{
  unsigned int widthValidated;
  unsigned int heightValidated;
} status1;
 
/* 定义位域结构 */
struct
{
  unsigned int widthValidated : 1;
  unsigned int heightValidated : 1;
} status2;
 
int main( )
{
   printf( "Memory size occupied by status1 : %d\n", sizeof(status1));
   printf( "Memory size occupied by status2 : %d\n", sizeof(status2));
 
   return 0;
}
```

结果：

```c
Memory size occupied by status1 : 8
Memory size occupied by status2 : 4
```

## 位域声明

有些信息在存储时，并不需要占用一个完整的字节，而只需占几个或一个二进制位。例如在存放一个开关量时，只有 0 和 1 两种状态，用 1 位二进位即可。为了节省存储空间，并使处理简便，C 语言又提供了一种数据结构，称为`位域`或 `位段`。

所谓`位域`是把一个字节中的二进位划分为几个不同的区域，并说明每个区域的位数。每个域有一个域名，允许在程序中按域名进行操作。这样就可以把几个不同的对象用一个字节的二进制位域来表示。

典型的实例：

* 用 1 位二进位存放一个开关量时，只有 0 和 1 两种状态。
* 读取外部文件格式——可以读取非标准的文件格式。例如：9 位的整数。

## 位域的定义和位域变量的说明

位域定义与结构定义相仿，其形式为：

```c
struct 位域结构名 
{
 位域列表
};
```

其中位域列表的形式为：

```c
type [member_name] : width ;
```

* **type** ：只能为 `int(整型)`，`unsigned int(无符号整型)`，`signed int(有符号整型) `三种类型，决定了如何解释位域的值。
* **member_name** ：位域的名称。
* **width**：位域中位的数量。宽度必须小于或等于指定类型的位宽度。

带有预定义宽度的变量被称为`位域`。位域可以存储多于 1 位的数，例如，需要一个变量来存储从 0 到 7 的值，可以定义一个宽度为 3 位的位域，如下：

结构定义指示 C 编译器，age 变量将只使用 3 位来存储这个值，如果试图使用超过 3 位，则无法完成。

实例1：

```c
struct
{
  unsigned int age : 3;
} Age;
```

实例2：
 
```c
struct bs{
    int a:8;
    int b:2;
    int c:6;
}data;
```

`data` 为 bs 变量，共占两个字节。其中位域 a 占 8 位，位域 b 占 2 位，位域 c 占 6 位。

实例3：

```c
struct packed_struct {
  unsigned int f1:1;
  unsigned int f2:1;
  unsigned int f3:1;
  unsigned int f4:1;
  unsigned int type:4;
  unsigned int my_int:9;
} pack;
```

`packed_struct` 包含了 6 个成员：四个 1 位的标识符 `f1..f4`、一个 4 位的 `type` 和一个 9 位的 `my_int`。

实例4：

```c
#include <stdio.h>
#include <string.h>
 
struct
{
  unsigned int age : 3;
} Age;
 
int main( )
{
   Age.age = 4;
   printf( "Sizeof( Age ) : %d\n", sizeof(Age) );
   printf( "Age.age : %d\n", Age.age );
 
   Age.age = 7;
   printf( "Age.age : %d\n", Age.age );
 
   Age.age = 8; // 二进制表示为 1000 有四位，超出
   printf( "Age.age : %d\n", Age.age );
 
   return 0;
}
```

当上面的代码被编译时，它会带有警告，当上面的代码被执行时，它会产生下列结果：

```c
Sizeof( Age ) : 4
Age.age : 4
Age.age : 7
Age.age : 0
```

对于位域的定义的说明：

* 1，一个位域存储在同一个字节中，如一个字节所剩空间不够存放另一位域时，则会从下一单元起存放该位域。也可以有意使某位域从下一单元开始。例如：

```c
struct bs{
    unsigned a:4;
    unsigned  :4;    /* 空域 */
    unsigned b:4;    /* 从下一单元开始存放 */
    unsigned c:4
}
```

在这个位域定义中，a 占第一字节的 4 位，后 4 位填 0 表示不使用，b 从第二字节开始，占用 4 位，c 占用 4 位。

* 位域的宽度不能超过它所依附的数据类型的长度，成员变量都是有类型的，这个类型限制了成员变量的最大长度，`:` 后面的数字不能超过这个长度。
* 位域可以是无名位域，这时它只用来作填充或调整位置。无名的位域是不能使用的。例如：

```c
struct k{
    int a:1;
    int  :2;    /* 该 2 位不能使用 */
    int b:3;
    int c:2;
};
```

从以上分析可以看出，位域在本质上就是一种结构类型，不过其成员是按二进位分配的。

## 位域的使用

位域的使用和结构成员的使用相同，其一般形式为：

```c
位域变量名.位域名 (使用)
位域变量名->位域名 （赋值）
```

位域允许用各种格式输出。

实例：

```c
int main(){
    struct bs{
        unsigned a:1;
        unsigned b:3;
        unsigned c:4;
    } bit,*pbit;
    bit.a=1;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */
    bit.b=7;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */
    bit.c=15;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */
    printf("%d,%d,%d\n",bit.a,bit.b,bit.c);    /* 以整型量格式输出三个域的内容 */
    pbit=&bit;    /* 把位域变量 bit 的地址送给指针变量 pbit */
    pbit->a=0;    /* 用指针方式给位域 a 重新赋值，赋为 0 */
    pbit->b&=3;    /* 使用了复合的位运算符 "&="，相当于：pbit->b=pbit->b&3，位域 b 中原有值为 7，与 3 作按位与运算的结果为 3（111&011=011，十进制值为 3） */
    pbit->c|=1;    /* 使用了复合位运算符"|="，相当于：pbit->c=pbit->c|1，其结果为 15 */
    printf("%d,%d,%d\n",pbit->a,pbit->b,pbit->c);    /* 用指针方式输出了这三个域的值 */
}
```

程序中定义了位域结构 bs，三个位域为 a、b、c。说明了 bs 类型的变量 bit 和指向 bs 类型的指针变量 pbit。这表示位域也是可以使用指针的。

## 结构体内存分配原则

（1）结构体内存分配原则：
* 原则一：结构体中元素按照定义顺序存放到内存中，但并不是紧密排列。从结构体存储的首地址开始 ，每一个元素存入内存中时，它都会认为内存是以自己的宽度来划分空间的，因此元素存放的位置一定会在自己大小的整数倍上开始。
* 原则二： 在原则一的基础上，检查计算出的存储单元是否为所有元素中最宽的元素长度的整数倍。若是，则结束；否则，将其补齐为它的整数倍。

（2）定义位域时，各个成员的类型最好保持一致，比如都用char，或都用int，不要混合使用，这样才能达到节省内存空间的目的。

```c
// 位域内存测试
#include <stdio.h>
struct ONE_BYTE
{
    unsigned char _bool : 1;
    unsigned char del_flag : 1;
    unsigned char status : 4;
} one_byte;

struct TWO_BYTE
{
    unsigned char ccc1 : 4;
    unsigned char ccc2 : 4;
    unsigned char ccc3 : 4;
    unsigned char ccc4 : 4;
} two_byte;

struct THREE_BYTE
{
    unsigned char ccc1 : 4;
    unsigned char ccc2 : 4;
    unsigned char ccc3 : 4;
    unsigned char ccc4 : 4;
    unsigned char ccc5 : 4;
} three_byte;

struct FOUR_BYTE
{
    unsigned int ccc1 : 16;
    unsigned int ccc2 : 16;
} four_byte;


struct EIGHT_BYTE
{
    unsigned char ccc1 : 1;
    unsigned int ccc2 : 1;
} eight_byte;

int main(int argc, char const *argv[])
{
    printf("sizeof one_byte is : %lu\n", sizeof(one_byte));
    printf("sizeof two_byte is : %lu\n", sizeof(two_byte));
    printf("sizeof three_byte is : %lu\n", sizeof(three_byte));
    printf("sizeof four_byte is : %lu\n", sizeof(four_byte));
    printf("sizeof eight_byte is : %lu\n", sizeof(eight_byte));
    return 0;
}
```

结果为：

```c
sizeof one_byte is : 1B
sizeof two_byte is : 2B
sizeof three_byte is : 3B
sizeof four_byte is : 4B
sizeof eight_byte is : 4B
```

[C 位域](https://www.runoob.com/cprogramming/c-bit-fields.html)