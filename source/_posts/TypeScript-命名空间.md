---
title: TypeScript-命名空间
date: 2021-09-29 16:35:53
tags:
- TypeScript
- JavaScript
categories: 
- TypeScript
---

## 定义

TypeScript 中命名空间使用 `namespace` 来定义，语法格式如下：

```ts
namespace SomeNameSpaceName { 
   export interface ISomeInterfaceName {      }  
   export class SomeClassName {      }  
}
```

以上定义了一个命名空间 `SomeNameSpaceName`，如果我们需要在外部可以调用 SomeNameSpaceName 中的类和接口，则需要在类和接口添加 `export` 关键字。

要在另外一个命名空间调用语法格式为：

```ts
SomeNameSpaceName.SomeClassName;
```

如果一个命名空间在一个单独的 TypeScript 文件中，则应使用三斜杠 `///` 引用它，语法格式如下：

```ts
/// <reference path = "SomeFileName.ts" />
```

<!--more-->
**实例:**

IShape.ts 文件代码：

```ts
namespace Drawing { 
    export interface IShape { 
        draw(); 
    }
}
```

Circle.ts 文件代码：

```ts
/// <reference path = "IShape.ts" /> 
namespace Drawing { 
    export class Circle implements IShape { 
        public draw() { 
            console.log("Circle is drawn"); 
        }  
    }
}
```

Triangle.ts 文件代码：

```ts
/// <reference path = "IShape.ts" /> 
namespace Drawing { 
    export class Triangle implements IShape { 
        public draw() { 
            console.log("Triangle is drawn"); 
        } 
    } 
}
```

TestShape.ts 文件代码：

```ts
/// <reference path = "IShape.ts" />   
/// <reference path = "Circle.ts" /> 
/// <reference path = "Triangle.ts" />  
function drawAllShapes(shape:Drawing.IShape) { 
    shape.draw(); 
} 
drawAllShapes(new Drawing.Circle());
drawAllShapes(new Drawing.Triangle());
```

使用 tsc 命令编译以上代码：

```ts
tsc --out app.js TestShape.ts  
```

使用 node 命令查看输出结果为：

```ts
$ node app.js
Circle is drawn
Triangle is drawn
```

## 嵌套命名空间

名空间支持嵌套，可以将命名空间定义在另外一个命名空间里头。

```ts
namespace namespace_name1 { 
    export namespace namespace_name2 {
        export class class_name {    } 
    } 
}
```

成员的访问使用点号 `.` 来实现，如下实例：

Invoice.ts 文件代码：

```ts
namespace Runoob { 
   export namespace invoiceApp { 
      export class Invoice { 
         public calculateDiscount(price: number) { 
            return price * .40; 
         } 
      } 
   } 
}
```

InvoiceTest.ts 文件代码：

```ts
/// <reference path = "Invoice.ts" />
var invoice = new Runoob.invoiceApp.Invoice(); 
console.log(invoice.calculateDiscount(500));
```

使用 `tsc` 命令编译以上代码：

```ts
tsc --out app.js InvoiceTest.ts
```

使用 node 命令查看输出结果为：

```ts
$ node app.js
200
```
