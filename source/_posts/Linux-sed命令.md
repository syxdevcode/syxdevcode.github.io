---
title: Linux sed命令
date: 2021-03-29 10:18:17
tags:
- Linux
- CentOS7
- Linux优化
- Linux基础命令
categories:
- Linux基础命令
---

Linux sed 命令是利用脚本来处理文本文件。sed 命令采用的是流编辑模式，最明显的特点是，在 sed 处理数据之前，需要预先提供一组规则，sed 会按照此规则来编辑数据。默认情况下，sed 会在所有的脚本指定执行完毕后，会自动输出处理后的内容，而该选项会屏蔽启动输出，需使用 print 命令来完成输出。

默认情况下 sed 并不会修改原始文件，这里被删除的行只是从 sed 的输出中消失了，原始文件没做任何改变。

Sed 主要用来自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序等。

sed 会根据脚本命令来处理文本文件中的数据，这些命令要么从命令行中输入，要么存储在一个文本文件中，此命令执行数据的顺序如下：

* 1，每次仅读取一行内容；
* 2，根据提供的规则命令匹配并修改数据。注意，sed 默认不会直接修改源文件数据，而是会将数据复制到缓冲区中，修改也仅限于缓冲区中的数据；
* 3，将执行结果输出。

当一行数据匹配完成后，它会继续读取下一行数据，并重复这个过程，直到将文件中所有数据处理完毕。

语法 `sed [-hnV][-e<script>][-f<script文件>][文本文件]`

参数说明：

| 选项   | 含义  |
| :-----: | :---- |
| -e 脚本命令 | 该选项会将其后跟的脚本命令添加到已有的命令中。 |
| -f 脚本命令文件 | 该选项会将其后文件中的脚本命令添加到已有的命令中。 |
| -n或--quiet或--silent | 仅显示script处理后的结果，默认情况下，sed 会在所有的脚本指定执行完毕后，会自动输出处理后的内容，而该选项会屏蔽启动输出，需使用 print 命令来完成输出。 |
| -h或--help |  显示帮助。 |
| -V或--version |  显示版本信息。 |

动作说明

a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ：删除， d 后面通常不接任何字符；
i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p ：打印， 将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行
s ：取代， 通常搭配正规表示法，例如 1,20s/old/new/g

## sed脚本命令

### sed s 替换脚本命令

此命令的基本格式为：

```sh
[address]s/pattern/replacement/flags
```
address 表示指定要操作的具体行，pattern 指的是需要替换的内容，replacement 指的是要替换的新内容。

sed s命令flags标记及功能:

| flags 标记   | 功能  |
| :-----: | :---- |
|n | 	1~512 之间的数字，表示指定要替换的字符串出现第几次时才进行替换，例如，一行中有 3 个 A，但用户只想替换第二个 A，这是就用到这个标记； | 
|g | 	对数据中所有匹配到的内容进行替换，如果没有 g，则只会在第一次匹配成功时做替换操作。例如，一行数据中有 3 个 A，则只会替换第一个 A； | 
|p | 	会打印与替换命令中指定的模式匹配的行。此标记通常与 -n 选项一起使用。 | 
|w file | 	将缓冲区中的内容写到指定的 file 文件中； | 
|& | 	用正则表达式匹配的内容进行替换； | 
|\n | 	匹配第 n 个子串，该子串之前在 pattern 中用 `\(\)` 指定。 | 
|\ | 	转义（转义替换部分包含：& ，\ 等）。 | 

**实例**

```sh
# sed 编辑器只替换每行中第 2 次出现的匹配模式
sed 's/test/trial/2' data4.txt

# 用 g 标记，在新文件替换所有匹配的字符串
sed 's/test/trial/g' data4.txt

# -n 选项会禁止 sed 输出，p 标记会输出修改过的行，
# 二者匹配使用的效果就是只输出被替换命令修改过的行
sed -n 's/test/trial/p' data5.txt

# w 标记会将匹配后的结果保存到指定文件中
sed 's/test/trial/w test.txt' data5.txt

# 替换类似文件路径的字符串，需要将路径中的正斜线进行转义
sed 's/\/bin\/bash/\/bin\/csh/' /etc/passwd
```

### sed d 替换脚本命令

基本格式为：`[address]d`

```sh
# 删除所有内容
sed 'd' data1.txt

# 通过行号指定，比如删除 data6.txt 文件内容中的第 3 行：
sed '3d' data6.txt

# 通过特定行区间指定，比如删除 data6.txt 文件内容中的第 2、3行
sed '2,3d' data6.txt

# 使用两个文本模式来删除某个区间内的行
# 指定的第一个模式会“打开”行删除功能，第二个模式会“关闭”行删除功能，
# 因此，sed 会删除两个指定行之间的所有行（包括指定的行）
# 删除第 1~3 行的文本数据
sed '/1/,/3/d' data6.txt

# 通过特殊的文件结尾字符，比如删除 data6.txt 文件内容中第 3 行开始的所有的内容
sed '3,$d' data6.txt
```

### sed a 和 i 脚本命令

a 命令表示在指定行的后面附加一行，i 命令表示在指定行的前面插入一行

基本格式：`[address]a（或 i）\新文本内容`

```sh
# 将一个新行插入到数据流第三行前
sed '3i\
> This is an inserted line.' data6.txt

# 将一个新行附加到数据流中第三行后
sed '3a\
> This is an appended line.' data6.txt

# 如果想将一个多行数据添加到数据流中，只需对要插入或附加的文本中的每一行末尾（除最后一行）添加反斜线即可
 sed '1i\
> This is one line of new text.\
> This is another line of new text.' data6.txt
```

### sed c 替换脚本命令

c 命令表示将指定行中的所有内容，替换成该选项后面的字符串。

基本格式为：`[address]c\用于替换的新文本`

```sh
sed '3c\
> This is a changed line of text.' data6.txt

# 或者
sed '/number 3/c\
> This is a changed line of text.' data6.txt
```

### sed y 转换脚本命令

y 转换命令是唯一可以处理单个字符的 sed 脚本命令

基本格式：`[address]y/inchars/outchars/`

转换命令会对 inchars 和 outchars 值进行一对一的映射，即 inchars 中的第一个字符会被转换为 outchars 中的第一个字符，第二个字符会被转换成 outchars 中的第二个字符...这个映射过程会一直持续到处理完指定字符。

如果 inchars 和 outchars 的长度不同，则 sed 会产生一条错误消息。

```sh
sed 'y/123/789/' data8.txt

# 转换命令是一个全局命令，它会文本行中找到的所有指定字符自动进行转换，
# 而不会考虑它们出现的位置
echo "This 1 is a test of 1 try." | sed 'y/123/456/'
This 4 is a test of 4 try.
```

### sed p 打印脚本命令

p 命令表示搜索符号条件的行，并输出该行的内容

基本格式为：`[address]p`

```sh
# p 命令常见的用法是打印包含匹配文本模式的行
sed -n '/number 3/p' data6.txt
This is line number 3.

# 查找包含数字 3 的行，然后执行两条命令。
# 首先，脚本用 p 命令来打印出原始行；然后它用 s 命令替换文本，并用 p 标记打印出替换结果。
# 输出同时显示了原来的行文本和新的行文本。
sed -n '/3/{
> p
> s/line/test/p
> }' data6.txt
This is line number 3.
This is test number 3.
```

### sed w 脚本命令

w 命令用来将文本中指定行的内容写入文件中

基本格式如下：`[address]w filename`

filename 表示文件名，可以使用相对路径或绝对路径，运行 sed 命令的用户都必须有文件的写权限。

```sh
# 将 data6.txt 1，2行打印到 test.txt
sed '1,2w test.txt' data6.txt

# 如果不想让行直接输出，可以用 -n 选项
sed -n '/Browncoat/w Browncoats.txt' data11.txt
```

### sed r 脚本命令

r 命令用于将一个独立文件的数据插入到当前数据流的指定位置，

基本格式为：`[address]r filename`

```sh
# sed 命令会将 filename 文件中的内容插入到 address 指定行的后面
[root@localhost ~]# cat data12.txt
This is an added line.
This is the second added line.
[root@localhost ~]# sed '3r data12.txt' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is an added line.
This is the second added line.
This is line number 4.

# 将指定文件中的数据插入到数据流的末尾，可以使用 $ 地址符
[root@localhost ~]# sed '$r data12.txt' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
This is an added line.
This is the second added line.
```

### sed q 退出脚本命令

q 命令的作用是使 sed 命令在第一次匹配任务结束后，退出 sed 程序，不再进行对后续数据的处理。

```sh
#sed 命令在打印输出第 2 行之后，就停止
[root@localhost ~]# sed '2q' test.txt
This is line number 1.
This is line number 2.

# 使用 q 命令之后，sed 命令会在匹配到 number 1 时，将其替换成 number 0，然后直接退出。
[root@localhost ~]# sed '/number 1/{ s/number 1/number 0/;q; }' test.txt
This is line number 0.
```

## sed 脚本命令的寻址方式

默认情况下，sed 命令会作用于文本数据的所有行。如果只想将命令作用于特定行或某些行，则必须写明 address 部分，表示的方法有以下 2 种：

* 1，以数字形式指定行区间；
* 2，用文本模式指定具体行区间。

两种形式都可以使用如下这 2 种格式，分别是：

```sh
[address]脚本命令

或者

address {
    多个脚本命令
}
```

**以数字形式指定行区间**

当使用数字方式的行寻址时，可以用行在文本流中的行位置来引用。sed 会将文本流中的第一行编号为 1，然后继续按顺序为接下来的行分配行号。

在脚本命令中，指定的地址可以是单个行号，或是用起始行号、逗号以及结尾行号指定的一定区间范围内的行。

```sh
# sed 只修改地址指定的第二行的文本
[root@localhost ~]#sed '2s/dog/cat/' data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy dog

# sed 只修改地址指定的第二,三行的文本
[root@localhost ~]# sed '2,3s/dog/cat/' data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy dog

# 如果想将命令作用到文本中从某行开始的所有行，可以用特殊地址——美元符（$）
[root@localhost ~]# sed '2,$s/dog/cat/' data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy cat
```

**用文本模式指定行区间**

sed 允许指定文本模式来过滤出命令要作用的行，格式如下：

```sh
/pattern/command
```

注意，必须用正斜线将要指定的 pattern 封起来，sed 会将该命令作用到包含指定文本模式的行上。

如果你想只修改用户 demo 的默认 shell，可以使用 sed 命令，执行命令如下：

```sh
[root@localhost ~]# grep demo /etc/passwd
demo:x:502:502::/home/Samantha:/bin/bash
[root@localhost ~]# sed '/demo/s/bash/csh/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
...
demo:x:502:502::/home/demo:/bin/csh
...
```

虽然使用固定文本模式能帮你过滤出特定的值，就跟上面这个用户名的例子一样，但其作用难免有限，因此，sed 允许在文本模式使用正则表达式指明作用的具体行。正则表达式允许创建高级文本模式匹配表达式来匹配各种数据。这些表达式结合了一系列通配符、特殊字符以及固定文本字符来生成能够匹配几乎任何形式文本的简练模式。




参考：

[Linux sed命令完全攻略](http://c.biancheng.net/view/4028.html)

[Linux sed命令高级用法精讲](http://c.biancheng.net/view/4056.html)
