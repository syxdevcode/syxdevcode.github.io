---
title: dot net数字格式化
date: 2019-11-13 16:06:35
tags:
- DotNet
- CSharp
- CSharp基础
categories: 
- CSharp基础
---

标准数字格式字符串用于格式化通用数值类型。 标准数字格式字符串采用 Axx 的形式。

A 是称为“格式说明符” 的单个字母字符。

xx 是称为“精度说明符” 的可选整数。 精度说明符的范围从 0 到 99，并且影响结果中的位数。

## 常用格式

### 十进制（“D”）格式说明符

“D”或“d” ：表示10进制；

示例：

```cs
int value;

value = 12345;
Console.WriteLine(value.ToString("D"));
// Displays 12345
Console.WriteLine(value.ToString("D8"));
// Displays 00012345

value = -12345;
Console.WriteLine(value.ToString("D"));
// Displays -12345
Console.WriteLine(value.ToString("D8"));
// Displays -00012345
```

### 定点（“F”）格式说明符

定点（“F”）格式说明符将数字转换为“-ddd.ddd…”形式的字符串，其中每个“d”表示一个数字 (0-9)。 如果该数字为负，则该字符串以减号开头。

```cs
int integerNumber;
integerNumber = 17843;
Console.WriteLine(integerNumber.ToString("F", 
                  CultureInfo.InvariantCulture));
// Displays 17843.00

integerNumber = -29541;
Console.WriteLine(integerNumber.ToString("F3", 
                  CultureInfo.InvariantCulture));
// Displays -29541.000

double doubleNumber;
doubleNumber = 18934.1879;
Console.WriteLine(doubleNumber.ToString("F", CultureInfo.InvariantCulture));
// Displays 18934.19

Console.WriteLine(doubleNumber.ToString("F0", CultureInfo.InvariantCulture));
// Displays 18934

doubleNumber = -1898300.1987;
Console.WriteLine(doubleNumber.ToString("F1", CultureInfo.InvariantCulture));  
// Displays -1898300.2

Console.WriteLine(doubleNumber.ToString("F3", 
                  CultureInfo.CreateSpecificCulture("es-ES")));
// Displays -1898300,199
```

### 百分比（“P”）格式说明符

百分比（“P”）格式说明符将数字乘以 100 并将其转换为表示百分比的字符串。

“P”或“p”

```cs
double number = .2468013;
Console.WriteLine(number.ToString("P", CultureInfo.InvariantCulture));
// Displays 24.68 %
Console.WriteLine(number.ToString("P", 
                  CultureInfo.CreateSpecificCulture("hr-HR")));
// Displays 24,68%
Console.WriteLine(number.ToString("P1", CultureInfo.InvariantCulture));
// Displays 24.7 %
```

### 数字（“N”）格式说明符

数字（"N"）格式说明符将数字转换为"-d,ddd,ddd.ddd…"形式的字符串

“N”或“n”

```cs
double dblValue = -12445.6789;
Console.WriteLine(dblValue.ToString("N", CultureInfo.InvariantCulture));
// Displays -12,445.68
Console.WriteLine(dblValue.ToString("N1", 
                  CultureInfo.CreateSpecificCulture("sv-SE")));
// Displays -12 445,7

int intValue = 123456789;
Console.WriteLine(intValue.ToString("N1", CultureInfo.InvariantCulture));
// Displays 123,456,789.0 
```

### 十六进制（“X”）格式说明符

十六进制（“X”）格式说明符将数字转换为十六进制数的字符串。

“X”或“x”

```cs
int value; 

value = 0x2045e;
Console.WriteLine(value.ToString("x"));
// Displays 2045e
Console.WriteLine(value.ToString("X"));
// Displays 2045E
Console.WriteLine(value.ToString("X8"));
// Displays 0002045E

value = 123456789;
Console.WriteLine(value.ToString("X"));
// Displays 75BCD15
Console.WriteLine(value.ToString("X2"));
// Displays 75BCD15
```

十六进制的表示:
C语言、Shell、Python语言及其他相近的语言使用字首“0x”，例如“0x5A3”。开头的“0”令解析器更易辨认数，而“x”则代表十六进制。在“0x”中的“x”可以大写或小写。

参考：

[.NET 中的格式类型](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/formatting-types?view=netframework-4.8)

[标准数字格式字符串](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/standard-numeric-format-strings?view=netframework-4.8#using-standard-numeric-format-strings)

[复合格式设置](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting)

[自定义数字格式字符串](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/custom-numeric-format-strings?view=netframework-4.8)