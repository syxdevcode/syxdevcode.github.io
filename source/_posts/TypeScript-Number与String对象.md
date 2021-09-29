---
title: TypeScript-Number与String对象
date: 2021-09-29 13:37:48
tags:
- TypeScript
- JavaScript
categories: 
- TypeScript
---

## Number对象

TypeScript 与 JavaScript 类似，支持 Number 对象。

Number 对象是原始数值的包装对象。

语法

```ts
var num = new Number(value);
```

注意： 如果一个参数值不能转换为一个数字将返回 NaN (非数字值)。

<!--more-->
### Number对象属性

* **MAX_VALUE**：可表示的最大的数，MAX_VALUE 属性值接近于 1.79E+308。大于 MAX_VALUE 的值代表 "Infinity"。
* **MIN_VALUE**：可表示的最小的数，即最接近 0 的正数 (实际上不会变成 0)。最大的负数是 -MIN_VALUE，MIN_VALUE 的值约为 5e-324。小于 MIN_VALUE ("underflow values") 的值将会转换为 0。
* **NaN**：非数字值（Not-A-Number）。
* **NEGATIVE_INFINITY**：负无穷大，溢出时返回该值。该值小于 MIN_VALUE。
* **POSITIVE_INFINITY**：正无穷大，溢出时返回该值。该值大于 MAX_VALUE。
* **prototype**：Number 对象的静态属性。使您有能力向对象添加属性和方法。
* **constructor**：返回对创建此对象的 Number 函数的引用。

实例：

```ts
console.log("TypeScript Number 属性: "); 
console.log("最大值为: " + Number.MAX_VALUE); 
console.log("最小值为: " + Number.MIN_VALUE); 
console.log("负无穷大: " + Number.NEGATIVE_INFINITY); 
console.log("正无穷大:" + Number.POSITIVE_INFINITY);
console.log("Number.NaN:" + Number.NaN);
```

### prototype实例

```ts
function employee(id:number,name:string) { 
    this.id = id 
    this.name = name 
} 
 
var emp = new employee(123,"admin") 
employee.prototype.email = "admin@runoob.com" 
 
console.log("员工号: "+emp.id) 
console.log("员工姓名: "+emp.name) 
console.log("员工邮箱: "+emp.email)
```

### Number对象方法

#### toExponential():把对象的值转换为指数计数法

```ts
//toExponential() 
var num1 = 1225.30 
var val = num1.toExponential(); 
console.log(val) // 输出： 1.2253e+3
```

#### toFixed()把数字转换为字符串，并对小数点指定位数

```ts
var num3 = 177.234 
console.log("num3.toFixed() 为 "+num3.toFixed())    // 输出：177
console.log("num3.toFixed(2) 为 "+num3.toFixed(2))  // 输出：177.23
console.log("num3.toFixed(6) 为 "+num3.toFixed(6))  // 输出：177.234000
```

#### toLocaleString()把数字转换为字符串，使用本地数字格式顺序

```ts
var num = new Number(177.1234); 
console.log( num.toLocaleString());  // 输出：177.1234
```

#### toPrecision()把数字格式化为指定的长度

```ts
var num = new Number(7.123456); 
console.log(num.toPrecision());  // 输出：7.123456 
console.log(num.toPrecision(1)); // 输出：7
console.log(num.toPrecision(2)); // 输出：7.1
```

#### toString()：把数字转换为字符串，使用指定的基数。数字的基数是 2 ~ 36 之间的整数。若省略该参数，则使用基数 10

```ts
var num = new Number(10); 
console.log(num.toString());  // 输出10进制：10
console.log(num.toString(2)); // 输出2进制：1010
console.log(num.toString(8)); // 输出8进制：12
```

#### valueOf()：返回一个 Number 对象的原始数字值

```ts
var num = new Number(10); 
console.log(num.valueOf()); // 输出：10
```

## String对象

String 对象用于处理文本（字符串）。

```ts
var txt = new String("string");

// 或者更简单方式：
var txt = "string";
```

### String对象属性

**constructor**：对创建该对象的函数的引用。

```ts
var str = new String( "This is string" ); 
console.log("str.constructor is:" + str.constructor)

// 输出结果：
str.constructor is:function String() { [native code] }
```

**length**：返回字符串的长度。

```ts
var uname = new String("Hello World") 
console.log("Length "+uname.length)  // 输出 11
```

**prototype**：允许您向对象添加属性和方法。

```ts
function employee(id:number,name:string) { 
    this.id = id 
    this.name = name 
 } 
 var emp = new employee(123,"admin") 
 employee.prototype.email="admin@runoob.com" // 添加属性 email
 console.log("员工号: "+emp.id) 
 console.log("员工姓名: "+emp.name) 
 console.log("员工邮箱: "+emp.email)
```

### String方法

[TypeScript String（字符串）](https://www.runoob.com/typescript/ts-string.html)

#### charAt() 返回在指定位置的字符

```ts
var str = new String("RUNOOB"); 
console.log("str.charAt(0) 为:" + str.charAt(0)); //R
```

#### charCodeAt()返回在指定的位置的字符的 Unicode 编码

```ts
var str = new String("RUNOOB"); 
console.log("str.charCodeAt(0) 为:" + str.charCodeAt(0)); // 82
```

#### concat()连接两个或更多字符串，并返回新的字符串

```ts
var str1 = new String( "RUNOOB" ); 
var str2 = new String( "GOOGLE" ); 
var str3 = str1.concat( str2 ); 
console.log("str1 + str2 : "+str3) // RUNOOBGOOGLE
```

#### indexOf() 返回某个指定的字符串值在字符串中首次出现的位置

```ts
var str1 = new String( "RUNOOB" ); 

var index = str1.indexOf( "OO" ); 
console.log("查找的字符串位置 :" + index );  // 3
```

#### lastIndexOf()从后向前搜索字符串，并从起始位置（0）开始计算返回字符串最后出现的位置

```ts
var str1 = new String( "This is string one and again string" ); 
var index = str1.lastIndexOf( "string" );
console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 29
    
index = str1.lastIndexOf( "one" ); 
console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 15
```

#### localeCompare()用本地特定的顺序来比较两个字符串

```ts
var str1 = new String( "This is beautiful string" );
  
var index = str1.localeCompare( "This is beautiful string");  

console.log("localeCompare first :" + index );  // 0
```

#### match()查找找到一个或多个正则表达式的匹配

```ts
var str="The rain in SPAIN stays mainly in the plain"; 
var n=str.match(/ain/g);  // ain,ain,ain
```

#### replace()替换与正则表达式匹配的子串

```ts
var re = /(\w+)\s(\w+)/; 
var str = "zara ali"; 
var newstr = str.replace(re, "$2, $1"); 
console.log(newstr); // ali, zara
```

#### search()检索与正则表达式相匹配的值

```ts
var re = /apples/gi; 
var str = "Apples are round, and apples are juicy.";
if (str.search(re) == -1 ) { 
   console.log("Does not contain Apples" ); 
} else { 
   console.log("Contains Apples" ); 
} 
```

#### slice()提取字符串的片断，并在新的字符串中返回被提取的部分

```ts
string.slice( beginslice [, endSlice] );
```

参数详情

* beginSlice - 开始提取的从零开始的索引。
* endSlice - 结束提取的从零开始的索引。如果省略，则将切片提取到字符串的末尾。

```ts
var str = "Apples are round, and apples are juicy."; 
var sliced = str.slice(3, -2); 
console.log(sliced);

// 输出如下：
les are round, and apples are juic
```

#### split()把字符串分割为子字符串数组

```ts
var str = "Apples are round, and apples are juicy."; 
var splitted = str.split(" ", 3); 
console.log(splitted)  // [ 'Apples', 'are', 'round,' ]
```

#### substr()从起始索引号提取字符串中指定数目的字符

```ts
string.substr(start[, length]);
```

参数详情

* start - 开始提取字符的位置（0到1之间的整数，小于字符串的长度）。
* length - 要提取的字符数。

注意 ：如果start为负数，则substr将其用作字符串末尾的字符索引。

```ts
var str = "Apples are round, and apples are juicy."; 
console.log("(1,2): "    + str.substr(1,2)); 
console.log("(-2,2): "   + str.substr(-2,2)); 

// 输出
(1,2): pp 
(-2,2): y. 
```

#### substring()提取字符串中两个指定的索引号之间的字符

```ts
var str = "RUNOOB GOOGLE TAOBAO FACEBOOK"; 
console.log("(1,2): "    + str.substring(1,2));   // U
console.log("(0,10): "   + str.substring(0, 10)); // RUNOOB GOO
console.log("(5): "      + str.substring(5));     // B GOOGLE TAOBAO FACEBOOK
```

#### toLocaleLowerCase()

根据主机的语言环境把字符串转换为小写，只有几种语言（如土耳其语）具有地方特有的大小写映射。

```ts
var str = "Runoob Google"; 
console.log(str.toLocaleLowerCase( ));  // runoob google
```

#### toLocaleUpperCase()

据主机的语言环境把字符串转换为大写，只有几种语言（如土耳其语）具有地方特有的大小写映射。

```ts
var str = "Runoob Google"; 
console.log(str.toLocaleUpperCase( ));  // RUNOOB GOOGLE
```

#### toLowerCase()把字符串转换为小写

```ts
var str = "Runoob Google"; 
console.log(str.toLowerCase( ));  // runoob google
```

#### toString()返回字符串

```ts
var str = "Runoob"; 
console.log(str.toString( )); // Runoob
```

#### toUpperCase()把字符串转换为大写

```ts
var str = "Runoob Google"; 
console.log(str.toUpperCase( ));  // RUNOOB GOOGLE
```

#### valueOf()返回指定字符串对象的原始值

```ts
var str = new String("Runoob"); 
console.log(str.valueOf( ));  // Runoob
```

