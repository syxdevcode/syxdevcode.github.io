---
title: CSharp基础-表达式树
date: 2019-01-02 16:21:12
tags:
- DotNet
- CSharp
- LINQ
- Expression Tree
- CSharp基础
categories: 
- Expression Tree
---
# CSharp基础-表达式树

表达式树(Expression Tree)是.NET 3.5推出的。
<font color=#ff0000 size=4 face="黑体">表达式树以树形数据结构表示代码，其中每一个节点都是一种表达式，比如方法调用和 x < y 这样的二元运算等。</font>

表达式树还能用于动态语言运行时 (DLR) 以提供动态语言和 .NET Framework 之间的互操作性，同时保证编译器编写员能够发射表达式树而非 Microsoft 中间语言 (MSIL)。

## Expressions 类

`System.Linq.Expressions` 命名空间包含一些类、接口和枚举，它们使语言级别的代码表达式能够表示为表达式目录树形式的对象。

类 | 解释
---|----
BinaryExpression| 表示具有二进制运算符的表达式。
UnaryExpression|表示具有一元运算符的表达式。
BlockExpression| 表示包含一个表达式序列的块，表达式中可定义变量。
CatchBlock|表示 try 块中的 catch 语句。
ConditionalExpression|表示包含条件运算符的表达式。
ConstantExpression|表示具有常量值的表达式。
GotoExpression|表示无条件跳转。 这包括返回语句，break 和 continue 语句以及其他跳转。
IndexExpression	|表示对一个属性或数组进行索引。
LabelTarget	|用于表示 GotoExpression 的目标。
LambdaExpression|介绍 lambda 表达式。 它捕获一个类似于 .NET 方法主体的代码块。
ListInitExpression|表示具有集合初始值设定项的构造函数调用。
LoopExpression|表示无限循环。 可通过“中断”退出该循环。
MemberAssignment|表示对象的字段或属性的赋值操作。
MemberBinding|提供表示绑定的类派生自的基类，这些绑定用于对新创建对象的成员进行初始化。
MemberExpression|表示访问字段或属性。
MemberInitExpression|表示调用构造函数并初始化新对象的一个或多个成员。
MemberListBinding|表示初始化新创建对象的一个集合成员的元素。
MemberMemberBinding|表示初始化新创建对象的一个成员的成员。
MethodCallExpression|表示对静态方法或实例方法的调用。
NewArrayExpression|表示创建一个新数组，并可能初始化该新数组的元素。
NewExpression|表示一个构造函数调用。
ParameterExpression|表示一个命名的参数表达式。
TryExpression|表示一个 try/catch/finally/fault 块。

参考：[System.Linq.Expressions](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions?view=netframework-4.7.2)

## 创建表达式树

### Lambda 表达式创建表达式树

若 lambda 表达式被分配给 `Expression<TDelegate>` 类型的变量，则编译器可以发射代码以创建表示该 lambda 表达式的表达式树。
C# 编译器只能从表达式 Lambda（或单行 Lambda）生成表达式树。 它无法解析语句 lambda （或多行 lambda）。

如：

```cs
// 示例展示如何通过 C# 编译器创建表示 Lambda 表达式 num => num < 5 的表达式树
Expression<Func<int, bool>> lambda = num => num < 5;  

// 下面的代码编译不通过
Expression<Func<int, int, int>> expr2 = (x, y) => { return x + y; };
Expression<Action<int>> expr3 = x => {  };
```

`Expression<TDelegate>` 是直接继承自 `LambdaExpression`。

```cs
public sealed class Expression<TDelegate> : LambdaExpression
{
    internal Expression(Expression body, string name, bool tailCall, ReadOnlyCollection<ParameterExpression> parameters) :
    base(typeof(TDelegate), name, body, tailCall, parameters)
    {
    }
}
```

### 通过 API 创建表达式树

通过 API 创建表达式树需要使用 `Expression` 类。类包含创建特定类型表达式树节点的静态工厂方法，比如表示参数变量的` ParameterExpression`，或者是表示方法调用的 `MethodCallExpression`。 `ParameterExpression` 名称空间还解释了 `MethodCallExpression`、`System.Linq.Expressions`和另一种具体表达式类型。 这些类型来源于抽象类型 `Expression`。

使用 API 创建表示 Lambda 表达式 num => num < 5 的表达式树:

```cs
using System.Linq.Expressions;

// Manually build the expression tree for
// the lambda expression num => num < 5.  
ParameterExpression numParam = Expression.Parameter(typeof(int), "num");  
ConstantExpression five = Expression.Constant(5, typeof(int));  
BinaryExpression numLessThanFive = Expression.LessThan(numParam, five);  
Expression<Func<int, bool>> lambda1 =  
    Expression.Lambda<Func<int, bool>>(  
        numLessThanFive,  
        new ParameterExpression[] { numParam });  
```

在 .NET Framework 4 或更高版本中，表达式树 API 还支持赋值表达式和控制流表达式，例如循环、条件块和 `try-catch` 块等。下列示例展示如何创建计算数字阶乘的表达式树。

```cs
// Creating a parameter expression.  
ParameterExpression value = Expression.Parameter(typeof(int), "value");

// Creating an expression to hold a local variable.
ParameterExpression result = Expression.Parameter(typeof(int), "result");

// Creating a label to jump to from a loop.  
LabelTarget label = Expression.Label(typeof(int));

// Creating a method body.  
BlockExpression block = Expression.Block(
    // Adding a local variable.  
    new[] { result },
    // Assigning a constant to a local variable: result = 1  
    Expression.Assign(result, Expression.Constant(1)),
        // Adding a loop.  
        Expression.Loop(
           // Adding a conditional block into the loop.  
           Expression.IfThenElse(
               // Condition: value > 1  
               Expression.GreaterThan(value, Expression.Constant(1)),
               // If true: result *= value --  
               Expression.MultiplyAssign(result,
                   Expression.PostDecrementAssign(value)),
               // If false, exit the loop and go to the label.  
               Expression.Break(label, result)
           ),
       // Label to jump to.  
       label
    )
);

// Compile and execute an expression tree.  
int factorial = Expression.Lambda<Func<int, int>>(block, value).Compile()(5);

Console.WriteLine(factorial);
// Prints 120.
```

### 解析表达式树

下列代码示例展示如何分解表示 Lambda 表达式 num => num < 5 的表达式树。

```cs
// Add the following using directive to your code file:  
// using System.Linq.Expressions;  
  
// Create an expression tree.  
Expression<Func<int, bool>> exprTree = num => num < 5;  
  
// Decompose the expression tree.  
ParameterExpression param = (ParameterExpression)exprTree.Parameters[0];  
BinaryExpression operation = (BinaryExpression)exprTree.Body;  
ParameterExpression left = (ParameterExpression)operation.Left;  
ConstantExpression right = (ConstantExpression)operation.Right;  
  
Console.WriteLine("Decomposed expression: {0} => {1} {2} {3}",  
                  param.Name, left.Name, operation.NodeType, right.Value);  
  
// This code produces the following output:  
  
// Decomposed expression: num => num LessThan 5
```

### 编译表达式树

`Expression<TDelegate>` 类型提供了 Compile 方法以将表达式树表示的代码编译成可执行委托。

下列代码示例展示如何编译表达式树并运行结果代码。

```cs
// Creating an expression tree.  
Expression<Func<int, bool>> expr = num => num < 5;  
  
// Compiling the expression tree into a delegate.  
Func<int, bool> result = expr.Compile();  
  
// Invoking the delegate and writing the result to the console.  
Console.WriteLine(result(4));  
  
// Prints True.  
  
// You can also use simplified syntax  
// to compile and run an expression tree.  
// The following line can replace two previous statements.  
Console.WriteLine(expr.Compile()(4));  
  
// Also prints True.
```

## 执行表达式树

Lambda 表达式的表达式树的类型为 `LambdaExpression` 或 `Expression<TDelegate>`。 若要执行这些表达式树，调用 Compile 方法来创建一个可执行的委托，然后调用该委托。

备注：

如果委托的类型未知，也就是说 Lambda 表达式的类型为 `LambdaExpression`，而不是 `Expression<TDelegate>`，则必须对委托调用 `DynamicInvoke` 方法，而不是直接调用委托。如 `lambdaExpr.Compile().DynamicInvoke(1)`;


下面的代码示例演示如何通过创建 lambda 表达式并执行它来执行代表幂运算的表达式树。

```cs
// The expression tree to execute.  
BinaryExpression be = Expression.Power(Expression.Constant(2D), Expression.Constant(3D));  
  
// Create a lambda expression.  
Expression<Func<double>> le = Expression.Lambda<Func<double>>(be);  
  
// Compile the lambda expression.  
Func<double> compiledExpression = le.Compile();  
  
// Execute the lambda expression.  
double result = compiledExpression();  
  
// Display the result.  
Console.WriteLine(result);  
  
// This code produces the following output:  
// 8
```

## 修改表达式树 (C#)

表达式树是不可变的，这意味着不能直接对它们进行修改。 若要更改表达式树，必须创建现有表达式树的副本，创建此副本后，进行必要的更改。 可以使用 `ExpressionVisitor` 类遍历现有表达式树，以及复制它访问的每个节点。

运算从条件 `AND` 更改为条件 `OR`。

```cs
private static void Main(string[] args)
{
    Expression<Func<string, bool>> expr = name => name.Length > 10 && name.StartsWith("G");
    Console.WriteLine(expr);

    AndAlsoModifier treeModifier = new AndAlsoModifier();
    Expression modifiedExpr = treeModifier.Modify((Expression)expr);

    Console.WriteLine(modifiedExpr);

    /*  This code produces the following output:

        name => ((name.Length > 10) && name.StartsWith("G"))
        name => ((name.Length > 10) || name.StartsWith("G"))
    */

    Console.Read();
}

public class AndAlsoModifier : ExpressionVisitor
{
    public Expression Modify(Expression expression)
    {
        return Visit(expression);
    }

    protected override Expression VisitBinary(BinaryExpression b)
    {
        if (b.NodeType == ExpressionType.AndAlso)
        {
            Expression left = this.Visit(b.Left);
            Expression right = this.Visit(b.Right);

            // Make this binary expression an OrElse operation instead of an AndAlso operation.
            return Expression.MakeBinary(ExpressionType.OrElse, left, right, b.IsLiftedToNull, b.Method);
        }

        return base.VisitBinary(b);
    }
}
```

## 使用表达式树来生成动态查询

在 LINQ 中，表达式树用于表示针对数据源的结构化查询，这些数据源可实现 `IQueryable<T>`。 例如，LINQ 提供程序可实现 `IQueryable<T>` 接口，用于查询关系数据存储。 C# 编译器将针对此类数据源的查询编译为代码，该代码在运行时会生成一个表达式树。 然后，查询提供程序可以遍历表达式树数据结构，并将其转换为适合于数据源的查询语言。

表达式树还可以用在 LINQ 中，用于表示分配给类型为 `Expression<TDelegate>` 的变量的 lambda 表达式。

下面的示例演示如何使用表达式树依据 `IQueryable` 数据源构造一个查询，然后执行该查询。 代码将生成一个表达式树来表示以下查询：

```cs
companies.Where(company => (company.ToLower() == "coho winery" || company.Length > 16)).OrderBy(company => company)
```

`System.Linq.Expressions` 命名空间中的工厂方法用于创建表达式树，这些表达式树表示构成总体查询的表达式。 表示标准查询运算符方法调用的表达式将引用这些方法的 `Queryable` 实现。 最终的表达式树将传递给 `IQueryable` 数据源的提供程序的 `CreateQuery<TElement>(Expression)` 实现，以创建 `IQueryable` 类型的可执行查询。 通过枚举该查询变量获得结果。

```cs
// Add a using directive for System.Linq.Expressions.
string[] companies = { "Consolidated Messenger", "Alpine Ski House", "Southridge Video", "City Power & Light",
       "Coho Winery", "Wide World Importers", "Graphic Design Institute", "Adventure Works",
       "Humongous Insurance", "Woodgrove Bank", "Margie's Travel", "Northwind Traders",
       "Blue Yonder Airlines", "Trey Research", "The Phone Company",
       "Wingtip Toys", "Lucerne Publishing", "Fourth Coffee" };

// The IQueryable data to query.
IQueryable<String> queryableData = companies.AsQueryable<string>();

// Compose the expression tree that represents the parameter to the predicate.
ParameterExpression pe = Expression.Parameter(typeof(string), "company");

// ***** Where(company => (company.ToLower() == "coho winery" || company.Length > 16)) *****
// Create an expression tree that represents the expression 'company.ToLower() == "coho winery"'.
Expression left = Expression.Call(pe, typeof(string).GetMethod("ToLower", System.Type.EmptyTypes));
Expression right = Expression.Constant("coho winery");
Expression e1 = Expression.Equal(left, right);

// Create an expression tree that represents the expression 'company.Length > 16'.
left = Expression.Property(pe, typeof(string).GetProperty("Length"));
right = Expression.Constant(16, typeof(int));
Expression e2 = Expression.GreaterThan(left, right);

// Combine the expression trees to create an expression tree that represents the
// expression '(company.ToLower() == "coho winery" || company.Length > 16)'.
Expression predicateBody = Expression.OrElse(e1, e2);

// Create an expression tree that represents the expression
// 'queryableData.Where(company => (company.ToLower() == "coho winery" || company.Length > 16))'
MethodCallExpression whereCallExpression = Expression.Call(
    typeof(Queryable),
    "Where",
    new Type[] { queryableData.ElementType },
    queryableData.Expression,
    Expression.Lambda<Func<string, bool>>(predicateBody, new ParameterExpression[] { pe }));
// ***** End Where *****

// ***** OrderBy(company => company) *****
// Create an expression tree that represents the expression
// 'whereCallExpression.OrderBy(company => company)'
MethodCallExpression orderByCallExpression = Expression.Call(
    typeof(Queryable),
    "OrderBy",
    new Type[] { queryableData.ElementType, queryableData.ElementType },
    whereCallExpression,
    Expression.Lambda<Func<string, string>>(pe, new ParameterExpression[] { pe }));
// ***** End OrderBy *****

// Create an executable query from the expression tree.
IQueryable<string> results = queryableData.Provider.CreateQuery<string>(orderByCallExpression);

// Enumerate the results.
foreach (string company in results)
    Console.WriteLine(company);

/*  This code produces the following output:

    Blue Yonder Airlines
    City Power & Light
    Coho Winery
    Consolidated Messenger
    Graphic Design Institute
    Humongous Insurance
    Lucerne Publishing
    Northwind Traders
    The Phone Company
    Wide World Importers
*/
```

## 在 Visual Studio 中调试表达式树

参考：[在 Visual Studio 中调试表达式树](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/expression-trees/debugging-expression-trees-in-visual-studio)

## 参考

[System.Linq.Expressions](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions?view=netframework-4.7.2)

[表达式树 (C#)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/expression-trees/)

[由浅入深表达式树（一）创建表达式树](http://www.cnblogs.com/jesse2013/p/expressiontree-part1.html)