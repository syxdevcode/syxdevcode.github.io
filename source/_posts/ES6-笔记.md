---
title: ES6-笔记
date: 2021-10-09 14:37:42
tags:
- JavaScript
- ES6
categories: 
- JavaScript
---

## let 与 const

参考：[ES6 let 与 const](https://www.runoob.com/w3cnote/es6-let-const.html)

ES2015(ES6) 新增加了两个重要的 JavaScript 关键字: `let` 和 `const`。

let 声明的变量只在 `let` 命令所在的代码块内有效。

const 声明一个只读的常量，一旦声明，常量的值就不能改变。

<!--more-->
**代码块内有效:**

let 是在代码块内有效，var 是在全局范围内有效:

```js
{
  let a = 0;
  var b = 1;
}
a  // ReferenceError: a is not defined
b  // 1
```

**let 只能声明一次 var 可以声明多次:**

```js
let a = 1;
let a = 2;
var b = 3;
var b = 4;
a  // Identifier 'a' has already been declared
b  // 4
```

**let 不存在变量提升，var 会变量提升:**

```js
console.log(a);  //ReferenceError: a is not defined
let a = "apple";
 
console.log(b);  //undefined
var b = "banana";
```

## 解构赋值

在解构中，有下面两部分参与：

* 解构的源，解构赋值表达式的右边部分。
* 解构的目标，解构赋值表达式的左边部分。

### 数组模型的解构（Array）

```js
// 基本
let [a, b, c] = [1, 2, 3];
// a = 1
// b = 2
// c = 3

// 可嵌套
let [a, [[b], c]] = [1, [[2], 3]];
// a = 1
// b = 2
// c = 3

// 可忽略
let [a, , b] = [1, 2, 3];
// a = 1
// b = 3

// 不完全解构
let [a = 1, b] = []; // a = 1, b = undefined

// 剩余运算符
let [a, ...b] = [1, 2, 3];
//a = 1
//b = [2, 3]
```

**字符串:**

在数组的解构中，解构的目标若为可遍历对象，皆可进行解构赋值。可遍历对象即实现 Iterator 接口的数据。

```js
let [a, b, c, d, e] = 'hello';
// a = 'h'
// b = 'e'
// c = 'l'
// d = 'l'
// e = 'o'
```

**解构默认值:**

```js
let [a = 2] = [undefined]; // a = 2
```

当解构模式有匹配结果，且匹配结果是 `undefined` 时，会触发默认值作为返回结果。

```js
let [a = 3, b = a] = [];     // a = 3, b = 3
let [a = 3, b = a] = [1];    // a = 1, b = 1
let [a = 3, b = a] = [1, 2]; // a = 1, b = 2
```

a 与 b 匹配结果为 undefined ，触发默认值：a = 3; b = a =3
a 正常解构赋值，匹配结果：a = 1，b 匹配结果 undefined ，触发默认值：b = a =1
a 与 b 正常解构赋值，匹配结果：a = 1，b = 2

### 对象模型的解构（Object）

```js
// 基本
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
// foo = 'aaa'
// bar = 'bbb'
 
let { baz : foo } = { baz : 'ddd' };
// foo = 'ddd'


// 可嵌套可忽略
let obj = {p: ['hello', {y: 'world'}] };
let {p: [x, { y }] } = obj;
// x = 'hello'
// y = 'world'
let obj = {p: ['hello', {y: 'world'}] };
let {p: [x, {  }] } = obj;
// x = 'hello'

// 不完全解构
let obj = {p: [{y: 'world'}] };
let {p: [{ y }, x ] } = obj;
// x = undefined
// y = 'world'

// 剩余运算符
let {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40};
// a = 10
// b = 20
// rest = {c: 30, d: 40}

// 解构默认值
let {a = 10, b = 5} = {a: 3};
// a = 3; b = 5;
let {a: aa = 10, b: bb = 5} = {a: 3};
// aa = 3; bb = 5;
```

## ES6 模块

参考：[ES6 模块](https://www.runoob.com/w3cnote/es6-module.html)

ES6 的模块化分为导出（`export`） @与导入（`import`）两个模块。

ES6 的模块自动开启严格模式，不管你有没有在模块头部加上 `use strict;`。

模块中可以导入和导出各种类型的变量，如函数，对象，字符串，数字，布尔值，类等。

每个模块都有自己的上下文，每一个模块内声明的变量都是局部变量，不会污染全局作用域。

每一个模块只加载一次（是单例的）， 若再去加载同目录下同文件，直接从内存中读取。

### 基本用法

模块导入导出各种类型的变量，如字符串，数值，函数，类。

* 导出的函数声明与类声明必须要有名称（`export default` 命令另外考虑）。
* 不仅能导出声明还能导出引用（例如函数）。
* `export` 命令可以出现在模块的任何位置，但必需处于模块顶层。
* `import` 命令会提升到整个模块的头部，首先执行。

```js
/*-----export [test.js]-----*/
let myName = "Tom";
let myAge = 20;
let myfn = function(){
    return "My name is" + myName + "! I'm '" + myAge + "years old."
}
let myClass =  class myClass {
    static a = "yeah!";
}
export { myName, myAge, myfn, myClass }
 
/*-----import [xxx.js]-----*/
import { myName, myAge, myfn, myClass } from "./test.js";
console.log(myfn());// My name is Tom! I'm 20 years old.
console.log(myAge);// 20
console.log(myName);// Tom
console.log(myClass.a );// yeah!
```

建议使用大括号指定所要输出的一组变量写在文档尾部，明确导出的接口。

函数与类都需要有对应的名称，导出文档尾部也避免了无对应名称。

### as 的用法

`export` 命令导出的接口名称，须和模块内部的变量有一一对应关系。

导入的变量名，须和导出的接口名称相同，即顺序可以不一致。

不同模块导出接口名称命名重复， 使用 as 重新定义变量名。

### import 命令的特点

**只读属性**：不允许在加载模块的脚本里面，改写接口的引用指向，即可以改写 import 变量类型为对象的属性值，不能改写 import 变量类型为基本类型的值。

```js
import {a} from "./xxx.js"
a = {}; // error
 
import {a} from "./xxx.js"
a.foo = "hello"; // a = { foo : 'hello' }
```

**单例模式**：多次重复执行同一句 `import` 语句，那么只会执行一次，而不会执行多次。`import` 同一模块，声明不同接口引用，会声明对应变量，但只执行一次 `import` 。

```js
import { a } "./xxx.js";
import { a } "./xxx.js";
// 相当于 import { a } "./xxx.js";
 
import { a } from "./xxx.js";
import { b } from "./xxx.js";
// 相当于 import { a, b } from "./xxx.js";
```

静态执行特性：`import` 是静态执行，所以不能使用表达式和变量。

```js
import { "f" + "oo" } from "methods";
// error
let module = "methods";
import { foo } from module;
// error
if (true) {
  import { foo } from "method1";
} else {
  import { foo } from "method2";
}
// error
```

### export default 命令

* 在一个文件或模块中，`export`、`import` 可以有多个，`export default` 仅有一个。
* `export default` 中的 `default` 是对应的导出接口变量。
* 通过 `export` 方式导出，在导入时要加 `{ }`，`export default` 则不需要。
* `export default` 向外暴露的成员，可以使用任意变量来接收。

```js
var a = "My name is Tom!";
export default a; // 仅有一个
export default var c = "error"; 
// error，default 已经是对应的导出变量，不能跟着变量声明语句
 
import b from "./xxx.js"; // 不需要加{}， 使用任意变量接收
```

### 复合使用

`export` 与 `import` 可以在同一模块使用，使用特点：

* 可以将导出接口改名，包括 `default`。
* 复合使用 `export` 与 `import` ，也可以导出全部，当前模块导出的接口会覆盖继承导出的。

```js
export { foo, bar } from "methods";
 
// 约等于下面两段语句，不过上面导入导出方式该模块没有导入 foo 与 bar
import { foo, bar } from "methods";
export { foo, bar };
 
/* ------- 特点 1 --------*/
// 普通改名
export { foo as bar } from "methods";
// 将 foo 转导成 default
export { foo as default } from "methods";
// 将 default 转导成 foo
export { default as foo } from "methods";
 
/* ------- 特点 2 --------*/
export * from "methods";
```









