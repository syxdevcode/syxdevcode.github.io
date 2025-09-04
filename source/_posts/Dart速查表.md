---
title: Dart速查表
date: 2025-09-04 09:56:44
tags:
  - Flutter
  - Dart
categories:
  - Dart
---

## 字符串插值

要将表达式的值放入字符串中，请使用 `${expression}`。如果表达式是标识符，则可以省略 `{}`。

```dart
'${3 + 2}'=> '5'
'${"word".toUpperCase()}'=>'WORD'
'$myObject' => myObject.toString() 的值
```

## 可空变量

Dart 强制执行健全的空安全。除非声明允许为空，否则值不能为 null。换句话说，类型默认不可为空。

```dart
int a = null; // INVALID.
int? a = null; // Valid.
int? a; // The initial value of a is null.
```

## 空感知运算符

`??=` 赋值运算符,在该变量当前为 null 时才向其赋值。

`??`，左侧不为空，返回其左侧表达式的值，否则返回右侧表达式的值。

```dart
int? a; // = null
a ??= 3;
print(a); // <-- Prints 3.

a ??= 5;
print(a); // <-- Still prints 3.

print(1 ?? 3); // <-- Prints 1.
print(null ?? 12); // <-- Prints 12.
```

## 条件属性访问

```dart
myObject?.someProperty
// 等同于
(myObject != null) ? myObject.someProperty : null

myObject?.someProperty?.someMethod()
```

## 集合字面量

Dart 对列表、映射和集合有内置支持。

```dart
final aListOfStrings = ['one', 'two', 'three'];
final aSetOfStrings = {'one', 'two', 'three'};
final aMapOfStringsToInts = {'one': 1, 'two': 2, 'three': 3};
```

Dart 的类型推断可以为你这些变量分配类型。在这种情况下，推断的类型是 `List<String>`、`Set<String>` 和 `Map<String, int>`。

自己指定类型

```dart
final aListOfInts = <int>[];
final aSetOfInts = <int>{};
final aMapOfIntToDouble = <int, double>{};
```

当你使用子类型的内容初始化列表，但仍希望列表为 `List<BaseType>` 时，指定类型会很方便

```dart
final aListOfBaseType = <BaseType>[SubType(), SubType()];
```

## 箭头语法

`=>` 符号,语法是定义一个函数的方式，该函数执行其右侧的表达式并返回其值。

```dart
bool hasEmpty = aListOfStrings.any((s) {
  return s.isEmpty;
});

// 简写
bool hasEmpty = aListOfStrings.any((s) => s.isEmpty);
```

## 级联操作

要对同一对象执行一系列操作，请使用级联操作 `..`。

```dart
final button = web.document.querySelector('#confirm');
button?.textContent = 'Confirm';
button?.classList.add('important');
button?.onClick.listen((e) => web.window.alert('Confirmed!'));
button?.scrollIntoView();
```

使用级联语法：

```dart
web.document.querySelector('#confirm')
  ?..textContent = 'Confirm'
  ..classList.add('important')
  ..onClick.listen((e) => web.window.alert('Confirmed!'))
  ..scrollIntoView();
```

## Getter 和 Setter

在 Dart 中，Getter 和 Setter 是用于控制对象属性访问和修改的特殊方法。它们允许你在获取属性值（Getter）或设置属性值（Setter）时添加额外逻辑（如验证、计算、日志等），同时保持属性访问的简洁语法。

### 基本概念

* **Getter**：用于获取属性值的方法，通过 get 关键字定义，无参数，有返回值。
* **Setter**：用于设置属性值的方法，通过 set 关键字定义，有一个参数，无返回值。

Dart 中，每个实例变量默认会隐式生成 `隐式 Getter`（所有变量都有）和 `隐式 Setter`（非 final 变量才有）。当你需要自定义访问逻辑时，可以显式定义 Getter 和 Setter。

### 语法与示例

#### 1. 隐式 Getter/Setter（默认行为）

对于普通实例变量，Dart 会自动生成 Getter 和 Setter，你可以直接通过 . 访问或修改：

```dart
class Person {
  String name; // 非 final 变量，有隐式 Getter 和 Setter
  final int age; // final 变量，只有隐式 Getter（无 Setter，值不可改）

  Person(this.name, this.age);
}

void main() {
  var person = Person('Alice', 20);
  
  // 隐式 Getter：获取值
  print(person.name); // 输出: Alice
  
  // 隐式 Setter：修改值（仅非 final 变量可用）
  person.name = 'Bob';
  print(person.name); // 输出: Bob
  
  // 错误：final 变量没有 Setter，无法修改
  // person.age = 21; 
}
```

#### 2. 自定义 Getter（计算属性）

当属性值需要动态计算（而非直接存储）时，可自定义 Getter。例如，通过宽高计算矩形面积：

```dart
class Rectangle {
  double width;
  double height;

  Rectangle(this.width, this.height);

  // 自定义 Getter：计算面积（无需括号，像访问属性一样调用）
  double get area {
    return width * height;
  }
}

void main() {
  var rect = Rectangle(3, 4);
  print(rect.area); // 输出: 12.0（调用 Getter，无需加 ()）
  
  rect.width = 5;
  print(rect.area); // 输出: 20.0（值随宽高动态变化）
}
```

#### 3. 自定义 Setter（添加验证逻辑）

当设置属性值需要验证（如年龄不能为负数）时，可自定义 Setter：

```dart
class Student {
  String _name; // 私有变量（下划线开头），外部不能直接访问
  int _age;

  Student(this._name, this._age);

  // 自定义 Getter：暴露 _name 的值
  String get name {
    return _name;
  }

  // 自定义 Setter：设置 _name 时去除首尾空格
  set name(String value) {
    _name = value.trim(); // 额外逻辑：去除空格
  }

  // 自定义 Getter：获取 _age
  int get age {
    return _age;
  }

  // 自定义 Setter：验证年龄合法性
  set age(int value) {
    if (value < 0) {
      throw ArgumentError('年龄不能为负数');
    }
    _age = value; // 仅当合法时才赋值
  }
}

void main() {
  var student = Student('  Charlie  ', 18);
  
  print(student.name); // 输出: Charlie（Getter 返回处理后的值）
  
  student.name = '  Dave  ';
  print(student.name); // 输出: Dave（Setter 处理后的值）
  
  student.age = 20;
  print(student.age); // 输出: 20
  
  // 错误：Setter 验证失败，抛出异常
  student.age = -5; // 抛出 ArgumentError
}
```

### 核心特点

* **封装性**：通过将变量设为私有（_ 开头），再用 Getter/Setter 控制访问，可隐藏内部实现细节。
* **简洁调用**：调用 Getter/Setter 时无需加括号，语法与访问普通属性一致（如 obj.area 而非 obj.area()）。
* **灵活性**：可在 Getter 中动态计算值（如面积、全名），在 Setter 中添加验证、格式化等逻辑。
与 final 配合：final 变量只能有 Getter（无 Setter），确保值不可修改。

### 使用场景

* **计算属性**：如 fullName（由 firstName 和 lastName 拼接）、area（由宽高计算）等。
* **输入验证**：设置值时检查合法性（如年龄、价格不能为负）。
* **数据格式化**：设置字符串时自动处理（如去除空格、转小写）。
* **懒加载**：Getter 中延迟初始化耗时资源（首次访问时才加载）。
* **封装私有变量**：隐藏内部状态，只暴露必要的访问接口。

## 可选位置参数

Dart 有两种函数参数：位置参数和命名参数。

位置参数：

```dart
int sumUp(int a, int b, int c) {
  return a + b + c;
}

// ···
int total = sumUp(1, 2, 3);
```

将这些位置参数用括号括起来使它们成为可选参数

```dart
int sumUpToFive(int a, [int? b, int? c, int? d, int? e]) {
  int sum = a;
  if (b != null) sum += b;
  if (c != null) sum += c;
  if (d != null) sum += d;
  if (e != null) sum += e;
  return sum;
}

// ···
int total = sumUpToFive(1, 2);
int otherTotal = sumUpToFive(1, 2, 3, 4, 5);
```

可选位置参数始终位于函数参数列表的末尾。它们的默认值为 null，除非你提供了另一个默认值

```dart
int sumUpToFive(int a, [int b = 2, int c = 3, int d = 4, int e = 5]) {
  // ···
}

void main() {
  int newTotal = sumUpToFive(1);
  print(newTotal); // <-- prints 15
}
```

## 命名参数

通过在参数列表末尾使用大括号语法，你可以定义具有名称的参数。

命名参数是可选的，除非它们被显式标记为 required。可空命名参数的默认值为 null，但你可以提供自定义的默认值。

```dart
void printName(String firstName, String lastName, {String? middleName}) {
  print('$firstName ${middleName ?? ''} $lastName');
}

void main() {
  printName('Dash', 'Dartisan');
  printName('John', 'Smith', middleName: 'Who');
  // Named arguments can be placed anywhere in the argument list.
  printName('John', middleName: 'Who', 'Smith');
}
```

如果参数的类型不可为空，则你必须提供一个默认值（如以下代码所示）或将参数标记为 required（如构造函数部分所示）。

```dart
void printName(String firstName, String lastName, {String middleName = ''}) {
  print('$firstName $middleName $lastName');
}
```

**函数不能同时拥有可选位置参数和命名参数。**

## 异常

Dart 代码可以抛出和捕获异常。Dart 的所有异常都是未检查的。方法不声明它们可能抛出哪些异常，你也不必捕获任何异常。

Dart 提供了 Exception 和 Error 类型，但你可以抛出任何非 null 对象

```dart
throw Exception('Something bad happened.');
throw 'Waaaaaaah!';
```

处理异常时使用 `try`、`on` 和 `catch` 关键词

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e) {
  // No specified type, handles all
  print('Something really unknown: $e');
}
```

`try` 关键词的作用与其他大多数语言相同。使用 `on` 关键词按类型筛选特定异常，使用 `catch` 关键词获取异常对象的引用。

如果你无法完全处理该异常，请使用 `rethrow` 关键词来传播异常

```dart
try {
  breedMoreLlamas();
} catch (e) {
  print('I was just trying to breed llamas!');
  rethrow;
}
```

无论是否抛出异常，要执行代码，请使用 `finally`

```dart
try {
  breedMoreLlamas();
} catch (e) {
  // ... handle exception ...
} finally {
  // Always clean up, even if an exception is thrown.
  cleanLlamaStalls();
}
```

## 在构造函数中使用 this

Dart 提供了一个方便的快捷方式，用于在构造函数中为属性赋值：在声明构造函数时使用 `this.propertyName`。

```dart
class MyColor {
  int red;
  int green;
  int blue;

  MyColor(this.red, this.green, this.blue);
}

final color = MyColor(80, 80, 128);
```

此技术也适用于命名参数。属性名成为参数名

```dart
class MyColor {
  // ...

  MyColor({required this.red, required this.green, required this.blue});
}

final color = MyColor(red: 80, green: 80, blue: 80);
```

在前面的代码中，red、green 和 blue 被标记为 required，因为这些 int 值不能为 null。如果你添加默认值，则可以省略 required

```dart
MyColor([this.red = 0, this.green = 0, this.blue = 0]);
// or
MyColor({this.red = 0, this.green = 0, this.blue = 0});
```

## 初始化列表

初始化列表（Initialization List） 是构造函数的一部分，用于在构造函数体执行之前初始化实例变量，尤其是针对 final 变量或需要基于构造函数参数计算初始值的场景。

### 基本语法

初始化列表位于构造函数参数列表之后、构造函数体（`{}`）之前，以冒号（`:`）开头，多个初始化表达式用逗号（`,`）分隔：

```dart
class ClassName {
  final type variable1;
  type variable2;

  // 构造函数 + 初始化列表
  ClassName(parameter1, parameter2) : 
    variable1 = parameter1 * 2,  // 初始化表达式1
    variable2 = parameter2 + 10  // 初始化表达式2
  {
    // 构造函数体（此时变量已完成初始化）
    print('构造函数体执行');
  }
}
```

初始化列表也是放置断言的方便位置，断言仅在开发期间运行:

```dart
NonNegativePoint(this.x, this.y) : assert(x >= 0), assert(y >= 0) {
  print('I just made a NonNegativePoint: ($x, $y)');
}
```

### 核心作用

#### 初始化 final 变量

final 变量必须在对象创建时完成初始化（不能在构造函数体中赋值），初始化列表是唯一途径：

```dart
class Person {
  final String name;
  final int age;

  // 用初始化列表为 final 变量赋值
  Person(String name, int birthYear) : 
    name = name, 
    age = DateTime.now().year - birthYear;
}
```

#### 基于参数计算初始值

当变量初始值依赖构造函数参数的计算结果时，初始化列表更简洁：

```dart
class Rectangle {
  final double area;
  double width;
  double height;

  // 用参数计算面积（area 是 final 变量）
  Rectangle(this.width, this.height) : area = width * height;
}
```

#### 调用父类构造函数

子类构造函数中，初始化列表可显式调用父类构造函数（必须放在初始化列表末尾）：

```dart
class Animal {
  String name;
  Animal(this.name);
}

class Dog extends Animal {
  String breed;

  // 初始化列表中调用父类构造函数
  Dog(String name, this.breed) : super(name);
}
```

#### 参数验证

可在初始化列表中对参数进行校验，不合法时抛出异常（早于构造函数体执行）：

```dart
class PositiveNumber {
  final int value;

  PositiveNumber(int input) : 
    // 验证参数，不合法则抛出异常
    value = input > 0 ? input : (throw ArgumentError('必须是正数'));
}
```

### 注意事项

* **执行顺序**：初始化列表 → 父类构造函数 → 子类构造函数体。
* **不能访问 this**：初始化列表中，对象尚未完全创建，因此不能使用 this 关键字。
* **命名构造函数也适用**：初始化列表对命名构造函数同样有效：

```dart
class Point {
  double x, y;

  // 命名构造函数 + 初始化列表
  Point.fromJson(Map<String, double> json) : 
    x = json['x'] ?? 0, 
    y = json['y'] ?? 0;
}
```

* **与默认参数配合**：初始化列表可结合构造函数的默认参数使用：

```dart
class Circle {
  final double radius;
  final double circumference;

  Circle({double radius = 1.0}) : 
    radius = radius, 
    circumference = 2 * 3.14 * radius;
}
```

## 命名构造函数

为了允许类有多个构造函数，Dart 支持命名构造函数。

```dart
class Point {
  double x, y;

  Point(this.x, this.y);

  Point.origin() : x = 0, y = 0;
}
```

要使用命名构造函数，请使用其完整名称调用它

```dart
final myPoint = Point.origin();
```

## 工厂构造函数

Dart 支持工厂构造函数，它可以返回子类型甚至 null。要创建工厂构造函数，请使用 `factory` 关键词。

```dart
class Square extends Shape {}

class Circle extends Shape {}

class Shape {
  Shape();

  factory Shape.fromTypeName(String typeName) {
    if (typeName == 'square') return Square();
    if (typeName == 'circle') return Circle();

    throw ArgumentError('Unrecognized $typeName');
  }
}
```

## 重定向构造函数

有时，构造函数唯一目的是重定向到同一类中的另一个构造函数。重定向构造函数的主体为空，构造函数调用出现在冒号 (`:`) 之后。

```dart
class Automobile {
  String make;
  String model;
  int mpg;

  // The main constructor for this class.
  Automobile(this.make, this.model, this.mpg);

  // Delegates to the main constructor.
  Automobile.hybrid(String make, String model) : this(make, model, 60);

  // Delegates to a named constructor
  Automobile.fancyHybrid() : this.hybrid('Futurecar', 'Mark 2');
}
```

## 常量构造函数

如果你的类生成永不改变的对象，你可以使这些对象成为编译时常量。为此，请定义一个 const 构造函数，并确保所有实例变量都是 final 的。

```dart
class ImmutablePoint {
  static const ImmutablePoint origin = ImmutablePoint(0, 0);

  final int x;
  final int y;

  const ImmutablePoint(this.x, this.y);
}
```











参考：

[Dart 速查表](https://dart.ac.cn/resources/dart-cheatsheet)
