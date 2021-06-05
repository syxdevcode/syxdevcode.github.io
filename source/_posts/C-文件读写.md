---
title: C-文件读写
date: 2021-06-05 13:56:21
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

## 打开文件

可以使用 `fopen()` 函数来创建一个新的文件或者打开一个已有的文件。

```c
FILE *fopen( const char * filename, const char * mode );
```

`filename` 是字符串，用来命名文件；访问模式 `mode` 的值可以是下列值中的一个：

| 模式 | 	描述 |
|:--- | :--- |
|r	|打开一个已有的文本文件，允许读取文件。|
|w	|打开一个文本文件，允许写入文件。如果文件不存在，则会创建一个新文件。在这里，程序会从文件的开头写入内容。如果文件存在，则该会被截断为零长度，重新写入。|
|a	|打开一个文本文件，以追加模式写入文件。如果文件不存在，则会创建一个新文件。在这里，您的程序会在已有的文件内容中追加内容。|
|r+ | 打开一个文本文件，允许读写文件。|
|w+	|打开一个文本文件，允许读写文件。如果文件已存在，则文件会被截断为零长度，如果文件不存在，则会创建一个新文件。|
|a+	|打开一个文本文件，允许读写文件。如果文件不存在，则会创建一个新文件。读取会从文件的开头开始，写入则只能是追加模式。|

如果处理的是二进制文件，则需使用下面的访问模式来取代上面的访问模式：

```c
"rb", "wb", "ab", "rb+", "r+b", "wb+", "w+b", "ab+", "a+b"
```

`fopen` 补充说明：

* 该函数可能执行失败，返回值是`NULL`，安全起见必须对返回值进行合法性判断；
* 该函数有多种模式，其中`r+`和`w+`看似一样，都是读写其实还是有几点区别的；
      1.模式`r+`找不到文件不会自动新建，而`w+`会；
      2.模式`r+`打开文件后，不会清除文件原数据，若直接开始写入，只会从起始位置开始进行覆盖，而`w+`会直接清零后，再开始读写；
* 模式的合法性说明：不能用大写，只能是小写，且`rb+`和`r+b`都是合法的，但`br+`和`+rb`等都是非法的，`w`和`a`也是一样的处理；
* 模式`w`的自动新建文件是有条件的，只有对应的路径存在(即文件所在的文件夹存在)，文件不存在才会新建，否则是不会新建的，返回`NULL`。

## 关闭文件

关闭文件 `fclose()` 函数。函数的原型如下：

```c
int fclose( FILE *fp );
```

如果成功关闭文件，`fclose()` 函数返回零，如果关闭文件时发生错误，函数返回 `EOF`。这个函数实际上，会清空缓冲区中的数据，关闭文件，并释放用于该文件的所有内存。`EOF` 是一个定义在头文件 `stdio.h` 中的常量。

## 写入文件

**把字符写入到流中:**

```c
int fputc( int c, FILE *fp );
```

函数 `fputc()` 把参数 c 的字符值写入到 fp 所指向的输出流中。如果写入成功，它会返回写入的字符，如果发生错误，则会返回 EOF。

**把一个以 `null` 结尾的字符串写入到流中：**

```c
int fputs( const char *s, FILE *fp );
```

函数 `fputs()` 把字符串 s 写入到 fp 所指向的输出流中。如果写入成功，它会返回一个非负值，如果发生错误，则会返回 EOF。

也可以使用 `int fprintf(FILE *fp,const char *format, ...)` 函数把一个字符串写入到文件中。

注意：确保有可用的 `tmp` 目录，如果不存在该目录，则需要在计算机上先创建该目录。

`/tmp` 一般是 `Linux` 系统上的临时目录，如果在 `Windows` 系统上运行，则需要修改为本地环境中已存在的目录，例如: `C:\tmp、D:\tmp`等。

实例

```c
#include <stdio.h>
 
int main()
{
   FILE *fp = NULL;
 
   fp = fopen("/tmp/test.txt", "w+");
   fprintf(fp, "This is testing for fprintf...\n");
   fputs("This is testing for fputs...\n", fp);
   fclose(fp);
}
```

当上面的代码被编译和执行时，它会在 `/tmp` 目录中创建一个新的文件 `test.txt`，并使用两个不同的函数写入两行。

Windows 平台

```c
#include <stdio.h>

int main()
{
    FILE *fp = NULL;
    fp = fopen("C:\\myc\\src\\temp\\test.txt", "w+");
    fprintf(fp, "This is testing for fprintf...\n");
    fputs("This is testing for fputs...\n", fp);
    fclose(fp);
}
```

## 读取文件

**从文件读取单个字符:**

```c
int fgetc(FILE * fp );
```

`fgetc()` 函数从 fp 所指向的输入文件中读取一个字符。返回值是读取的字符，如果发生错误则返回 EOF。

**从流中读取一个字符串：**

```c
char *fgets(char *buf, int n, FILE *fp );
```

函数 `fgets()` 从 fp 所指向的输入流中读取 `n - 1` 个字符。
它会把读取的字符串复制到缓冲区 `buf`，并在最后追加一个 `null` 字符来终止字符串。

如果这个函数在读取最后一个字符之前就遇到一个换行符 `\n` 或文件的末尾 EOF，则只会返回读取到的字符，包括换行符。

`int fscanf(FILE *fp, const char *format, ...)` 函数来从文件中读取字符串，但是在遇到第一个空格和换行符时，它会停止读取。

实例

```c
#include <stdio.h>
 
int main()
{
   FILE *fp = NULL;
   char buff[255];
 
   fp = fopen("/tmp/test.txt", "r");
   fscanf(fp, "%s", buff);
   printf("1: %s\n", buff );
 
   fgets(buff, 255, (FILE*)fp);
   printf("2: %s\n", buff );
   
   fgets(buff, 255, (FILE*)fp);
   printf("3: %s\n", buff );
   fclose(fp); 
}
```

结果：

```c
1: This
2: is testing for fprintf...

3: This is testing for fputs...
```

首先，`fscanf()` 方法只读取了 This，因为它在后边遇到了一个空格。其次，调用 `fgets()` 读取剩余的部分，直到行尾。最后，调用 `fgets()` 完整地读取第二行。

## 二进制 I/O 函数

```c
size_t fread(void *ptr, size_t size_of_elements, 
             size_t number_of_elements, FILE *a_file);
              
size_t fwrite(const void *ptr, size_t size_of_elements, 
             size_t number_of_elements, FILE *a_file);
```

这两个函数都是用于存储块的读写 - 通常是数组或结构体。

`fseek` 可以移动文件指针到指定位置读,或插入写:

```c
int fseek(FILE *stream, long offset, int whence);
```

`fseek` 设置当前读写点到 `offset` 处, `whence` 可以是 `SEEK_SET`,`SEEK_CUR`,`SEEK_END` 这些值决定是从文件头、当前点和文件尾计算偏移量 `offset`。

你可以定义一个文件指针 `FILE *fp`,当你打开一个文件时，文件指针指向开头，你要指到多少个字节，只要控制偏移量就好。

例如, 相对当前位置往后移动一个字节：`fseek(fp,1,SEEK_CUR);` 中间的值就是偏移量。
如果你要往前移动一个字节，直接改为负值就可以：`fseek(fp,-1,SEEK_CUR)`。

执行以下实例前，确保当前目录下 `test.txt` 文件已创建：

```c
#include <stdio.h>

int main(){   
    FILE *fp = NULL;
    fp = fopen("test.txt", "r+");  // 确保 test.txt 文件已创建
    fprintf(fp, "This is testing for fprintf...\n");   
    fseek(fp, 10, SEEK_SET);
    if (fputc(65,fp) == EOF) {
        printf("fputc fail");   
    }   
    fclose(fp);
}
```

执行结束后，打开 test.txt 文件：

```
This is teAting for fprintf...
```

注意： 只有用 `r+` 模式打开文件才能插入内容，`w` 或 `w+` 模式都会清空掉原来文件的内容再来写，`a` 或 `a+` 模式即总会在文件最尾添加内容，哪怕用 `fseek()` 移动了文件指针位置。

**fopen_s**

在新版的 VS 编译环境中提示 `fopen` 不安全，推荐使用 `fopen_s` 代替。
`fopen_s` 和 `fopen` 的区别在于：

```c
errno_t fopen_s( FILE** pFile, const char *filename, const char *mode );
FILE *fopen(const char *filename, const char *mode)
```

`fopen_s` 有三个参数，第一个参数是二级指针，用于存放文件流指针地址，其他的同 `fopen` 一致。

也就是说，用 `fopen` 需要这么写：

```c
FILE* fp=NULL;
fp=fopen("\\123.txt","w+");
```

而如果用 `fopen_s`：

```c
FILE* fp=NULL;
fopen_s(&fp,"\\123.txt","w+");
```

[C 文件读写](https://www.runoob.com/cprogramming/c-file-io.html)