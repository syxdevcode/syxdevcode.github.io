---
title: ELF格式文件解析
date: 2021-03-24 15:13:06
tags:
- Linux
- CentOS7
- 物联网
- 嵌入式
- ELF
categories:
- ELF
---

## 对象文件(Object files)分类

### 一，可重定位的对象文件(Relocatable file)

&emsp;&emsp;由汇编器汇编生成的 `.o` 文件。后面的链接器(link editor)拿一个或一些 Relocatable object files 作为输入，经链接处理后，生成一个可执行的对象文件 (Executable file) 或者一个可被共享的对象文件(Shared object file)。可以使用 `ar` 工具将众多的 `.o` (Relocatable object files) 归档(archive)成 `.a` 静态库文件。内核可加载模块 `.ko` 文件也是 Relocatable object file。

### 二，可执行的对象文件(Executable file)

&emsp;&emsp;文本编辑器vi、调式用的工具gdb、播放mp3歌曲的软件mplayer等等都是 Executable object file。在 Linux 系统里面，存在两种可执行的东西。除了 Executable object file，另外一种就是可执行的脚本(如shell脚本)。注意这些脚本不是 Executable object file，它们只是文本文件，但是执行这些脚本所用的解释器就是 Executable object file，比如 bash shell 程序。

### 三，可被共享的对象文件(Shared object file)

&emsp;&emsp;可被共享的对象文件，即 动态库文件，也即 `.so` 文件。

动态库在发挥作用的过程中，必须经过两个步骤：

* a) 链接编辑器(link editor)拿它和其他 Relocatable object file 以及其他 shared object file 作为输入，经链接处理后，生存另外的 shared object file 或者 executable file。
* b) 在运行时，动态链接器(dynamic linker)拿它和一个 Executable file 以及另外一些 Shared object file 来一起处理，在 Linux 系统里面创建一个进程映像。

## gcc翻译过程

![20170611205306090.png](/img/20170611205306090.png)

在Unix系统中，从源文件到可执行目标文件是由编译驱动程序完成的，如大名鼎鼎的gcc，翻译过程包括图中的是个阶段；

**一，预处理阶段**

预处理器（cpp）根据以字符#开头的命令修给原始的C程序，结果得到另一个C程序，通常以.i作为文件扩展名。主要是进行文本替换、宏展开、删除注释这类简单工作。

对应的命令：`linux> gcc -E hello.c hello.i`

**二，编译阶段**

编译器将文本文件hello.i翻译成hello.s，包含相应的汇编语言程序

对应的命令：`linux> gcc -S hello.c hello.s`

**三，汇编阶段**

将.s文件翻译成机器语言指令，把这些指令打包成一种叫做可重定位目标程序的格式，并将结果保存在目标文件.o中(把汇编语言翻译成机器语言的过程)。

把一个源程序翻译成目标程序的工作过程分为五个阶段：词法分析；语法分析；语义检查和中间代码生成；代码优化；目标代码生成。主要是进行词法分析和语法分析，又称为源程序分析，分析过程中发现有语法错误，给出提示信息。

对应的命令：`linux> gcc -c hello.c hello.o`

**四，链接阶段**

&emsp;&emsp;此时hello程序调用了 printf 函数。 printf 函数存在于一个名为printf.o的单独的预编译目标文件中。 链接器（ld）就负责处理把这个文件并入到 hello.o 程序中，结果得到 hell.o 文件，一个可执行文件。最后可执行文件加载到储存器后由系统负责执行。

函数库一般分为静态库和动态库两种。

* 静态库是指编译链接时，把库文件的代码全部加入到可执行文件中，因此生成的文件比较大，但在运行时也就不再需要库文件了。其后缀名一般为 `.a`。
* 动态库与之相反，在编译链接时并没有把库文件的代码加入到可执行文件中，而是在程序执行时由运行时链接文件加载库，这样可以节省系统的开销。动态库一般后缀名为 `.so`，gcc 在编译时默认使用动态库。

## ELF文件格式

&emsp;&emsp;ELF 全称 `Executable and Linkable Format`，可执行可链接文件格式，目前常见的 Linux、 Android 可执行文件、共享库（`.so`）、目标文件（`.o`）以及Core 文件（吐核）均为此格式。

&emsp;&emsp;ELF 文件由4部分组成，分别是 ELF 头（ELF header）、程序头表（Program header table）、节（Section）和节区头部表（Section header table）。ELF 头的位置是固定的，其余各部分的位置、大小等信息由ELF头中的各项值来决定。

![20160521110158483.png](/img/20160521110158483.png)

* ELF header： 描述整个文件的组织。
* Program Header Table: 描述文件中的各种 segments，用来告诉系统如何创建进程映像的。
* sections 或者 segments：segments是从运行的角度来描述elf文件，sections是从链接的角度来描述elf文件，也就是说，在链接阶段，我们可以忽略program header table来处理此文件，在运行阶段可以忽略section header table来处理此程序（所以很多加固手段删除了section header table）。从图中我们也可以看出，segments与sections是包含的关系，一个segment包含若干个section。
* Section Header Table: 包含了文件各个segction的属性信息。

&emsp;&emsp;ELF 文件格式提供了两种视图，分别是链接视图和执行视图，链接视图是以节（section）为单位，执行视图是以段（segment）为单位。

&emsp;&emsp;在汇编器和链接器看来，ELF 文件是由 Section Header Table 描述的一系列 Section 的集合，而执行一个 ELF 文件时，在加载器（Loader）看来它是由 Program Header Table 描述的一系列Segment的集合。

![elf.png](/img/elf.png)

**程序头部表**（Program Header Table），如果存在的话，告诉系统如何创建进程映像。
**节区头部表**（Section Header Table）包含了描述文件节区的信息，比如大小、偏移等。

执行命令 `readelf -S android_server` 来查看该可执行文件中有哪些section。

![20160521110452230.png](/img/20160521110452230.png)

行命令 `readelf –segments android_server`，可以查看该文件的执行视图。

![20160521110508766.png](/img/20160521110508766.png)

**segment是section的一个集合，sections按照一定规则映射到segment。为什么需要区分两种不同视图？**

&emsp;&emsp;当ELF文件被加载到内存中后，系统会将多个具有相同权限（flg值）section合并一个segment。操作系统往往以页为基本单位来管理内存分配，一般页的大小为4096B，即4KB的大小。同时，内存的权限管理的粒度也是以页为单位，页内的内存是具有同样的权限等属性，并且操作系统对内存的管理往往追求高效和高利用率这样的目标。ELF文件在被映射时，是以系统的页长度为单位的，那么每个section在映射时的长度都是系统页长度的整数倍，如果section的长度不是其整数倍，则导致多余部分也将占用一个页。而我们从上面的例子中知道，一个ELF文件具有很多的section，那么会导致内存浪费严重。这样可以减少页面内部的碎片，节省了空间，显著提高内存利用率。

### ELF Header

转载自：[ELF文件格式解析](https://blog.csdn.net/feglass/article/details/51469511?spm=1001.2014.3001.5501)

32位ELF文件中常用的数据格式：

![20160521110646983.png](/img/20160521110646983.png)

`readelf -h android_server` 命令，可以看到 ELF Header 结构的内容:

![20160521110756954.png](/img/20160521110756954.png)

对比以下三类ELF文件，我们得到了以下结论：
（1）e_type标识了文件类型
（2）Relocatable File（.o文件）不需要执行，因此e_entry字段为0，且没有Program Header Table等执行视图
（3）不同类型的ELF文件的Section也有较大区别，比如只有Relocatable File有.strtab节。 

(1) Shared Object File（.so文件）:
![20160521110949018.png](/img/20160521110949018.png)

(2) Executable File（可执行文件android_server）:

![20160521111008408.png](/img/20160521111008408.png)

(3) Relocatable File（.o文件）:

![20160521111020956.png](/img/20160521111020956.png)

在 ELF Header 中需要重点关注以下几个字段：

* 1，e_entry：程序入口地址
这个 sum.o 的进入点是 0x0(e_entry)，这表面Relocatable objects不会有程序进入点。所谓程序进入点是指当程序真正执行起来的时候，其第一条要运行的指令的运行时地址。因为Relocatable objects file只是供再链接而已，所以它不存在进入点。而可执行文件test和动态库.so都存在所谓的进入点，且可执行文件的 e_entry 指向C库中的_start，而动态库.so中的进入点指向 call_gmon_start。
如上图中 e_entry = 0xD8B0 (Executable File 文件)，我们用ida打开该文件看到确实是 _start() 函数的地址。

![20160521111227521.png](/img/20160521111227521.png)

* 2，e_ehsize：ELF Header结构大小
* 3，e_phoff、e_phentsize、e_phnum：描述Program Header Table的偏移、大小、结构。
* 4，e_shoff、e_shentsize、e_shnum：描述Section Header Table的偏移、大小、结构。
* 5，e_shstrndx：这一项描述的是字符串表在 Section Header Table 中的索引，值25表示的是 Section Header Table 中第25项是字符串表（String Table）。

### Section Header Table

&emsp;&emsp;一个ELF文件中到底有哪些具体的 sections，由包含在这个ELF文件中的 section head table(SHT) 决定。在SHT中，针对每一个section，都设置有一个条目（entry），用来描述对应的这个section，其内容主要包括该 section 的名称、类型、大小以及在整个ELF文件中的字节偏移位置等等。我们也可以在TISCv1.2规范中找到SHT表中条目的C结构定义：

![20160521111301410.png](/img/20160521111301410.png)

解析 android_server 可执行ELF文件，我们可以看到 Section Header Table 中确实有23（17h (16进制表示)）个条目，且索引为22（16h(16进制表示)）确实为 section header section string table。

![20160521111322394.png](/img/20160521111322394.png)

打开条目，我们可以看到每个 entry 的具体字段，与上图的 Elf32_Shdr 结构一致。

![20160521111341347.png](/img/20160521111341347.png)

&emsp;&emsp;需要注意的是，sh_name 值实际上是 .shstrtab 中的索引，该string table中存储着所有section的名字。下图中蓝色部分是.shstrtab的数据，我们可以看到，sh_name实际上是从索引1开始的”.shstrtab”字符串，因此这里的sh_name值为1h。

![20160521111400597.png](/img/20160521111400597.png)

### Section

下面我们分析一些so文件中重要的Section，包括符号表、重定位表、GOT表等。

**-符号表(.dynsym)**

&emsp;&emsp;符号表包含用来定位、重定位程序中符号定义和引用的信息，简单的理解就是符号表记录了该文件中的所有符号，所谓的符号就是经过修饰了的函数名或者变量名，不同的编译器有不同的修饰规则。例如符号_ZL15global_static_a，就是由global_static_a变量名经过修饰而来。

符号表项的格式如下：

```c
typedef struct {  
     Elf32_Word st_name;        //符号表项名称。如果该值非0，则表示符号名的字
                                   //符串表索引(offset)，否则符号表项没有名称。
     Elf32_Addr st_value;       //符号的取值。依赖于具体的上下文，可能是一个绝对值、一个地址等等。
     Elf32_Word st_size;        //符号的尺寸大小。例如一个数据对象的大小是对象中包含的字节数。
     unsigned char st_info;     //符号的类型和绑定属性。
     unsigned char st_other;    //未定义。
     Elf32_Half st_shndx;        //每个符号表项都以和其他节区的关系的方式给出定义。
　　　　　　　　　　　　　         //此成员给出相关的节区头部表索引。
} Elf32_sym;
```

通过 010Editor 解析出的符号表 .dynsym 的 section header 表项：

![20160521111537942.png](/img/20160521111537942.png)

符号表的具体内容：

![20160521111608073.png](/img/20160521111608073.png)

**-字符串表（.dynstr）**

字符串表中存放着所有符号的名称字符串。

字符串表的section header表项：

![20160521111628840.png](/img/20160521111628840.png)

再看一下下图中字符串表的具体内容，我们可以看出，.dynstr和.shstrtab 结构完全相同，不过一个存储的是符号名称的字符串，而另一个是Section 名称的字符串。

![20160521111652559.png](/img/20160521111652559.png)

**-重定位表**

&emsp;&emsp;重定位表在ELF文件中扮演很重要的角色，首先我们得理解重定位的概念，程序从代码到可执行文件这个过程中，要经历编译器，汇编器和链接器对代码的处理。然而编译器和汇编器通常为每个文件创建程序地址从0开始的目标代码，但是几乎没有计算机会允许从地址0加载你的程序。如果一个程序是由多个子程序组成的，那么所有的子程序必需要加载到互不重叠的地址上。重定位就是为程序不同部分分配加载地址，调整程序中的数据和代码以反映所分配地址的过程。简单的言之，则是将程序中的各个部分映射到合理的地址上来。

&emsp;&emsp;换句话来说，重定位是将符号引用与符号定义进行连接的过程。例如，当程序调用了一个函数时，相关的调用指令必须把控制传输到适当的目标执行地址。
具体来说，就是把符号的value进行重新定位。

可重定位文件必须包含如何修改其节区内容的信息，从而允许可执行文件和共享目标文件保存进程的程序映象的正确信息。这就是重定位表项做的工作。重定位表项的格式如下： 

```c
typedef struct {  
    Elf32_Addr r_offset;     //重定位动作所适用的位置（受影响的存储单位的第一个字节的偏移或者虚拟地址）
    Elf32_Word r_info;       //要进行重定位的符号表索引，以及将实施的重定位类型（哪些位需要修改，以及如何计算它们的取值）
                                         //其中 .rel.dyn 重定位类型一般为R_386_GLOB_DAT和R_386_COPY；.rel.plt为R_386_JUMP_SLOT
} Elf32_Rel; 
 ```
 ```
typedef struct {  
    Elf32_Addr r_offset;  
    Elf32_Word r_info;  
    Elf32_Word r_addend;
 } Elf32_Rela; 
```

对 r_info 成员使用 ELF32_R_TYPE 宏运算可得到重定位类型，使用 ELF32_R_SYM 宏运算可得到符号在符号表里的索引值。 三种宏的具体定义如下：

```c
#define ELF32_R_SYM(i) ((i)>>8) 
#define ELF32_R_TYPE(i) ((unsigned char)(i)) 
#define ELF32_R_INFO(s, t) (((s)
```

重定位表中的内容:

![20160521111838248.png](/img/20160521111838248.png)

以下是.rel.plt表的具体内容：

![20160521111856928.png](/img/20160521111856928.png)

我们可以看到，每8个字节(s_entsize)一个表项。第一个表项中的r_offset值为0xc7660，r_info为0xa16。其中r_offset指向下图中GOT表中第一项__imp_clock_gettime外部函数地址。那么我们如何利用r_offset值来找到其对应的符号呢？如上所述，进行 ELF32_R_SYM宏运算实际上就是将r_info右移8位，0xa16右移8位得到0xa，因此这就是其在符号表中的索引。 

![20160521111921663.png](/img/20160521111921663.png)

从下图中可以看见符号表的s_entsize值为10h，即16个字节每条目。因此我们可以找到其索引为0xa的条目的st_name值为0x9ea。那么怎么证明我们确实找到的是clock_gettime函数的符号呢？我们再来看一下st_name值是不是正确的。

![20160521111941390.png](/img/20160521111941390.png)

st_name值表示的是符号名字符串中的第一个字符在字符串表中的偏移量，因此我们用0x9ea加上符号表的起始位置(0x7548)就能得到该字符串在‭0x7F32位置。如下图所示:

![20160521112219618.png](/img/20160521112219618.png)

**-常见的重定位表类型：**

* .rel.text：重定位的地方在.text段内，以offset指定具体要定位位置。在链接时候由链接器完成。.rel.text属于普通重定位辅助段 ,他由编译器编译产生，存在于obj文件内。连接器连接时，他用于最终可执行文件或者动态库的重定位。通过它修改原obj文件的.text段后，合并到最终可执行文件或者动态文件的.text段。其类型一般为R_386_32和R_386_PC32。

* .rel.dyn：重定位的地方在.got段内。主要是针对外部数据变量符号。例如全局数据。重定位在程序运行时定位，一般是在.init段内。定位过程：获得符号对应value后，根据rel.dyn表中对应的offset，修改.got表对应位置的value。另外，.rel.dyn 含义是指和dyn有关，一般是指在程序运行时候，动态加载。区别于rel.plt，rel.plt是指和plt相关，具体是指在某个函数被调用时候加载。我个人理解这个Section的作用是，在重定位过程中，动态链接器根据r_offset找到.got对应表项，来完成对.got表项值的修改。

* .rel.dyn和.rel.plt是动态定位辅助段。由连接器产生，存在于可执行文件或者动态库文件内。借助这两个辅助段可以动态修改对应.got和.got.plt段，从而实现运行时重定位。

* .rel.plt：重定位的地方在.got.plt段内（注意也是.got内,具体区分而已）。 主要是针对外部函数符号。一般是函数首次被调用时候重定位。首次调用时会重定位函数地址，把最终函数地址放到.got内，以后读取该.got就直接得到最终函数地址。我个人理解这个Section的作用是，在重定位过程中，动态链接器根据r_offset找到.got对应表项，来完成对.got表项值的修改。

* .plt段（过程链接表）：所有外部函数调用都是经过一个对应桩函数，这些桩函数都在.plt段内。具体调用外部函数过程是：
调用对应桩函数—>桩函数取出.got表表内地址—>然后跳转到这个地址.如果是第一次,这个跳转地址默认是桩函数本身跳转处地址的下一个指令地址(目的是通过桩函数统一集中取地址和加载地址),后续接着把对应函数的真实地址加载进来放到.got表对应处,同时跳转执行该地址指令.以后桩函数从.got取得地址都是真实函数地址了。
下图是.plt某表项，它包含了取.got表地址和跳转执行两条指令。

![20160521112646041.png](/img/20160521112646041.png)

* .got（全局偏移表）

### Program Header Table

&emsp;&emsp;程序头部（Program Header）描述与程序执行直接相关的目标文件结构信息。用来在文件中定位各个段的映像。同时包含其他一些用来为程序创建映像所必须的信息。
可执行文件或者共享目标文件的程序头部是一个结构数组，每个结构描述了一个段或者系统准备程序执行所必须的其他信息。目标文件的 "段" 包含一个或者多个 "节区"，也就是 "段内容（Segment Contents）"。程序头部仅对可执行文件和共享目标文件有意义。

程序头部的数据结构如下：

```c
typedef struct {  
    Elf32_Word p_type;           //此数组元素描述的段的类型，或者如何解释此数组元素的信息。 
    Elf32_Off  p_offset;           //此成员给出从文件头到该段第一个字节的偏移
    Elf32_Addr p_vaddr;         //此成员给出段的第一个字节将被放到内存中的虚拟地址
    Elf32_Addr p_paddr;        //此成员仅用于与物理地址相关的系统中。System V忽略所有应用程序的物理地址信息。
    Elf32_Word p_filesz;         //此成员给出段在文件映像中所占的字节数。可以为0。
    Elf32_Word p_memsz;     //此成员给出段在内存映像中占用的字节数。可以为0。
    Elf32_Word p_flags;         //此成员给出与段相关的标志。
    Elf32_Word p_align;        //此成员给出段在文件中和内存中如何对齐。
} Elf32_phdr;
```

我们看到，以下两个工具确实是照此格式解析的:

![20160521112705213.png](/img/20160521112705213.png)

![20160521112720713.png](/img/20160521112720713.png)

参考：

[ELF文件格式解析](https://blog.csdn.net/mergerly/article/details/94585901)

[ELF文件详解—初步认识](https://blog.csdn.net/daide2012/article/details/73065204)

[ELF 格式详解（一）](https://blog.virbox.com/?p=119)

[ELF文件格式解析](https://blog.csdn.net/feglass/article/details/51469511?spm=1001.2014.3001.5501)

[Linux进程地址空间学习(二)](http://www.choudan.net/2013/10/25/Linux%E8%BF%9B%E7%A8%8B%E5%9C%B0%E5%9D%80%E7%A9%BA%E9%97%B4%E5%AD%A6%E4%B9%A0%28%E4%BA%8C%29.html)