---
title: Lua简单学习笔记
date: 2020-10-19 14:22:35
tags:
- Lua
- Linux
categories: 
- Lua
---

## 简介

Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

## Lua 特性

* 轻量级: 它用标准C语言编写并以源代码形式开放，编译后仅仅一百余K，可以很方便的嵌入别的程序里。
* 可扩展: Lua提供了非常易于使用的扩展接口和机制：由宿主语言(通常是C或C++)提供这些功能，Lua可以使用它们，就像是本来就内置的功能一样。
* 其它特性:
支持面向过程(procedure-oriented)编程和函数式编程(functional programming)；
自动内存管理；只提供了一种通用类型的表（table），用它可以实现数组，哈希表，集合，对象；
语言内置模式匹配；闭包(closure)；函数也可以看做一个值；提供多线程（协同进程，并非操作系统所支持的线程）支持；
通过闭包和table可以很方便地支持面向对象编程所需要的一些关键机制，比如数据抽象，虚函数，继承和重载等。

## Lua 应用场景

* 游戏开发
* 独立应用脚本
* Web 应用脚本
* 扩展和数据库插件如：MySQL Proxy 和 MySQL WorkBench
* 安全系统，如入侵检测系统

## Lua 环境安装

参考：

[Lua 环境安装](https://www.runoob.com/lua/lua-environment.html)

## Lua 基本语法

### 交互式编程

```sh
# 启用
lua -i 或 lua

#命令行输入命令
print("Hello World！")
Hello World！# 输出
```

### 脚本式编程

将 Lua 程序代码保存到一个以 lua 结尾的文件，并执行，该模式称为脚本式编程

hello.lua

```sh
print("Hello World！")
print("test1")
```

执行：

```sh
$ lua hello.lua
```

也可以将代码修改为如下形式来执行脚本（在开头添加：`#!/usr/local/bin/lua`）,指定了 Lua 的解释器 `/usr/local/bin directory`。加上 # 号标记解释器会忽略它。接下来我们为脚本添加可执行权限，并执行：

```sh
#!/usr/local/bin/lua

print("Hello World！")
print("www.runoob.com")
```

执行：

```sh
./hello.lua
```

### 注释

单行注释: 

两个减号是单行注释: --

多行注释: 

```sh
--[[
 多行注释
 多行注释
 --]]
```

多行注释推荐使用 --[=[注释内容]=]，这样可以避免遇到 table[table[idx]] 时就将多行注释结束了。

注意：多行注释加 - 取消注释中间代码可以继续运行，单行注释没有此功能。

### 标示符

标示符以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头后加上 0 个或多个字母，下划线，数字（0 到 9）。

最好不要使用下划线加大写字母的标示符，因为Lua的保留字也是这样的。

Lua 不允许使用特殊字符如 @, $, 和 % 来定义标示符。 Lua 是一个区分大小写的编程语言。

### 关键词

```lua
and	break	do	else
elseif	end	false	for
function	if	in	local
nil	not	or	repeat
return	then	true	until
while	goto
```

一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量。

### 全局变量

在默认情况下，变量总是认为是全局的。

全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil。

如果你想删除一个全局变量，只需要将变量赋值为nil。当且仅当一个变量不等于nil时，这个变量即存在。

```lua
print(b)
nil
> b=10
> print(b)
10
```

## 数据类型

Lua 是动态类型语言，变量不要类型定义,只需要为变量赋值。 值可以存储在变量中，作为参数传递或结果返回。

Lua 中有 8 个基本类型分别为：nil、boolean、number、string、userdata、function、thread 和 table。

![微信截图_20201019145429.png](/img/微信截图_20201019145429.png)

```lua
print(type("Hello world"))      --> string
print(type(10.4*3))             --> number
print(type(print))              --> function
print(type(type))               --> function
print(type(true))               --> boolean
print(type(nil))                --> nil
print(type(type(X)))            --> string
```

### nil（空）

```lua
> print(type(a))
nil
```

nil 作比较时应该加上双引号 "：

```lua
> type(X)
nil
> type(X)==nil
false
> type(X)=="nil"
true

-- 没有使用type(变量)这个形式，就不需要加引号
print(x == nil) --结果为true
print(x == "nil")--结果为false
```

type(X)==nil 结果为 false 的原因是因为 type(type(X))==string。

### boolean（布尔）

boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是 false，其他的都为 true，数字 0 也是 true:

### number（数字）

Lua 默认只有一种 number 类型 -- double（双精度）类型（默认类型可以修改 luaconf.h 里的定义），以下几种写法都被看作是 number 类型：

```lua
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```

### string（字符串）

字符串由一对双引号或单引号来表示。

```lua
string1 = "this is string1"
string2 = 'this is string2'
```

也可以用 2 个方括号 "[[]]" 来表示"一块"字符串。

```lua
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```

对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字:

```lua
> print("2" + 6)
8.0
> print("2" + "6")
8.0
> print("2 + 6")
2 + 6
> print("-2e2" * "6")
-1200.0
```

使用 # 来计算字符串的长度，放在字符串前面，如下实例：

```lua
> len = "www.runoob.com"
> print(#len)
14
> print(#"www.runoob.com")
14
```

运行时，Lua会自动在string和numbers之间自动进行类型转换，当一个字符串使用算术操作符时， string 就会被转成数字。反过来，当 Lua 期望一个 string 而碰到数字时，会将数字转成 string。

### table（表）

在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表:

```lua
-- 创建一个空的 table
local tbl1 = {}
 
-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}
```

Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串。

```lua
-- table_test.lua 脚本文件
a = {}
a["key"] = "value"
key = 10
a[key] = 22
a[key] = a[key] + 11
for k, v in pairs(a) do
    print(k .. " : " .. v)
end
```

脚本执行结果为：

```lua
$ lua table_test.lua 
key : value
10 : 33
```

不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始。

```lua
-- table_test2.lua 脚本文件
local tbl = {"apple", "pear", "orange", "grape"}
for key, val in pairs(tbl) do
    print("Key", key)
end
```

脚本执行结果为：

```lua
$ lua table_test2.lua 
Key    1
Key    2
Key    3
Key    4
```

table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。

```lua
-- table_test3.lua 脚本文件
a3 = {}
for i = 1, 10 do
    a3[i] = i
end
a3["key"] = "val"
print(a3["key"])
print(a3["none"])
```

脚本执行结果为：

```lua
$ lua table_test3.lua 
val
nil
```

### function（函数）

在 Lua 中，函数是被看作是"第一类值（First-Class Value）"，函数可以存在变量里:

```lua
-- function_test.lua 脚本文件
function factorial1(n)
    if n == 0 then
        return 1
    else
        return n * factorial1(n - 1)
    end
end
print(factorial1(5))
factorial2 = factorial1
print(factorial2(5))
```

脚本执行结果为：

```lua
$ lua function_test.lua 
120
120
```

function 可以以匿名函数（anonymous function）的方式通过参数传递:

```lua
-- function_test2.lua 脚本文件
function testFun(tab,fun)
        for k ,v in pairs(tab) do
                print(fun(k,v));
        end
end


tab={key1="val1",key2="val2"};
testFun(tab,
function(key,val)--匿名函数
        return key.."="..val;
end
);
```

脚本执行结果为：

```lua
$ lua function_test2.lua 
key1 = val1
key2 = val2
```

### thread（线程）

在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。

### userdata（自定义类型）

userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。

## Lua 变量

变量在使用前，需要在代码中进行声明，即创建该变量。

编译程序执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。

Lua 变量有三种类型：全局变量、局部变量、表中的域。

Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显式声明为局部变量。

局部变量的作用域为从声明位置开始到所在语句块结束。

变量的默认值均为 nil。

尽可能的使用局部变量，有两个好处：

* 避免命名冲突。
* 访问局部变量的速度比全局变量更快。

```lua
-- test.lua 文件脚本
a = 5               -- 全局变量
local b = 5         -- 局部变量

function joke()
    c = 5           -- 全局变量
    local d = 6     -- 局部变量
end

joke()
print(c,d)          --> 5 nil

do
    local a = 6     -- 局部变量
    b = 6           -- 对局部变量重新赋值
    print(a,b);     --> 6 6
end

print(a,b)      --> 5 6
```

### 赋值语句

```lua
a = "hello" .. "world"
t.n = t.n + 1
```

Lua 可以对多个变量同时赋值，变量列表和值列表的各个元素用逗号分开，赋值语句右边的值会依次赋给左边的变量。

```lua
a, b = 10, 2*x       <-->       a=10; b=2*x
```

遇到赋值语句Lua会先计算右边所有的值然后再执行赋值操作，所以我们可以这样进行交换变量的值：

```lua
x, y = y, x                     -- swap 'x' for 'y'
a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'
```

当变量个数和值的个数不一致时，Lua会一直以变量个数为基础采取以下策略：

```lua
a. 变量个数 > 值的个数             按变量个数补足nil
b. 变量个数 < 值的个数             多余的值会被忽略
```

```lua
a, b, c = 0, 1
print(a,b,c)             --> 0   1   nil
 
a, b = a+1, b+1, b+2     -- value of b+2 is ignored
print(a,b)               --> 1   2
 
a, b, c = 0
print(a,b,c)             --> 0   nil   nil
```

Lua 对多个变量同时赋值，不会进行变量传递，仅做值传递：

```lua
a, b = 0, 1
a, b = a+1, a+1
print(a,b)               --> 1   1
a, b = 0, 1
a, b = b+1, b+1
print(a,b)               --> 2   2
a, b = 0, 1
a = a+1
b = a+1
print(a,b)               --> 1   2
```

### 索引

对 table 的索引使用方括号 []。Lua 也提供了 `.` 操作。

```lua
t[i]
t.i                 -- 当索引为字符串类型时的一种简化写法
gettable_event(t,i) -- 采用索引访问本质上是一个类似这样的函数调用
```

```lua
> site = {}
> site["key"] = "www.runoob.com"
> print(site["key"])
www.runoob.com
> print(site.key)
www.runoob.com
```

## 循环

### while 循环

while 循环语法：

```lua
while(condition)
do
   statements
end
```

### for 循环

for语句有两大类：：

数值for循环

泛型for循环

#### 数值for循环

```lua
for var=exp1,exp2,exp3 do  
    <执行体>  
end  
```

var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。exp3 是可选的，如果不指定，默认为1。

for的三个表达式在循环开始前一次性求值，以后不再进行求值。比如上面的f(x)只会在循环开始前执行一次，其结果用在后面的循环中。

```lua
#!/usr/local/bin/lua  
function f(x)  
    print("function")  
    return x*2  
end  
for i=1,f(5) do print(i)  
end
```

输出：

```txt
function
1
2
3
4
5
6
7
8
9
10
```

#### 泛型for循环

泛型 for 循环通过一个迭代器函数来遍历所有值，类似 java 中的 foreach 语句。迭代的下标是从1开始。

Lua 编程语言中泛型 for 循环语法格式:

```lua
--打印数组a的所有值  
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end
```

i是数组索引值，v是对应索引的数组元素值。ipairs是Lua提供的一个迭代器函数，用来迭代数组。

for 循环中，循环的索引 i 为外部索引，修改循环语句中的内部索引 i，不会影响循环次数:

```lua
for i=1,10 do
    i = 10
    print("one time,i:"..i)
end
```

仍然循环 10 次，只是 i 的值被修改了。

在 lua 中 pairs 与 ipairs 两个迭代器的用法相近，但有一点是不一样的：

pairs 能迭代所有键值对。

ipairs 可以想象成 int+pairs，只会迭代键为数字的键值对，只能遍历所有数组下标的值。
如果 ipairs 在迭代过程中是会直接跳过所有手动设定key值的变量。

pairs 可以遍历表中所有的 key，并且除了迭代器本身以及遍历表本身还可以返回 nil;

但是 ipairs 则不能返回nil,只能返回数字0，如果遇到nil则退出。它只能遍历到表中出现的第一个不是整数的key。

例如：

```lua
tab = {1,2,a= nil,"d"}
for i,v in ipairs(tab) do
    print(i,v)
end
```

输出结果为：

```
1  1
2  2
3  d
```

这里是直接跳过了 a=nil 这个变量

第二，ipairs 在迭代过程中如果遇到nil时会直接停止。

```lua
tab = {1,2,a= nil,nil,"d"}
for i,v in ipairs(tab) do
    print(i,v)
end
```

输出结果为：

```
1  1
2  2
```

这里会在遇到 nil 的时候直接跳出循环。

### repeat...until 循环

repeat...until 循环的条件语句在当前循环结束后判断。

```lua
repeat
   statements
until( condition )
```

```lua
--[ 变量定义 --]
a = 10
--[ 执行循环 --]
repeat
   print("a的值为:", a)
   a = a + 1
until( a > 15 )
```

输出：

```
a的值为:    10
a的值为:    11
a的值为:    12
a的值为:    13
a的值为:    14
a的值为:    15
```

### 循环控制语句

#### break 语句

```lua
--[ 定义变量 --]
a = 10

--[ while 循环 --]
while( a < 20 )
do
   print("a 的值为:", a)
   a=a+1
   if( a > 15)
   then
      --[ 使用 break 语句终止循环 --]
      break
   end
end
```

#### goto 语句

Lua 语言中的 goto 语句允许将控制流程无条件地转到被标记的语句处。

语法格式如下所示：

```lua
goto Label
```

Label 的格式为：

```lua
:: Label ::
```

以下实例在判断语句中使用 goto：

```lua
local a = 1
::label:: print("--- goto label ---")

a = a+1
if a < 3 then
    goto label   -- a 小于 3 的时候跳转到标签 label
end
```

输出结果为：

```
--- goto label ---
--- goto label ---
```

从输出结果可以看出，多输出了一次 --- goto label ---。

以下实例演示了可以在 lable 中设置多个语句：

```lua
i = 0
::s1:: do
  print(i)
  i = i+1
end
if i>3 then
  os.exit()   -- i 大于 3 时退出
end
goto s1
```

输出结果为：

```
0
1
2
3
```

#### continue功能

```lua
for i=1, 3 do
    if i <= 2 then
        print(i, "yes continue")
        goto continue
    end
    print(i, " no continue")
    ::continue::
    print([[i'm end]])
end
```

输出：

```
1   yes continue
i'm end
2   yes continue
i'm end
3    no continue
i'm end
```

## 流程控制

### if 语句

```lua
--[ 定义变量 --]
a = 10;

--[ 使用 if 语句 --]
if( a < 20 )
then
   --[ if 条件为 true 时打印以下信息 --]
   print("a 小于 20" );
end
print("a 的值为:", a);
```

### if...else 语句

```lua
--[ 定义变量 --]
a = 100;
--[ 检查条件 --]
if( a < 20 )
then
   --[ if 条件为 true 时执行该语句块 --]
   print("a 小于 20" )
else
   --[ if 条件为 false 时执行该语句块 --]
   print("a 大于 20" )
end
print("a 的值为 :", a)
```

### if...elseif...else 语句

```lua
--[ 定义变量 --]
a = 100

--[ 检查布尔条件 --]
if( a == 10 )
then
   --[ 如果条件为 true 打印以下信息 --]
   print("a 的值为 10" )
elseif( a == 20 )
then  
   --[ if else if 条件为 true 时打印以下信息 --]
   print("a 的值为 20" )
elseif( a == 30 )
then
   --[ if else if condition 条件为 true 时打印以下信息 --]
   print("a 的值为 30" )
else
   --[ 以上条件语句没有一个为 true 时打印以下信息 --]
   print("没有匹配 a 的值" )
end
print("a 的真实值为: ", a )
```

## 函数

在Lua中，函数是对语句和表达式进行抽象的主要方法。既可以用来处理一些特殊的工作，也可以用来计算一些值。

Lua 提供了许多的内建函数，你可以很方便的在程序中调用它们，如 print() 函数可以将传入的参数打印在控制台上。

Lua 函数主要有两种用途：

* 1.完成指定的任务，这种情况下函数作为调用语句使用；
* 2.计算并返回值，这种情况下函数作为赋值语句的表达式使用。

### 函数定义

```lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```

解析：

* optional_function_scope: 该参数是可选的制定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local。

* function_name: 指定函数名称。

* argument1, argument2, argument3..., argumentn: 函数参数，多个参数以逗号隔开，函数也可以不带参数。

* function_body: 函数体，函数中需要执行的代码语句块。

* result_params_comma_separated: 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。

将函数作为参数传递给函数，如下实例：

```lua
myprint = function(param)
   print("这是打印函数 -   ##",param,"##")
end

function add(num1,num2,functionPrint)
   result = num1 + num2
   -- 调用传递的函数参数
   functionPrint(result)
end
myprint(10)
-- myprint 函数作为参数传递
add(2,5,myprint)
```

以上代码执行结果为：

```
这是打印函数 -   ##    10    ##
这是打印函数 -   ##    7    ##
```

### 多返回值

Lua函数可以返回多个结果值，比如string.find，其返回匹配串"开始和结束的下标"（如果不存在匹配串返回nil）。

```lua
> s, e = string.find("www.runoob.com", "runoob") 
> print(s, e)
5    10
```

Lua函数中，在return后列出要返回的值的列表即可返回多值，如：

```lua
function maximum (a)
    local mi = 1             -- 最大值索引
    local m = a[mi]          -- 最大值
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end

print(maximum({8,10,23,12,5}))
```

以上代码执行结果为：

```
23    3
```

### 可变参数

Lua 函数可以接受可变数目的参数，和 C 语言类似，在函数参数列表中使用三点 ... 表示函数有可变的参数。

```lua
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25
```

通过 `select("#",...)` 来获取可变参数的数量:

```lua
function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. select("#",...) .. " 个数")
   return result/select("#",...)
end

print("平均值为",average(10,5,3,4,5,6))
```

以上代码执行结果为：

```
总共传入 6 个数
平均值为    5.5
```

如果是几个固定参数加上可变参数，固定参数必须放在变长参数之前

```lua
function fwrite(fmt, ...)  ---> 固定的参数fmt
    return io.write(string.format(fmt, ...))    
end

fwrite("runoob\n")       --->fmt = "runoob", 没有变长参数。  
fwrite("%d%d\n", 1, 2)   --->fmt = "%d%d", 变长参数为 1 和 2
```

通常在遍历变长参数的时候只需要使用 {…}，然而变长参数可能会包含一些 nil，那么就可以用 select 函数来访问变长参数了：select('#', …) 或者 select(n, …)

* select('#', …) 返回可变参数的长度
* select(n, …) 用于返回 n 到 select('#',…) 的参数

调用 select 时，必须传入一个固定实参 selector(选择开关)和一系列变长参数。如果selector 为数字n,那么 select 返回它的第n个可变实参，否则只能为字符串"#",这样 select 会返回变长参数的总数。例子代码：

```lua
do  
    function foo(...)  
        for i = 1, select('#', ...) do  -->获取参数总数
            local arg = select(i, ...); -->读取参数
            print("arg", arg);  
        end  
    end  
 
    foo(1, 2, 3, 4);  
end
```

输出结果为：

```
arg    1
arg    2
arg    3
arg    4
```

```lua
select(n,...) --> 返回的是多个参数，而不是一个table
arg = select(n,...) --> 只是接收了第一个参数，参考<多返回值>部分

-- 验证代码
arg = select(2, 1, 6, 8, 8, 9) --> 相当于arg = 6, 8, 8, 9
print(arg)  --> 输出为：6
print(type(arg)) --> 输出为：number（验证了是多返回值的第一个number，而不是打印arg第一个参数的类型table）
print(select(2, 1, 6, 8, 8, 9))  --> 输出为：6    8    8    9
```

注意：多返回值的函数在赋值时的情况，仅仅只有放在所有逗号之后的那个函数会把返回值展开。

这里列举一个典型情况：

```lua
function add()
    return 1,0
end

local b,c,d,e = add(),add()

print(b) -- 1
print(c) -- 1
print(d) -- 0
print(e) -- nil
```

## 运算符

### 算术运算符

![微信截图_20201019163051.png](/img/微信截图_20201019163051.png)

### 关系运算符

![微信截图_20201019163124.png](/img/微信截图_20201019163124.png)

### 逻辑运算符

![微信截图_20201019163151.png](/img/微信截图_20201019163151.png)

### 其他运算符

![微信截图_20201019163220.png](/img/微信截图_20201019163220.png)

```lua
a = "Hello "
b = "World"

print("连接字符串 a 和 b ", a..b )

print("b 字符串长度 ",#b )

print("字符串 Test 长度 ",#"Test" )

print("菜鸟教程网址长度 ",#"www.runoob.com" )
```

输出：

```
连接字符串 a 和 b     Hello World
b 字符串长度     5
字符串 Test 长度     4
菜鸟教程网址长度     14
```

`#` 获取表的最大索引的值。

```lua
tab1 = {"1","2"}
print("tab1长度"..#tab1)
tab2 = {key1="1","2"}
print("tab2长度"..#tab2)
tab3 = {}
tab3[1]="1"
tab3[2]="2"
tab3[4]="4"
print("tab3长度"..#tab3)
```

输出:

```
tab1长度2
tab2长度1
tab3长度4
```
 
下标越过 1 位以上，长度还是为 2：

```lua
tab3={}
tab3[1]="1"
tab3[2]="2"
tab3[5]="5"
print("tab3的长度",#tab3)
```

输出：

```
tab3的长度    2
```

### 三元表达式

利用 and 和 or 的特性，若(A and B) A 为 false 返回 A，(A or B）A 为 false 返回 B，以及除 nil 外其他数据类型被当做 true。

可以非常简单的完成:

```lua
local isAppel = false
print(isAppel and "苹果" or "梨")
```

### 运算符优先级

从高到低的顺序：

```lua
^
not    - (unary)
*      /       %
+      -
..
<      >      <=     >=     ~=     ==
and
or
```

除了 ^ 和 .. 外所有的二元运算符都是左连接的。

```lua
a+i < b/2+1          <-->       (a+i) < ((b/2)+1)
5+x^2*8              <-->       5+((x^2)*8)
a < y and y <= z     <-->       (a < y) and (y <= z)
-x^2                 <-->       -(x^2)
x^y^z                <-->       x^(y^z)
```

```lua
a = 20
b = 10
c = 15
d = 5

e = (a + b) * c / d;-- ( 30 * 15 ) / 5
print("(a + b) * c / d 运算值为  :",e )

e = ((a + b) * c) / d; -- (30 * 15 ) / 5
print("((a + b) * c) / d 运算值为 :",e )

e = (a + b) * (c / d);-- (30) * (15/5)
print("(a + b) * (c / d) 运算值为 :",e )

e = a + (b * c) / d;  -- 20 + (150/5)
print("a + (b * c) / d 运算值为   :",e )
```

输出：

```
(a + b) * c / d 运算值为  :    90.0
((a + b) * c) / d 运算值为 :    90.0
(a + b) * (c / d) 运算值为 :    90.0
a + (b * c) / d 运算值为   :    50.0
```

## 字符串

字符串或串(String)是由数字、字母、下划线组成的一串字符。

Lua 语言中字符串可以使用以下三种方式来表示：

* 单引号间的一串字符。
* 双引号间的一串字符。
* [[ 与 ]] 间的一串字符。

转义字符用于表示不能直接显示的字符，比如后退键，回车键，等。如在字符串转换双引号可以使用 "\""。

![微信截图_20201019165254.png](/img/微信截图_20201019165254.png)

### 字符串操作

```lua
string.upper(argument) -- 字符串全部转为大写字母。
string.lower(argument) -- 字符串全部转为小写字母。
```

### 字符串截取

字符串截取使用 sub() 方法。

```lua
string.sub() 用于截取字符串，原型为：
string.sub(s, i [, j])
```

参数说明：

* s：要截取的字符串。
* i：截取开始位置。
* j：截取结束位置，默认为 -1，最后一个字符。

### 字符串大小写转换

```lua
string1 = "Lua";
print(string.upper(string1))
print(string.lower(string1))
```

### 字符串查找与反转

```lua
string = "Lua Tutorial"
-- 查找字符串
print(string.find(string,"Tutorial"))
reversedString = string.reverse(string)
print("新字符串为",reversedString)
```

### 字符串格式化

string.format() 函数来生成具有特定格式的字符串

### 字符与整数相互转换

```lua
-- 字符转换
-- 转换第一个字符
print(string.byte("Lua"))
-- 转换第三个字符
print(string.byte("Lua",3))
-- 转换末尾第一个字符
print(string.byte("Lua",-1))
-- 第二个字符
print(string.byte("Lua",2))
-- 转换末尾第二个字符
print(string.byte("Lua",-2))

-- 整数 ASCII 码转换为字符
print(string.char(97))
```

输出：

```lua
76
97
97
117
117
a
```

### 其他常用函数

```lua
- 字符串长度
print("字符串长度 ",string.len(string2))

-- 字符串复制 2 次
repeatedString = string.rep(string2,2)
print(repeatedString)
```

### 匹配模式

Lua 中的匹配模式直接用常规的字符串来描述。 它用于模式匹配函数 string.find, string.gmatch, string.gsub, string.match。

## table(表)

```lua
-- 初始化表
mytable = {}

-- 指定值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 垃圾回收会释放内存
```

参考：

[https://www.runoob.com/lua/lua-tables.html](https://www.runoob.com/lua/lua-tables.html)































































