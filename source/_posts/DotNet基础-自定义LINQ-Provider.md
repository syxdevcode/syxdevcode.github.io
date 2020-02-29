---
title: DotNet基础-自定义LINQ Provider
date: 2019-01-07 15:12:23
tags:
- DotNet
- CSharp
- LINQ
- Expression Tree
- CSharp基础
categories: 
- LINQ
---
## IQueryable和IQueryProvider接口

`IQueryable<T>` 接口是 `Linq to Provider`的入口。

### IQueryable接口

在.NET中，`IQueryable<T>` 继承于 `IEnumerable<T>` , `IEnumerable` 和 `IQueryable` 接口

```cs
public interface IQueryable<out T> : IEnumerable<T>, IEnumerable, IQueryable
{
}

public interface IQueryable : IEnumerable
{
  Expression Expression { [__DynamicallyInvokable] get; }

  Type ElementType { [__DynamicallyInvokable] get; }

  IQueryProvider Provider { [__DynamicallyInvokable] get; }
}
```

* ElementType ：当前Query所对应的类型
* Expression ：获取与 `IQueryable` 的实例关联的表达式树
* Provider：获取与此数据源关联的查询提供程序

我们所有定义在查询表达式中方法调用或者Lambda表达式都将由该Expression属性表示，最终会由Provider表示的提供程序翻译为它所对应的数据源的查询语言。

`IQueryable` 扩展方法传入的是 `Expression` 类型

### IEnumerable接口

`IEnumrable` 扩展方法传入的是委托。

```cs
public interface IEnumerable<out T> : IEnumerable
{
  IEnumerator<T> GetEnumerator();
}

public interface IEnumerable
{
  IEnumerator GetEnumerator();
}
```

### IQueryProvider接口

```cs
public interface IQueryProvider
{
  IQueryable CreateQuery(Expression expression);

  IQueryable<TElement> CreateQuery<TElement>(Expression expression);

  object Execute(Expression expression);

  TResult Execute<TResult>(Expression expression);
}
```

`Provider` 负责执行表达式目录树并返回结果。如果是 `LINQ to SQL` 的 `Provider` ，则它会负责把表达式目录树翻译为T-SQL语句并并传递给数据库服务器，并返回最后的执行的结果；如果是一个 `Web Service` 的 `Provider`，则它会负责翻译表达式目录树并调用 `Web Service`，最终返回结果。

## 自定义Provider

首先，实现一个 `Query<T>` 类，它实现 `IQueryable<T>` 接口,提供了一个 `IQueryProvider` 和 `Expression` 属性。

```cs
public class Query<T> : IQueryable<T>, IQueryable, IEnumerable<T>, IEnumerable, IOrderedQueryable<T>, IOrderedQueryable
{
    private readonly QueryProvider provider;

    private readonly Expression expression;

    public Query(QueryProvider provider)
    {
        this.provider = provider ?? throw new ArgumentNullException(nameof(provider));

        this.expression = Expression.Constant(this);
    }

    public Query(QueryProvider provider, Expression expression)
    {
        if (expression == null)
        {
            throw new ArgumentNullException(nameof(expression));
        }

        if (!typeof(IQueryable<T>).IsAssignableFrom(expression.Type))
        {
            throw new ArgumentOutOfRangeException(nameof(expression));
        }

        this.provider = provider ?? throw new ArgumentNullException(nameof(provider));

        this.expression = expression;
    }

    Expression IQueryable.Expression => this.expression;

    Type IQueryable.ElementType => typeof(T);

    IQueryProvider IQueryable.Provider => this.provider;

    public IEnumerator<T> GetEnumerator()
    {
        return ((IEnumerable<T>)this.provider.Execute(this.expression)).GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return ((IEnumerable)this.provider.Execute(this.expression)).GetEnumerator();
    }

    public override string ToString()
    {
        return this.provider.GetQueryText(this.expression);
    }
}
```

然后，定义一个抽象类 `QueryProvider`，它将会实现 `IQueryProvider` 接口，包括 `CreateQuery` 和 `Execute`方法（都包含泛型方法）。

```cs
public abstract class QueryProvider : IQueryProvider
{
    protected QueryProvider()
    {
    }

    IQueryable<S> IQueryProvider.CreateQuery<S>(Expression expression)
    {
        return new Query<S>(this, expression);
    }

    IQueryable IQueryProvider.CreateQuery(Expression expression)
    {
        Type elementType = TypeSystem.GetElementType(expression.Type);

        try
        {
            return (IQueryable)Activator.CreateInstance(typeof(Query<>).MakeGenericType(elementType), new object[] { this, expression });
        }
        catch (TargetInvocationException tie)
        {
            throw tie.InnerException;
        }
    }

    S IQueryProvider.Execute<S>(Expression expression)
    {
        return (S)this.Execute(expression);
    }

    object IQueryProvider.Execute(Expression expression)
    {
        return this.Execute(expression);
    }

    public abstract string GetQueryText(Expression expression);

    public abstract object Execute(Expression expression);
}
```

最后定义一个具体 `Provider`，继承自抽象类 `QueryProvider`；

```cs
public class CnblogsQueryProvider : QueryProvider
{
    public override String GetQueryText(Expression expression)
    {
        SearchCriteria criteria;

        // 翻译查询条件
        criteria = new PostExpressionVisitor().ProcessExpression(expression);

        // 生成URL
        String url = PostHelper.BuildUrl(criteria);

        return url;
    }

    public override object Execute(Expression expression)
    {
        String url = GetQueryText(expression);
        IEnumerable<Post> results = PostHelper.PerformWebQuery(url);
        return results;
    }
}
```

调用：

```cs
static void Main(string[] args)
{
    var provider = new CnblogsQueryProvider();
    var queryable = new Query<Post>(provider);

    var query =
        from p in queryable
        where p.Diggs >= 10 &&
              p.Comments > 10 &&
              p.Views > 10 &&
              p.Comments < 20
        select p;

    var list = query.ToList();


    Console.ReadLine();
}
```

源码：[https://github.com/syxdevcode/LinqToCnBlogs](https://github.com/syxdevcode/LinqToCnBlogs)

参考：

[打造自己的LINQ Provider（中）：IQueryable和IQueryProvider](http://www.cnblogs.com/Terrylee/archive/2008/08/25/custom-linq-provider-part-2-IQueryable-IQueryProvider.html)

[由浅入深表达式树（完结篇）重磅打造 Linq To 博客园](http://www.cnblogs.com/jesse2013/p/expressiontree-Linq-to-cnblogs.html)

[打造自己的LINQ Provider（中）：IQueryable和IQueryProvider](http://www.cnblogs.com/Terrylee/archive/2008/08/25/custom-linq-provider-part-2-IQueryable-IQueryProvider.html)

[LINQ: Building an IQueryable Provider – Part I](https://blogs.msdn.microsoft.com/mattwar/2007/07/30/linq-building-an-iqueryable-provider-part-i/)