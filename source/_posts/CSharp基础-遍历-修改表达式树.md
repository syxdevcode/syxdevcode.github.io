---
title: CSharp基础-遍历/修改表达式树
date: 2019-01-04 15:44:45
tags:
- DotNet
- CSharp
- LINQ
- Expression Tree
- CSharp基础
categories: 
- Expression Tree
---
# 遍历/修改表达式树

可以使用 [ExpressionVisitor](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.expressionvisitor?view=netframework-4.7.2) 类遍历现有表达式树，以及复制它访问的每个节点。

## 表达式的遍历

参考：

[ExpressionType](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.expressiontype?view=netframework-4.7.2)

[Expression](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.expression?view=netframework-4.7.2)

[ExpressionVisitor](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.expressionvisitor?view=netframework-4.7.2)

表达式类型|表达式操作节点类型|ExpressionVisitor类访问方法
----------|----------------|--------------------------
[UnaryExpression](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.unaryexpression?view=netframework-4.7.2) | ExpressionType.ArrayLength </br>ExpressionType.ArrayLength </br>ExpressionType.Convert </br>ExpressionType.ConvertChecked </br>ExpressionType.Decrement </br>ExpressionType.Increment </br>ExpressionType.Negate </br>ExpressionType.NegateChecked </br>ExpressionType.Not </br>ExpressionType.NotEqual </br>ExpressionType.Quote </br>ExpressionType.TypeAs | VisitUnary(UnaryExpression)
[BinaryExpression](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.binaryexpression?view=netframework-4.7.2)|ExpressionType.Add</br> ExpressionType.AddAssign</br>ExpressionType.AddAssignChecked</br>ExpressionType.AddChecked</br>ExpressionType.And</br>ExpressionType.AndAlso</br>ExpressionType.AndAssign</br>ExpressionType.ArrayIndex</br>ExpressionType.Assign</br>ExpressionType.Coalesce</br>ExpressionType.Divide</br>ExpressionType.DivideAssign</br>ExpressionType.Equal</br>ExpressionType.GreaterThan</br>ExpressionType.GreaterThanOrEqual</br>ExpressionType.LessThan</br>ExpressionType.LessThanOrEqual</br>ExpressionType.Modulo</br>ExpressionType.ModuloAssign</br>ExpressionType.Multiply</br>ExpressionType.MultiplyAssign</br>ExpressionType.NotEqual</br>ExpressionType.Or</br>ExpressionType.OrElse</br>ExpressionType.Subtract|VisitBinary(BinaryExpression)
BlockExpression|ExpressionType.Block |VisitBlock(BlockExpression)
ConditionalExpression|ExpressionType.Conditional|VisitConditional(ConditionalExpression)
ConstantExpression|ExpressionType.Constant|VisitConstant(ConstantExpression)
ParameterExpression|ExpressionType.Parameter|VisitParameter(ParameterExpression)
MemberExpression|ExpressionType.MemberAccess|VisitMember(MemberExpression)
MemberInitExpression|ExpressionType.MemberInit|VisitMemberInit(MemberInitExpression)
MethodCallExpression|ExpressionType.Call|VisitMethodCall(MethodCallExpression)
LambdaExpression|ExpressionType.Lambda|VisitLambda()
NewExpression|ExpressionType.New|VisitNew(NewExpression)
NewArrayExpression|ExpressionType.NewArrayBounds</br>ExpressionType.NewArrayInit|VisitNewArray(NewArrayExpression)
InvocationExpression|ExpressionType.Invoke|VisitInvocation(InvocationExpression)
ListInitExpression|ExpressionType.ListInit|VisitListInit(ListInitExpression)
TypeBinaryExpression|ExpressionType.TypeIs|VisitTypeBinary(TypeBinaryExpression)

扩展IQueryable的Where方法，根据输入的查询条件来构造SQL语句。

```cs
internal class Program
{
    private static void Main(string[] args)
    {
        List<User> myUsers = new List<User>();
        var userSql = myUsers.AsQueryable().Where(u => u.Age > 2);
        Console.WriteLine(userSql);
        // SELECT * FROM (SELECT * FROM User) AS T WHERE (Age>2)

        List<User> myUsers2 = new List<User>();
        var userSql2 = myUsers2.AsQueryable().Where(u => u.Name == "QueryExtension");
        Console.WriteLine(userSql2);
        //SELECT * FROM (SELECT * FROM USER) AS T WHERE (Name='QueryExtension')

        Console.ReadKey();
    }
}

public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public static class QueryExtensions
{
    public static string Where<TSource>(this IQueryable<TSource> source,
        Expression<Func<TSource, bool>> predicate)
    {
        var expression = Expression.Call(null, ((MethodInfo)MethodBase.GetCurrentMethod())
            .MakeGenericMethod(new Type[] { typeof(TSource) }),
            new Expression[] { source.Expression, Expression.Quote(predicate) });

        var translator = new QueryTranslator();
        return translator.Translate(expression);
    }
}

internal class QueryTranslator : ExpressionVisitor
{
    private StringBuilder sb;

    internal string Translate(Expression expression)
    {
        this.sb = new StringBuilder();
        this.Visit(expression);
        return this.sb.ToString();
    }

    private static Expression StripQuotes(Expression e)
    {
        while (e.NodeType == ExpressionType.Quote)
        {
            e = ((UnaryExpression)e).Operand;
        }
        return e;
    }

    protected override Expression VisitMethodCall(MethodCallExpression m)
    {
        if (m.Method.DeclaringType == typeof(QueryExtensions) && m.Method.Name == "Where")
        {
            sb.Append("SELECT * FROM (");
            this.Visit(m.Arguments[0]);
            sb.Append(") AS T WHERE ");
            LambdaExpression lambda = (LambdaExpression)StripQuotes(m.Arguments[1]);
            this.Visit(lambda.Body);
            return m;
        }
        throw new NotSupportedException(string.Format("方法{0}不支持", m.Method.Name));
    }

    protected override Expression VisitUnary(UnaryExpression u)
    {
        switch (u.NodeType)
        {
            case ExpressionType.Not:
                sb.Append(" NOT ");
                this.Visit(u.Operand);
                break;

            default:
                throw new NotSupportedException(string.Format("运算{0}不支持", u.NodeType));
        }
        return u;
    }

    protected override Expression VisitBinary(BinaryExpression b)
    {
        sb.Append("(");
        this.Visit(b.Left);
        switch (b.NodeType)
        {
            case ExpressionType.And:
                sb.Append(" AND ");
                break;

            case ExpressionType.Or:
                sb.Append(" OR");
                break;

            case ExpressionType.Equal:
                sb.Append(" = ");
                break;

            case ExpressionType.NotEqual:
                sb.Append(" <> ");
                break;

            case ExpressionType.LessThan:
                sb.Append(" < ");
                break;

            case ExpressionType.LessThanOrEqual:
                sb.Append(" <= ");
                break;

            case ExpressionType.GreaterThan:
                sb.Append(" > ");
                break;

            case ExpressionType.GreaterThanOrEqual:
                sb.Append(" >= ");
                break;

            default:
                throw new NotSupportedException(string.Format("运算符{0}不支持", b.NodeType));
        }
        this.Visit(b.Right);
        sb.Append(")");
        return b;
    }

    protected override Expression VisitConstant(ConstantExpression c)
    {
        IQueryable q = c.Value as IQueryable;
        if (q != null)
        {
            // 我们假设我们那个Queryable就是对应的表
            sb.Append("SELECT * FROM ");
            sb.Append(q.ElementType.Name);
        }
        else if (c.Value == null)
        {
            sb.Append("NULL");
        }
        else
        {
            switch (Type.GetTypeCode(c.Value.GetType()))
            {
                case TypeCode.Boolean:
                    sb.Append(((bool)c.Value) ? 1 : 0);
                    break;

                case TypeCode.String:
                    sb.Append("'");
                    sb.Append(c.Value);
                    sb.Append("'");
                    break;

                case TypeCode.Object:
                    throw new NotSupportedException(string.Format("常量{0}不支持", c.Value));
                default:
                    sb.Append(c.Value);
                    break;
            }
        }
        return c;
    }

    protected override Expression VisitMember(MemberExpression m)
    {
        if (m.Expression != null && m.Expression.NodeType == ExpressionType.Parameter)
        {
            sb.Append(m.Member.Name);
            return m;
        }
        throw new NotSupportedException(string.Format("成员{0}不支持", m.Member.Name));
    }
}
```

## 修改表达式树

```cs
public class OperationsVisitor : ExpressionVisitor
{
    public Expression Modify(Expression expression)
    {
        return this.Visit(expression);
    }

    protected override Expression VisitBinary(BinaryExpression b)
    {
        if (b.NodeType == ExpressionType.Add)
        {
            Expression left = this.Visit(b.Left); // // 或b.Left
            Expression right = this.Visit(b.Right); // 或b.Right
            return Expression.Subtract(left,right);
        }

        return base.VisitBinary(b);
    }
}

static void Main(string[] args)
{
    Expression<Func<int, int, int>> lambda = (a, b) => a + b * 2;

    var operationsVisitor = new OperationsVisitor();
    Expression modifyExpression = operationsVisitor.Modify(lambda);

    Console.WriteLine(modifyExpression.ToString());
}
```

参考：

[表达式树](https://docs.microsoft.com/zh-cn/dotnet/csharp/expression-trees)

[ExpressionVisitor](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions.expressionvisitor?view=netframework-4.7.2)

[System.Linq.Expressions](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.expressions?view=netframework-4.7.2)

[由浅入深表达式树（二）遍历表达式树](https://www.cnblogs.com/jesse2013/p/expressiontree-part2.html)