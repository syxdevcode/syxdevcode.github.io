---
title: TypeScript-模块
date: 2021-09-29 16:57:56
tags:
- TypeScript
- JavaScript
categories: 
- TypeScript
---

模块是在其自身的作用域里执行，并不是在全局作用域，这意味着定义在模块里面的变量、函数和类等在模块外部是不可见的，除非明确地使用 `export` 导出它们。必须通过 `import` 导入其他模块导出的变量、函数、类等。

两个模块之间的关系是通过在文件级别上使用 `import` 和 `export` 建立的。

模块导出使用关键字 `export` 关键字，语法格式如下：

```ts
// 文件名 : SomeInterface.ts 
export interface SomeInterface { 
   // 代码部分
}
```

要在另外一个文件使用该模块就需要使用 `import` 关键字来导入:

```ts
import someInterfaceRef = require("./SomeInterface");
```

<!--more-->

**实例:**

IShape.ts 文件代码：

```ts
/// <reference path = "IShape.ts" /> 
export interface IShape { 
   draw(); 
}
```

Circle.ts 文件代码：

```ts
import shape = require("./IShape"); 
export class Circle implements shape.IShape { 
   public draw() { 
      console.log("Cirlce is drawn (external module)"); 
   } 
}
```

Triangle.ts 文件代码：

```ts
import shape = require("./IShape"); 
export class Triangle implements shape.IShape { 
   public draw() { 
      console.log("Triangle is drawn (external module)"); 
   } 
}
```

TestShape.ts 文件代码：

```ts
import shape = require("./IShape"); 
import circle = require("./Circle"); 
import triangle = require("./Triangle");  
 
function drawAllShapes(shapeToDraw: shape.IShape) {
   shapeToDraw.draw(); 
} 
 
drawAllShapes(new circle.Circle()); 
drawAllShapes(new triangle.Triangle());
```

使用 `tsc` 命令编译以上代码（AMD）：

```ts
tsc --module amd TestShape.ts 
```
