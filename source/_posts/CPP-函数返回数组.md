---
title: CPP-函数返回数组
date: 2021-06-29 16:49:48
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

C++中函数是不能直接返回一个数组的，但是数组其实就是指针，所以可以让函数返回指针来实现。

```cpp
int * myFunction()
{
.
.
}
```

注意：C++ 不支持在函数外返回局部变量的地址，除非定义局部变量为 static 变量。

**实例1**

```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>

using namespace std;

// 要生成和返回随机数的函数
int *getRandom()
{
    static int r[10];

    // 设置种子
    srand((unsigned)time(NULL));
    for (int i = 0; i < 10; ++i)
    {
        r[i] = rand();
        cout << r[i] << endl;
    }

    return r;
}

// 要调用上面定义函数的主函数
int main()
{
    // 一个指向整数的指针
    int *p;

    p = getRandom();
    for (int i = 0; i < 10; i++)
    {
        cout << "*(p + " << i << ") : ";
        cout << *(p + i) << endl;
    }

    return 0;
}
```

结果：

```cpp
8270
28596
6574
20207
25231
26290
20914
19752
32086
15690
*(p + 0) : 8270
*(p + 1) : 28596
*(p + 2) : 6574
*(p + 3) : 20207
*(p + 4) : 25231
*(p + 5) : 26290
*(p + 6) : 20914
*(p + 7) : 19752
*(p + 8) : 32086
*(p + 9) : 15690
```

**实例2**

```cpp
#include <iostream>

using namespace std;

void MultMatrix(float M[4], float A[4], float B[4])
{
    M[0] = A[0] * B[0] + A[1] * B[2];
    M[1] = A[0] * B[1] + A[1] * B[3];
    M[2] = A[2] * B[0] + A[3] * B[2];
    M[3] = A[2] * B[1] + A[3] * B[3];

    cout << M[0] << " " << M[1] << endl;
    cout << M[2] << " " << M[3] << endl;
}

int main()
{
    float A[4] = {1.75, 0.66, 0, 1.75};
    float B[4] = {1, 1, 0, 0};

    float *M = new float[4];
    MultMatrix(M, A, B);

    cout << M[0] << " " << M[1] << endl;
    cout << M[2] << " " << M[3] << endl;
    delete[] M;

    return 0;
}
```

数组的 `delete` 是 `delete[]`。
其次，C++ 里面手动内存分配的一个重要原则是谁分配谁释放。
所以，不应该在 `MultMatrix` 里 `new` 数组，而应该在外面 `new` 好了之后传进去修改。

结果：

```cpp
1.75 1.75
0 0
1.75 1.75
0 0
```s