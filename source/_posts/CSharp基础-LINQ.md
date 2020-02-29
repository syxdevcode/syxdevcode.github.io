---
title: CSharp基础-LINQ
date: 2018-12-28 14:15:20
tags:
- DotNet
- CSharp
- LINQ
- CSharp基础
categories: 
- LINQ
---
## LINQ 简介

&emsp;&emsp;LINQ（语言集成查询，`Language Integrated Query`） 是 Visual Studio 2008 和 .NET Framework 3.5 版中引入的一项创新功能。它在对象领域和数据领域之间架起了一座桥梁。语言集成查询 (LINQ) 是一系列直接将查询功能集成到 C# 语言的技术统称。

&emsp;&emsp;<font color=#ff0000 size=4 face="黑体">LINQ 最明显的“语言集成”部分就是查询表达式。 查询表达式采用声明性查询语法编写而成。</font> 使用查询语法，可以用最少的代码对数据源执行筛选、排序和分组操作。 可使用相同的基本查询表达式模式来查询和转换 SQL 数据库、ADO .NET 数据集、XML 文档和流以及 .NET 集合中的数据。

### 查询表达式概述

* 查询表达式可用于查询并转换所有启用了 LINQ 的数据源中的数据。
* 查询表达式中的变量全都是强类型，尽管在许多情况下，无需显式提供类型，因为编译器可以推断出。
* 只有在循环访问查询变量后，才会执行查询（例如，在 `foreach` 语句中）
* 一些查询操作（如 `Count` 或 `Max`）没有等效的查询表达式子句，因此必须表示为方法调用。
* 查询表达式可被编译成表达式树或委托，具体视应用查询的类型而定。 <font color=#ff0000 size=4 face="黑体">`IEnumerable<T>` 查询编译为委托。 `IQueryable` 和 `IQueryable<T>` 查询编译为表达式树。</font>

### LINQ 中的查询语法和方法语法

&emsp;&emsp;在编译代码时，查询语法必须转换为针对 .NET 公共语言运行时 (CLR) 的方法调用。 这些方法调用会调用标准查询运算符（名称为 Where、Select、GroupBy、Join、Max 和 Average 等）。 可以使用方法语法（而不查询语法）来直接调用它们。

#### 查询语法

```cs
 //Query syntax:
 IEnumerable<int> numQuery1 =
    from num in numbers
     where num % 2 == 0
     orderby num
     select num;
```

#### 方法语法

```cs
//Method syntax:
IEnumerable<int> numQuery2 = numbers.Where(num => num % 2 == 0).OrderBy(n => n);
```

### Lambda 表达式

在 C# 中，`=>` 是 `lambda` 运算符（读为“转到”）。

<font color=#ff0000 size=4 face="黑体">当我们的Lambda表达式里面用到了外部变量的时候，编译器会为这个Lambda生成一个类，在这个类中包含了我们表达式方法。</font>

```cs
//Query syntax:
IEnumerable<int> numQuery1 =
    from num in numbers
    where num % 2 == 0
    orderby num
    select num;

//Method syntax:
IEnumerable<int> numQuery2 = numbers.Where(num => num % 2 == 0).OrderBy(n => n);
```

&emsp;&emsp;在上面的示例中，请注意，条件表达式 (num % 2 == 0) 作为内联参数传递给 Where 方法：Where(num => num % 2 == 0). 此内联表达式称为 lambda 表达式。 可采用匿名方法、泛型委托或表达式树的形式编写原本必须以更繁琐的形式编写的代码。

&emsp;&emsp;Lambda 的主体与查询语法中或任何其他 C# 表达式或语句中的表达式完全相同；它可以包含方法调用和其他复杂逻辑。 “返回值”就是表达式结果。

某些查询只能采用方法语法进行表示，而其中一些查询需要 lambda 表达式。

### 对象和集合初始值设定项

通过对象和集合初始值设定项，初始化对象时无需为对象显式调用构造函数。 初始值设定项通常用在将源数据投影到新数据类型的查询表达式中。 假定一个类名为 `Customer`，具有公共 `Name` 和 `Phone` 属性，可以按下列代码中所示使用对象初始值设定项：

```cs
Customer cust = new Customer { Name = "Mike", Phone = "555-1212" }; 
var newLargeOrderCustomers = from o in IncomingOrders
                            where o.OrderSize > 5
                            select new Customer { Name = o.Name, Phone = o.Phone };
```

述代码也可以使用 LINQ 的方法语法编写:

```cs
var newLargeOrderCustomers = IncomingOrders.Where(x => x.OrderSize > 5).Select(y => new Customer { Name = y.Name, Phone = y.Phone });
```

## LINQ 查询

所有 LINQ 查询操作都由以下三个不同的操作组成：
获取数据源。
创建查询。
执行查询。

### 数据源

基本规则很简单：LINQ 数据源是支持泛型 `IEnumerable<T>` 接口或从中继承的接口的任意对象。

### 查询

在 LINQ 中，查询变量本身不执行任何操作并且不返回任何数据。 它只是存储在以后某个时刻执行查询时为生成结果而必需的信息。

### 查询执行

#### 延迟执行

查询的实际执行将推迟到在 `foreach` 语句中循环访问查询变量之后进行。 此概念称为延迟执行，下面的示例对此进行了演示：

```cs
//  Query execution. 
foreach (int num in numQuery)
{
    Console.Write("{0,1} ", num);
}
```

在应用程序中，可以创建一个检索最新数据的查询，并可以按某一时间间隔反复执行该查询以便每次检索不同的结果。

#### 强制立即执行

对一系列源元素执行聚合函数的查询必须首先循环访问这些元素。 `Count`、`Max`、`Average` 和 `First` 就属于此类查询。由于查询本身必须使用 `foreach` 以便返回结果，因此这些查询在执行时不使用显式 `foreach` 语句。 另外还要注意，这些类型的查询返回单个值，而不是 `IEnumerable` 集合。 下面的查询返回源数组中偶数的计数：

```cs
var evenNumQuery =
    from num in numbers
    where (num % 2) == 0
    select num;

int evenNumCount = evenNumQuery.Count();
```

要强制立即执行任何查询并缓存其结果，可调用 `ToList` 或 `ToArray` 方法。

```cs
List<int> numQuery2 =
    (from num in numbers
     where (num % 2) == 0
     select num).ToList();

// or like this:
// numQuery3 is still an int[]

var numQuery3 =
    (from num in numbers
     where (num % 2) == 0
     select num).ToArray();
```

## 基本 LINQ 查询操作

### 获取数据源

使用 from 子句引入数据源:

```cs
//queryAllCustomers is an IEnumerable<Customer>
var queryAllCustomers = from cust in customers
                        select cust;
```

可通过 `let` 子句引入其他范围变量。

```cs
var earlyBirdQuery =
     from sentence in strings
     let words = sentence.Split(' ')
     from word in words
     let w = word.ToLower()
     where w[0] == 'a' || w[0] == 'e'
         || w[0] == 'i' || w[0] == 'o'
         || w[0] == 'u'
     select word;
```

### 筛选

筛选器使查询仅返回表达式为 true 的元素。 将通过使用 `where` 子句生成结果。

```cs
var queryLondonCustomers = from cust in customers
                           where cust.City == "London"
                           select cust;
```

### 中间件排序

`orderby` 子句根据要排序类型的默认比较器，对返回序列中的元素排序。

```cs
var queryLondonCustomers3 =
    from cust in customers
    where cust.City == "London"
    orderby cust.Name ascending
    select cust;
```

要对结果进行从 Z 到 A 的逆序排序，请使用 `orderby…descending` 子句。

### 分组

`group` 子句用于对根据指定的键所获得的结果进行分组。

使用 `group` 子句结束查询时，结果将以列表的形式列出。 列表中的每个元素都是具有 `Key` 成员的对象，列表中的元素根据该键被分组。 在循环访问生成组序列的查询时，必须使用嵌套 `foreach` 循环。 外层循环循环访问每个组，内层循环循环访问每个组的成员。

```cs
// queryCustomersByCity is an IEnumerable<IGrouping<string, Customer>>
  var queryCustomersByCity =
      from cust in customers
      group cust by cust.City;

  // customerGroup is an IGrouping<string, Customer>
  foreach (var customerGroup in queryCustomersByCity)
  {
      Console.WriteLine(customerGroup.Key);
      foreach (Customer customer in customerGroup)
      {
          Console.WriteLine("    {0}", customer.Name);
      }
  }
```

如果必须引用某个组操作的结果，可使用 into 关键字创建能被进一步查询的标识符。 下列查询仅返回包含两个以上客户的组：

```cs
// custQuery is an IEnumerable<IGrouping<string, Customer>>
var custQuery =
    from cust in customers
    group cust by cust.City into custGroup
    where custGroup.Count() > 2
    orderby custGroup.Key
    select custGroup;
```

### 联接

在 LINQ 中，join 子句始终作用于对象集合，而非直接作用于数据库表。

```cs
var innerJoinQuery =
    from cust in customers
    join dist in distributors on cust.City equals dist.City
    select new { CustomerName = cust.Name, DistributorName = dist.Name };
```

在 LINQ 中，不必像在 SQL 中那样频繁使用 join，因为 LINQ 中的外键在对象模型中表示为包含项集合的属性。 例如 Customer 对象包含 Order 对象的集合。 不必执行联接，只需使用点表示法访问订单：

```cs
from order in Customer.Orders...
```

### 选择（投影）

<font color=#0099ff size=4 face="黑体">select 子句生成查询结果并指定每个返回的元素的“形状”或类型。当 select 子句生成除源元素副本以外的内容时，该操作称为投影。</font>

## 使用 LINQ 进行数据转换

语言集成查询 (LINQ) 不只是检索数据。 它也是用于转换数据的强大工具。 通过使用 LINQ 查询，可以使用源序列作为输入，并通过多种方式对其进行修改，以创建新的输出序列。通过排序和分组，你可以修改序列本身，而无需修改这些元素本身。

`select` 可以执行下列任务

* 将多个输入序列合并为具有新类型的单个输出序列。
* 创建其元素由源序列中每个元素的一个或多个属性组成的输出序列。
* 创建其元素由对源数据执行的操作结果组成的输出序列。
* 创建其他格式的输出序列。 例如，可以将数据从 SQL 行或文本文件转换为 XML。

### 将多个输入联接到一个输出序列中

<font color=#0099ff size=4 face="黑体">可以使用 LINQ 查询创建包含元素的输出序列，这些元素来自多个输入序列。</font>

例如：

```cs
class Student
{
    public string First { get; set; }
    public string Last {get; set;}
    public int ID { get; set; }
    public string Street { get; set; }
    public string City { get; set; }
    public List<int> Scores;
}

class Teacher
{
    public string First { get; set; }
    public string Last { get; set; }
    public int ID { get; set; }
    public string City { get; set; }
}
class DataTransformations
{
    static void Main()
    {
        // Create the first data source.
        List<Student> students = new List<Student>()
        {
            new Student { First="Svetlana",
                Last="Omelchenko",
                ID=111,
                Street="123 Main Street",
                City="Seattle",
                Scores= new List<int> { 97, 92, 81, 60 } },
            new Student { First="Claire",
                Last="O’Donnell",
                ID=112,
                Street="124 Main Street",
                City="Redmond",
                Scores= new List<int> { 75, 84, 91, 39 } },
            new Student { First="Sven",
                Last="Mortensen",
                ID=113,
                Street="125 Main Street",
                City="Lake City",
                Scores= new List<int> { 88, 94, 65, 91 } },
        };

        // Create the second data source.
        List<Teacher> teachers = new List<Teacher>()
        {
            new Teacher { First="Ann", Last="Beebe", ID=945, City="Seattle" },
            new Teacher { First="Alex", Last="Robinson", ID=956, City="Redmond" },
            new Teacher { First="Michiyo", Last="Sato", ID=972, City="Tacoma" }
        };

        // Create the query.
        var peopleInSeattle = (from student in students
                    where student.City == "Seattle"
                    select student.Last)
                    .Concat(from teacher in teachers
                            where teacher.City == "Seattle"
                            select teacher.Last);

        Console.WriteLine("The following students and teachers live in Seattle:");
        // Execute the query.
        foreach (var person in peopleInSeattle)
        {
            Console.WriteLine(person);
        }

        Console.WriteLine("Press any key to exit.");
        Console.ReadKey();
    }
}
/* Output:
    The following students and teachers live in Seattle:
    Omelchenko
    Beebe
 */
```

### 选择每个源元素的子集

有两种主要方法来选择源序列中每个元素的子集：

1，若要仅选择源元素的一个成员，请使用点操作。

```cs
var query = from cust in Customers  
            select cust.City; 
```

2，要创建包含多个源元素属性的元素，可以使用带有命名对象或匿名类型的对象初始值设定项。

```cs
var query = from cust in Customer  
            select new {Name = cust.Name, City = cust.City};
```

### 将内存中对象转换为 XML

LINQ 查询可以方便地在内存中数据结构、SQL 数据库、ADO.NET 数据集和 XML 流或文档之间转换数据。 

```cs
static void Main()
{
    // Create the data source by using a collection initializer.
    // The Student class was defined previously in this topic.
    List<Student> students = new List<Student>()
    {
        new Student {First="Svetlana", Last="Omelchenko", ID=111, Scores = new List<int>{97, 92, 81, 60}},
        new Student {First="Claire", Last="O’Donnell", ID=112, Scores = new List<int>{75, 84, 91, 39}},
        new Student {First="Sven", Last="Mortensen", ID=113, Scores = new List<int>{88, 94, 65, 91}},
    };

    // Create the query.
    var studentsToXML = new XElement("Root",
        from student in students
        let scores = string.Join(",", student.Scores)
        select new XElement("student",
                   new XElement("First", student.First),
                   new XElement("Last", student.Last),
                   new XElement("Scores", scores)
                ) // end "student"
            ); // end "Root"

    // Execute the query.
    Console.WriteLine(studentsToXML);

    // Keep the console open in debug mode.
    Console.WriteLine("Press any key to exit.");
    Console.ReadKey();
}
```

输出：

```cs
<Root>  
  <student>  
    <First>Svetlana</First>  
    <Last>Omelchenko</Last>  
    <Scores>97,92,81,60</Scores>  
  </student>  
  <student>  
    <First>Claire</First>  
    <Last>O'Donnell</Last>  
    <Scores>75,84,91,39</Scores>  
  </student>  
  <student>  
    <First>Sven</First>  
    <Last>Mortensen</Last>  
    <Scores>88,94,65,91</Scores>  
  </student>  
</Root>  
```

### 对源元素执行操作

&emsp;&emsp;<font color=#0099ff size=4 face="黑体">输出序列可能不包含源序列中的任何元素或元素属性。 输出可能是使用源元素作为输入参数而计算得出的值序列。</font>

备注:
&emsp;&emsp;如果查询将被转换为另一个域，则不支持在查询表达式中调用方法。 例如，不能在 LINQ to SQL 中调用普通的 C# 方法，因为 SQL Server 没有用于它的上下文。 但是，可以将存储过程映射到方法并调用这些方法。

```cs
static void Main()
{
    // Data source.
    double[] radii = { 1, 2, 3 };

    // Query.
    IEnumerable<string> query =
        from rad in radii
        select $"Area = {rad * rad * Math.PI:F2}";

    // Query execution. 
    foreach (string s in query)
        Console.WriteLine(s);

    // Keep the console open in debug mode.
    Console.WriteLine("Press any key to exit.");
    Console.ReadKey();
}

/* Output:
    Area = 3.14
    Area = 12.57
    Area = 28.27
*/
```

## LINQ 查询操作中的类型关系

&emsp;&emsp;LINQ 查询操作在数据源、查询本身及查询执行中是强类型化的。 查询中变量的类型必须与数据源中元素的类型和 foreach 语句中迭代变量的类型兼容。 此强类型保证在编译时捕获类型错误，以便可以在用户遇到这些错误之前更正它们。

### 不转换源数据的查询

![linq_flow1.png](/img/linq_flow1.png)

### 转换源数据的查询

![linq_flow2.png](/img/linq_flow2.png)

![linq_flow3.png](/img/linq_flow3.png)

### 让编译器推断类型信息

关键字 `var` 可用于查询操作中的任何本地变量。

![linq_flow4.png](/img/linq_flow4.png)

参考：

[https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/let-clause](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/let-clause)

[语言集成查询 (LINQ)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/)