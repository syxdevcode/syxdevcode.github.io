---
title: TypeScript变量，函数，语法
date: 2021-09-28 09:17:19
tags:
- TypeScript
- JavaScript
categories: 
- TypeScript
---

## 变量

### 变量定义

TypeScript 变量的命名规则：

* 变量名称可以包含数字和字母。
* 除了下划线 `_` 和美元 `$` 符号外，不能包含其他特殊字符，包括空格。
* 变量名不能以数字开头。

<!--more-->
* 1，变量使用前必须先声明，可以使用 `var` 来声明变量。

声明变量的类型及初始值：

```ts
var [变量名] : [类型] = 值;

// 实例：
var uname:string = "hello world";
```

* 2，声明变量的类型，但没有初始值，变量值会设置为 `undefined`：

```ts
var [变量名] : [类型];

// 例如：
var uname:string;
```

* 3，声明变量并初始值，但不设置类型，该变量可以是任意类型：

```ts
var [变量名] = 值;

// 例如：
var uname = "Runoob";
```

* 4，声明变量没有设置类型和初始值，类型可以是任意类型，默认初始值为 `undefined`：

```ts
var [变量名];

// 例如：
var uname;
```

实例

```ts
var uname:string = "Runoob";
var score1:number = 50;
var score2:number = 42.50
var sum = score1 + score2
console.log("名字: " + uname)
console.log("第一个科目成绩: "+score1)
console.log("第二个科目成绩: "+score2)
console.log("总成绩: "+sum)
```

注意：变量不要使用 name 否则会与 DOM 中的全局 window 对象下的 name 属性出现了重名。

使用 tsc 命令编译以上代码，得到如下 JavaScript 代码：

```js
var uname = "Runoob";
var score1 = 50;
var score2 = 42.50;
var sum = score1 + score2;
console.log("名字: " + uname);
console.log("第一个科目成绩: " + score1);
console.log("第二个科目成绩: " + score2);
console.log("总成绩: " + sum);
```

执行该 JavaScript 代码输出结果为：

```js
名字: Runoob
第一个科目成绩: 50
第二个科目成绩: 42.5
总成绩: 92.5
```

TypeScript 遵循强类型，如果将不同的类型赋值给变量会编译错误，如下实例：

```ts
var num:number = "hello"    // 这个代码会编译错误
```

### 类型断言（Type Assertion）

类型断言可以用来手动指定一个值的类型，即允许变量从一种类型更改为另一种类型。

语法格式：

```ts
<类型>值

// 或:
值 as 类型
```

实例

```ts
var str = '1' 
var str2:number = <number> <any> str   //str、str2 是 string 类型
console.log(str2)
```

TypeScript 是怎么确定单个断言是否足够

当 S 类型是 T 类型的子集，或者 T 类型是 S 类型的子集时，S 能被成功断言成 T。这是为了在进行类型断言时提供额外的安全性，完全毫无根据的断言是危险的，如果你想这么做，你可以使用 any。

它之所以不被称为类型转换，是因为转换通常意味着某种运行时的支持。但是，类型断言纯粹是一个编译时语法，同时，它也是一种为编译器提供关于如何分析代码的方法。

编译后，以上代码会生成如下 JavaScript 代码：

```js
var str = '1';
var str2 = str;  //str、str2 是 string 类型
console.log(str2);
```

执行输出结果为：

```ts
1
```

### 类型推断

当类型没有给出时，TypeScript 编译器利用类型推断来推断类型。

如果由于缺乏声明而不能推断出类型，那么它的类型被视作默认的动态 `any` 类型。

```ts
var num = 2;    // 类型推断为 number
console.log("num 变量的值为 "+num); 
num = "12";    // 编译错误
console.log(num);
```

* 第一行代码声明了变量 num 并=设置初始值为 2。 注意变量声明没有指定类型。因此，程序使用类型推断来确定变量的数据类型，第一次赋值为 2，num 设置为 number 类型。
* 第三行代码，当我们再次为变量设置字符串类型的值时，这时编译会错误。因为变量已经设置为了 number 类型。

```ts
error TS2322: Type '"12"' is not assignable to type 'number'.
```

### 变量作用域

变量作用域指定了变量定义的位置。

程序中变量的可用性由变量作用域决定。

TypeScript 有以下几种作用域：

* **全局作用域** − 全局变量定义在程序结构的外部，它可以在你代码的任何位置使用。
* **类作用域** − 这个变量也可以称为 字段。类变量声明在一个类里头，但在类的方法外面。 该变量可以通过类的对象来访问。类变量也可以是静态的，静态的变量可以通过类名直接访问。
* **局部作用域** − 局部变量，局部变量只能在声明它的一个代码块（如：方法）中使用。

实例：

```ts
var global_num = 12          // 全局变量
class Numbers { 
   num_val = 13;             // 实例变量
   static sval = 10;         // 静态变量
   
   storeNum():void { 
      var local_num = 14;    // 局部变量
   } 
} 
console.log("全局变量为: "+global_num)  
console.log(Numbers.sval)   // 静态变量
var obj = new Numbers(); 
console.log("实例变量: "+obj.num_val)
```

以上代码使用 tsc 命令编译为 JavaScript 代码为：

```js
var global_num = 12; // 全局变量
var Numbers = /** @class */ (function () {
    function Numbers() {
        this.num_val = 13; // 实例变量
    }
    Numbers.prototype.storeNum = function () {
        var local_num = 14; // 局部变量
    };
    Numbers.sval = 10; // 静态变量
    return Numbers;
}());
console.log("全局变量为: " + global_num);
console.log(Numbers.sval); // 静态变量
var obj = new Numbers();
console.log("实例变量: " + obj.num_val);
```

执行以上 JavaScript 代码，输出结果为：

```ts
全局变量为: 12
10
实例变量: 13
```

如果我们在方法外部调用局部变量 local_num，会报错：

```ts
error TS2322: Could not find symbol 'local_num'.
```

## 运算符

参考：[TypeScript 运算符](https://www.runoob.com/typescript/ts-operators.html)

TypeScript 主要包含以下几种运算：

* 算术运算符
* 逻辑运算符
* 关系运算符
* 按位运算符
* 赋值运算符
* 三元/条件运算符
* 字符串运算符
* 类型运算符

### 算术运算符

假定 y=5，下面的表格解释了这些算术运算符的操作：

![微信截图_20210929112231.png](/img/微信截图_20210929112231.png)

### 关系运算符

关系运算符用于计算结果是否为 true 或者 false。

x=5，下面的表格解释了关系运算符的操作：

![微信截图_20210929112316.png](/img/微信截图_20210929112316.png)

### 逻辑运算符

逻辑运算符用于测定变量或值之间的逻辑。

给定 x=6 以及 y=3，下表解释了逻辑运算符：

![微信截图_20210929112424.png](/img/微信截图_20210929112424.png)

### 短路运算符(&& 与 ||)

```ts
var a = 10 
var result = ( a<10 && a>5)

var a = 10 
var result = ( a>5 || a<10)
```

### 位运算符

位操作是程序设计中对位模式按位或二进制数的一元和二元操作。

![微信截图_20210929112634.png](/img/微信截图_20210929112634.png)

### 赋值运算符

给定 x=10 和 y=5，下面的表格解释了赋值运算符：

![微信截图_20210929112717.png](/img/微信截图_20210929112717.png)

### 三元运算符 (?)

```ts
var num:number = -2 
var result = num > 0 ? "大于 0" : "小于 0，或等于 0" 
console.log(result)
```

### 类型运算符

#### typeof运算符

typeof 是一元运算符，返回操作数的数据类型。

查看以下实例:

```ts
var num = 12 
console.log(typeof num);   //输出结果: number
```

使用 tsc 命令编译以上代码得到如下 JavaScript 代码：

```ts
var num = 12;
console.log(typeof num); //输出结果: number
```

以上实例输出结果如下：

```ts
number
```

#### instanceof

instanceof 运算符用于判断对象是否为指定的类型。


### 其他运算符

#### 负号运算符(-)

更改操作数的符号，查看以下实例：

```ts
var x:number = 4 
var y = -x; 
console.log("x 值为: ",x);   // 输出结果 4 
console.log("y 值为: ",y);   // 输出结果 -4
```

使用 tsc 命令编译以上代码得到如下 JavaScript 代码：

```ts
var x = 4;
var y = -x;
console.log("x 值为: ", x); // 输出结果 4 
console.log("y 值为: ", y); // 输出结果 -4
```

以上实例输出结果如下：

```ts
x 值为:  4
y 值为:  -4
```

#### 字符串运算符: 连接运算符 (+)

`+` 运算符可以拼接两个字符串，查看以下实例：

```ts
var msg:string = "testcn"+".COM" 
console.log(msg)
```

使用 tsc 命令编译以上代码得到如下 JavaScript 代码：

```ts
var msg = "testcn" + ".COM";
console.log(msg);
```

以上实例输出结果如下：

```ts
testcn.COM
```

## 语法

### 条件语句

条件语句：

* if 语句 - 只有当指定条件为 true 时，使用该语句来执行代码
* if...else 语句 - 当条件为 true 时执行代码，当条件为 false 时执行其他代码
* if...else if....else 语句- 使用该语句来选择多个代码块之一来执行
* switch 语句 - 使用该语句来选择多个代码块之一来执行

示例：

```ts
// if 语句
var  num:number = 5
if (num > 0) { 
   console.log("数字是正数") 
}

// if...else 语句
var num:number = 12; 
if (num % 2==0) { 
    console.log("偶数"); 
} else {
    console.log("奇数"); 
}

// if...else if....else 语句
var num:number = 2 
if(num > 0) { 
    console.log(num+" 是正数") 
} else if(num < 0) { 
    console.log(num+" 是负数") 
} else { 
    console.log(num+" 不是正数也不是负数") 
}

// switch 语句
var grade:string = "A"; 
switch(grade) { 
    case "A": { 
        console.log("优"); 
        break; 
    } 
    case "B": { 
        console.log("良"); 
        break; 
    } 
    case "C": {
        console.log("及格"); 
        break;    
    } 
    case "D": { 
        console.log("不及格"); 
        break; 
    }  
    default: { 
        console.log("非法输入"); 
        break;              
    } 
}
```

### 循环

#### for循环

语法格式如下所示：

```ts
for ( init; condition; increment ){
    statement(s);
}
```

流程解析：

1，init 会首先被执行，且只会执行一次。这一步允许您声明并初始化任何循环控制变量。也可以不在这里写任何语句，只要有一个分号出现即可。
2，接下来，会判断 condition。如果为 true，则执行循环主体。如果为 false，则不执行循环主体，且控制流会跳转到紧接着 for 循环的下一条语句。
3，在执行完 for 循环主体后，控制流会跳回上面的 increment 语句。该语句允许您更新循环控制变量。该语句可以留空，只要在条件后有一个分号出现即可。
4，条件再次被判断。如果为 true，则执行循环，这个过程会不断重复（循环主体，然后增加步值，再然后重新判断条件）。在条件变为 false 时，for 循环终止。

statement(s) 可以是一个单独的语句，也可以是几个语句组成的代码块。
condition 可以是任意的表达式，当条件为 true 时执行循环，当条件为 false 时，退出循环。

实例：

计算 5 的阶乘， for 循环生成从 5 到 1 的数字，并计算每次循环数字的乘积。

```ts
var num:number = 5; 
var i:number; 
var factorial = 1; 
 
for(i = num;i>=1;i--) {
   factorial *= i;
}
console.log(factorial)
```

#### for...in 循环

语法格式如下所示：

```ts
for (var val in list) { 
    //语句 
}
```

val 需要为 string 或 any 类型

实例

```ts
var j:any; 
var n:any = "a b c" 
 
for(j in n) {
    console.log(n[j])  
}
```

#### for…of 、forEach、every 和 some 循环

TypeScript 还支持 for…of 、forEach、every 和 some 循环。

for...of 语句创建一个循环来迭代可迭代的对象。在 ES6 中引入的 for...of 循环，以替代 for...in 和 forEach() ，并支持新的迭代协议。for...of 允许遍历 Arrays（数组）, Strings（字符串）, Maps（映射）, Sets（集合）等可迭代的数据结构等。

**for...of 循环:**

```ts
let someArray = [1, "string", false];
 
for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```

forEach、every 和 some 是 JavaScript 的循环语法，TypeScript 作为 JavaScript 的语法超集，当然默认也是支持的。

因为 forEach 在 iteration 中是无法返回的，所以可以使用 every 和 some 来取代 forEach。

**forEach 循环:**

```ts
let list = [4, 5, 6];
list.forEach((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
});
```

**every循环:**

```ts
let list = [4, 5, 6];
list.every((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
    return true; // Continues
    // Return false will quit the iteration
});
```

#### while 循环

```ts
var num:number = 5; 
var factorial:number = 1; 
 
while(num >=1) { 
    factorial = factorial * num; 
    num--; 
} 
console.log("5 的阶乘为："+factorial);
```

#### do...while 循环

```ts
var n:number = 10;
do { 
    console.log(n); 
    n--; 
} while(n>=0);
```

#### break,continue与无限循环

break     // 找到一个后退出循环
continue  // 会跳过当前循环中的代码，强迫开始下一次循环

**无限循环:**

```ts
for(;;) { 
   // 语句
}
```
