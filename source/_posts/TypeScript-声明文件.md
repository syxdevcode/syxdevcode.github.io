---
title: TypeScript-声明文件
date: 2021-09-29 17:18:42
tags:
- TypeScript
- JavaScript
categories: 
- TypeScript
---

[TypeScript 声明文件](https://www.runoob.com/typescript/ts-ambient.html)

TypeScript 作为 JavaScript 的超集，在开发过程中不可避免要引用其他第三方的 JavaScript 的库。虽然通过直接引用可以调用库的类和方法，但是却无法使用TypeScript 诸如类型检查等特性功能。为了解决这个问题，需要将这些库里的函数和方法体去掉后只保留导出类型声明，而产生了一个描述 JavaScript 库和模块信息的声明文件。通过引用这个声明文件，就可以借用 TypeScript 的各种特性来使用库文件了。

## 声明文件

声明文件以 `.d.ts` 为后缀，例如：

```ts
runoob.d.ts
```

声明文件或模块的语法格式如下：

```ts
declare module Module_Name {
}
```

TypeScript 引入声明文件语法格式：

```ts
/// <reference path = "runoob.d.ts" />
```

<!--more-->

## 实例

以下定义一个第三方库来演示：

CalcThirdPartyJsLib.js 文件代码：

```js
var Runoob;  
(function(Runoob) {
    var Calc = (function () { 
        function Calc() { 
        } 
    })
    Calc.prototype.doSum = function (limit) {
        var sum = 0; 
 
        for (var i = 0; i <= limit; i++) { 
            sum = sum + i; 
        }
        return sum; 
    }
    Runoob.Calc = Calc; 
    return Calc; 
})(Runoob || (Runoob = {})); 
var test = new Runoob.Calc();
```

如果我们想在 TypeScript 中引用上面的代码，则需要设置声明文件 Calc.d.ts，代码如下：

Calc.d.ts 文件代码：

```ts
declare module Runoob { 
   export class Calc { 
      doSum(limit:number) : number; 
   }
}
```

声明文件不包含实现，它只是类型声明，把声明文件加入到 TypeScript 中：

CalcTest.ts 文件代码：

```ts
/// <reference path = "Calc.d.ts" /> 
var obj = new Runoob.Calc(); 
// obj.doSum("Hello"); // 编译错误
console.log(obj.doSum(10));
```

下面这行导致编译错误，因为我们需要传入数字参数：

```ts
obj.doSum("Hello");
```

使用 tsc 命令来编译以上代码文件：

```ts
tsc CalcTest.ts
```

生成的 JavaScript 代码如下：

CalcTest.js 文件代码：

```ts
/// <reference path = "Calc.d.ts" /> 
var obj = new Runoob.Calc();
//obj.doSum("Hello"); // 编译错误
console.log(obj.doSum(10));
```

最后编写一个 runoob.html 文件，引入 CalcTest.js 文件及第三方库 CalcThirdPartyJsLib.js：

实例

```ts
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
<script src = "CalcThirdPartyJsLib.js"></script> 
<script src = "CalcTest.js"></script> 
</head>
<body>
    <h1>声明文件测试</h1>
    <p>菜鸟测试一下。</p>
</body>
</html>
```

浏览器打开该文件输出结果如下：

![847256CE-6F06-41FC-944E-EEB89176F358.jpg](/img/847256CE-6F06-41FC-944E-EEB89176F358.jpg)
