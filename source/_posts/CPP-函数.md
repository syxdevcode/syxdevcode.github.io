---
title: CPP-函数
date: 2021-06-29 10:39:01
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

函数是一组一起执行一个任务的语句。每个 C++ 程序都至少有一个函数，即主函数 `main()` ，所有简单的程序都可以定义其他额外的函数。

C++ 标准库提供了大量的程序可以调用的内置函数。例如，函数 `strcat()` 用来连接两个字符串，函数 `memcpy()` 用来复制内存到另一个位置。

<!--more-->
## 定义函数

C++ 中的函数定义的一般形式如下：

```cpp
return_type function_name( parameter list )
{
   body of the function
}
```

实例：

```cpp
// 函数返回两个数中较大的那个数
int max(int num1, int num2) 
{
   // 局部变量声明
   int result;
 
   if (num1 > num2)
      result = num1;
   else
      result = num2;
 
   return result; 
}
```

## 函数声明

函数声明会告诉编译器函数名称及如何调用函数。函数的实际主体可以单独定义。

函数声明包括以下几个部分：

```cpp
return_type function_name( parameter list );
```

在函数声明中，参数的名称并不重要，只有参数的类型是必需的，因此下面也是有效的声明：

```cpp
int max(int, int);
```

在一个源文件中定义函数，且在另一个文件中调用函数时，函数声明是必需的。在这种情况下，应该在调用函数的文件顶部声明函数。

## 调用函数

当程序调用函数时，程序控制权会转移给被调用的函数。被调用的函数执行已定义的任务，当函数的返回语句被执行时，或到达函数的结束括号时，会把程序控制权交还给主程序。

调用函数时，传递所需参数，如果函数返回一个值，则可以存储返回值。例如：

实例

```cpp
#include <iostream>
using namespace std;
 
// 函数声明
int max(int num1, int num2);
 
int main ()
{
   // 局部变量声明
   int a = 100;
   int b = 200;
   int ret;
 
   // 调用函数来获取最大值
   ret = max(a, b);
 
   cout << "Max value is : " << ret << endl;
 
   return 0;
}
 
// 函数返回两个数中较大的那个数
int max(int num1, int num2) 
{
   // 局部变量声明
   int result;
 
   if (num1 > num2)
      result = num1;
   else
      result = num2;
 
   return result; 
}
```

结果：`Max value is : 200`

## 函数参数

如果函数要使用参数，则必须声明接受参数值的变量。这些变量称为函数的形式参数。

形式参数就像函数内的其他局部变量，在进入函数时被创建，退出函数时被销毁。

### 传值调用

向函数传递参数的传值调用方法，把参数的实际值复制给函数的形式参数，修改函数内的形式参数不会影响实际参数。

默认情况下，C++ 使用传值调用方法来传递参数。一般来说，这意味着函数内的代码不会改变用于调用函数的实际参数。

实例：

```cpp
#include <iostream>
using namespace std;

 
// 函数声明
void swap(int x, int y);
 
int main ()
{
   // 局部变量声明
   int a = 100;
   int b = 200;
 
   cout << "交换前，a 的值：" << a << endl;
   cout << "交换前，b 的值：" << b << endl;
 
   // 调用函数来交换值
   swap(a, b);
 
   cout << "交换后，a 的值：" << a << endl;
   cout << "交换后，b 的值：" << b << endl;
 
   return 0;
}

// 函数定义
void swap(int x, int y)
{
   int temp;
 
   temp = x; /* 保存 x 的值 */
   x = y;    /* 把 y 赋值给 x */
   y = temp; /* 把 x 赋值给 y */
  
   return;
}
```

结果：

```cpp
交换前，a 的值： 100
交换前，b 的值： 200
交换后，a 的值： 100
交换后，b 的值： 200
```

### 指针调用

向函数传递参数的指针调用方法，把参数的地址复制给形式参数。在函数内，该地址用于访问调用中要用到的实际参数，修改形式参数会影响实际参数。

实例：

```cpp
#include <iostream>
using namespace std;

// 函数声明
void swap(int *x, int *y);

int main ()
{
   // 局部变量声明
   int a = 100;
   int b = 200;
 
   cout << "交换前，a 的值：" << a << endl;
   cout << "交换前，b 的值：" << b << endl;

   /* 调用函数来交换值
    * &a 表示指向 a 的指针，即变量 a 的地址 
    * &b 表示指向 b 的指针，即变量 b 的地址 
    */
   swap(&a, &b);

   cout << "交换后，a 的值：" << a << endl;
   cout << "交换后，b 的值：" << b << endl;
 
   return 0;
}

// 函数定义
void swap(int *x, int *y)
{
   int temp;
   temp = *x;    /* 保存地址 x 的值 */
   *x = *y;        /* 把 y 赋值给 x */
   *y = temp;    /* 把 x 赋值给 y */
  
   return;
}
```

结果：

```cpp
交换前，a 的值： 100
交换前，b 的值： 200
交换后，a 的值： 200
交换后，b 的值： 100
```

### 引用调用

向函数传递参数的引用调用方法，把引用的地址复制给形式参数。在函数内，该引用用于访问调用中要用到的实际参数，修改形式参数会影响实际参数。

按引用传递值，参数引用被传递给函数，就像传递其他值给函数一样。

```cpp
#include <iostream>
using namespace std;
 
// 函数声明
void swap(int &x, int &y);
 
int main ()
{
   // 局部变量声明
   int a = 100;
   int b = 200;
 
   cout << "交换前，a 的值：" << a << endl;
   cout << "交换前，b 的值：" << b << endl;
 
   /* 调用函数来交换值 */
   swap(a, b);
 
   cout << "交换后，a 的值：" << a << endl;
   cout << "交换后，b 的值：" << b << endl;
 
   return 0;
}

// 函数定义
void swap(int &x, int &y)
{
   int temp;
   temp = x; /* 保存地址 x 的值 */
   x = y;    /* 把 y 赋值给 x */
   y = temp; /* 把 x 赋值给 y  */
  
   return;
}
```

结果：

```cpp
交换前，a 的值： 100
交换前，b 的值： 200
交换后，a 的值： 200
交换后，b 的值： 100
```

### 参数的默认值

定义一个函数的时候，可以为参数列表中后边的每一个参数指定默认值。当调用函数时，如果实际参数的值留空，则使用这个默认值。

实例：

```cpp
#include <iostream>
using namespace std;
 
int sum(int a, int b=20)
{
  int result;
 
  result = a + b;
  
  return (result);
}
 
int main ()
{
   // 局部变量声明
   int a = 100;
   int b = 200;
   int result;
 
   // 调用函数来添加值
   result = sum(a, b);
   cout << "Total value is :" << result << endl;
 
   // 再次调用函数
   result = sum(a);
   cout << "Total value is :" << result << endl;
 
   return 0;
}
```

结果：

```cpp
Total value is :300
Total value is :120
```

## Lambda函数与表达式

C++11 提供了对匿名函数的支持,称为 `Lambda` 函数(也叫 `Lambda` 表达式)。

`Lambda` 表达式把函数看作对象。`Lambda` 表达式可以像对象一样使用，比如可以将它们赋给变量和作为参数传递，还可以像函数一样对其求值。

`Lambda` 表达式本质上与函数声明非常类似。`Lambda` 表达式具体形式如下:

```cpp
[capture](parameters) mutable ->return-type{statement}
```

* **[capture]**：捕捉列表。捕捉列表总是出现在 `lambda` 表达式的开始处。事实上，`[]` 是 `lambda` 引出符。编译器根据该引出符判断接下来的代码是否是 `lambda` 函数。捕捉列表能够捕捉上下文中的变量供 `lambda` 函数使用。
* **(parameters)**：参数列表。与普通函数的参数列表一致。如果不需要参数传递，则可以连同括号 `()` 一起省略。
* **mutable**：`mutable` 修饰符。默认情况下，`lambda` 函数总是一个 `const` 函数，`mutable` 可以取消其常量性。在使用该修饰符时，参数列表不可省略（即使参数为空）。
* **->return_type**：返回类型。用追踪返回类型形式声明函数的返回类型。出于方便，不需要返回值的时候也可以连同符号 `->` 一起省略。此外，在返回类型明确的情况下，也可以省略该部分，让编译器对返回类型进行推导。
* **{statement}**：函数体。内容与普通函数一样，不过除了可以使用参数之外，还可以使用所有捕获的变量。

在 `lambda` 函数的定义式中，参数列表和返回类型都是可选部分，而捕捉列表和函数体都可能为空，C++ 中最简单的 `lambda` 函数只需要声明为：

```cpp
[]{};
```

实例：

```cpp
// 自动推断返回类型
[](int x, int y){ return x < y ; }

[]{ ++global_x; } 

// 指明返回类型
[](int x, int y) -> int { int z = x + y; return z + x; }
```

在 `Lambda` 表达式内可以访问当前作用域的变量，这是 `Lambda` 表达式的闭包（`Closure`）行为。 可以通过前面的 `[]` 来指定：

```cpp
[]：默认不捕获任何变量；
[=]：默认以值捕获所有变量；
[&]：默认以引用捕获所有变量；
[x]：仅以值捕获x，其它变量不捕获；
[&x]：仅以引用捕获x，其它变量不捕获；
[x, &y] // x以传值方式传入（默认），y以引用方式传入。
[=, &x]：默认以值捕获所有变量，但是x是例外，通过引用捕获；
[&, x]：默认以引用捕获所有变量，但是x是例外，通过值捕获；
[this]：通过引用捕获当前对象（其实是复制指针）；
[*this]：通过传值方式捕获当前对象；
```

另外有一点需要注意。对于`[=]`或`[&]`的形式，`lambda` 表达式可以直接使用 `this` 指针。但是，对于`[]`的形式，如果要使用 `this` 指针，必须显式传入：

```cpp
[this]() { this->someFunc(); }();
```

实例：

```cpp
int main()
{
    int x = 10;

    // mutable 取消lambda函数常量性
    auto add_x = [x](int a) mutable { x *= 2; return a + x; };  // 复制捕捉x

    cout << add_x(10) << endl; // 输出 30
    return 0;
}
```

实例：

1，以传值方式捕获。

```cpp
#include <iostream>
using namespace std;
 
int main()
{
    int i = 1024;
    auto func = [=]{  // [=] 表明将外部的所有变量拷贝一份到该Lambda函数内部
        cout << i;
    };
    func();
}
```

2，引用捕获

```cpp
#include <iostream>
using namespace std;
 
int main()
{
    int i = 1024;
    cout << &i << endl;
    auto fun1 = [&]{
        cout << &i << endl;
    };
    fun1();
}
```

3，复制并引用捕获

```cpp
#include <iostream>
using namespace std;
 
int main()
{
    int i = 1024, j = 2048;
 
    cout << "i:" << &i << endl;
    cout << "j:" << &j << endl;
 
    auto fun1 = [=, &i]{ // 默认拷贝外部所有变量，但引用变量 i
        cout << "i:" << &i << endl;
        cout << "j:" << &j << endl;
    };
    fun1();
}
```

4，指定引用或复制

```cpp
#include <iostream>
using namespace std;
 
int main()
{
    int i = 1024, j = 2048;
 
    cout << "outside i value:" << i << " addr:" << &i << endl;
 
    auto fun1 = [i]{
        cout << "inside  i value:" << i << " addr:" << &i << endl;
        // cout << j << endl; // j 未捕获
    };
    fun1();
}
```

5，捕获this指针

```cpp
#include <iostream>
using namespace std;
 
class test
{
public:
    void hello() {
        cout << "test hello!n";
    };
    void lambda() {
        auto fun = [this]{ // 捕获了 this 指针
            this->hello(); // 这里 this 调用的就是 class test 的对象了
        };
        fun();
    }
};
 
int main()
{
    test t;
    t.lambda();
}
```