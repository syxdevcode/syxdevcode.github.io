---
title: CPP-循环与判断
date: 2021-06-29 09:43:17
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

## 循环语句

* while 循环
* for 循环
* do...while 循环

**break 语句**	终止循环或 `switch` 语句，程序流将继续执行紧接着循环或 `switch` 的下一条语句。
**continue 语句**	告诉一个循环体立刻停止本次循环迭代，重新开始下次循环迭代。

### while循环

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 局部变量声明
   int a = 10;

   // while 循环执行
   while( a < 20 )
   {
       cout << "a 的值：" << a << endl;
       a++;
   }
 
   return 0;
}
```

<!--more-->
### for循环

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // for 循环执行
   for( int a = 10; a < 20; a = a + 1 )
   {
       cout << "a 的值：" << a << endl;
   }
 
   return 0;
}
```

### do...while循环

`do...while` 循环与 `while` 循环类似，但是 `do...while` 循环会确保至少执行一次循环。

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 局部变量声明
   int a = 10;

   // do 循环执行
   do
   {
       cout << "a 的值：" << a << endl;
       a = a + 1;
   }while( a < 20 );
 
   return 0;
}
```

### goto 语句

注意：在任何编程语言中，都不建议使用 `goto` 语句。因为它使得程序的控制流难以跟踪，使程序难以理解和难以修改。任何使用 `goto` 语句的程序可以改写成不需要使用 `goto` 语句的写法。

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 局部变量声明
   int a = 10;

   // do 循环执行
   LOOP:do
   {
       if( a == 15)
       {
          // 跳过迭代
          a = a + 1;
          goto LOOP;
       }
       cout << "a 的值：" << a << endl;
       a = a + 1;
   }while( a < 20 );
 
   return 0;
}
```

结果：

```cpp
a 的值： 10
a 的值： 11
a 的值： 12
a 的值： 13
a 的值： 14
a 的值： 16
a 的值： 17
a 的值： 18
a 的值： 19
```

### 无限循环

`for` 循环在传统意义上可用于实现无限循环。由于构成循环的三个表达式中任何一个都不是必需的，可以将某些条件表达式留空来构成一个无限循环。

实例

```cpp
#include <iostream>
using namespace std;

int main ()
{
   for( ; ; )
   {
      cout << "该循环会永远执行下去" << endl;
   }
   return 0;
}
```

当条件表达式不存在时，它被假设为真。可以设置一个初始值和增量表达式，但是一般情况下，C 程序员偏向于使用 `for(;;)` 结构来表示一个无限循环。

注意：可以按 `Ctrl + C` 键终止一个无限循环。

## 判断语句

### if 语句

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 局部变量声明
   int a = 10;
 
   // 使用 if 语句检查布尔条件
   if( a < 20 )
   {
       // 如果条件为真，则输出下面的语句
       cout << "a 小于 20" << endl;
   }
   cout << "a 的值是 " << a << endl;
 
   return 0;
}
```

## if...else 语句

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 局部变量声明
   int a = 100;
 
   // 检查布尔条件
   if( a < 20 )
   {
       // 如果条件为真，则输出下面的语句
       cout << "a 小于 20" << endl;
   }
   else
   {
       // 如果条件为假，则输出下面的语句
       cout << "a 大于 20" << endl;
   }
   cout << "a 的值是 " << a << endl;
 
   return 0;
}
```

## switch 语句

```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 局部变量声明
   char grade = 'D';
 
   switch(grade)
   {
   case 'A' :
      cout << "很棒！" << endl; 
      break;
   case 'B' :
   case 'C' :
      cout << "做得好" << endl;
      break;
   case 'D' :
      cout << "您通过了" << endl;
      break;
   case 'F' :
      cout << "最好再试一下" << endl;
      break;
   default :
      cout << "无效的成绩" << endl;
   }
   cout << "您的成绩是 " << grade << endl;
 
   return 0;
}
```

## ? : 运算符(三元运算符)

```cpp
Exp1 ? Exp2 : Exp3;
```

如果 Exp1 为真，则计算 Exp2 的值。
如果 Exp1 为假，则计算 Exp3 的值。

比较两个数的大小:

```cpp
#include<iostream>
using namespace std;

int main(){
    int a,b;
    cout<<"请输入两个数字：";
    cin>>a>>b;
    a>b?cout<<a<<"大于"<<b<<endl:cout<<b<<"大于"<<a<<endl;
    return 0;
}
```