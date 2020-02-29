---
title: Emit基本操作
date: 2017-03-02 16:32:27
tags: 
- 反射
- CSharp基础
- Emit(反射发出)
categories: 
- Emit(反射发出)
---
## ILGenerator

ILGenerator：生成IL代码

获取ILGenerator类的实例方法：

> ConstructorBuilder.GetILGenerator方法
> DynamicMethod.GetILGenerator 方法
> MethodBuilder.GetILGenerator 方法

上面涉及到了在Emit中能够动态生成方法的三种途径，

ConstructorBuilder类用来配置的构造函数，构造函数内部的IL要使用它的

GetILGenerator方法返回的ILGenerator类发出。

DynamicMethod类，是在当前运行上下文环境中动态生成方法的类，使用该类不必

事先创建程序集、模块和类型，同样发出其内部的IL使用DynamicMethod.GetILGenerator

方法返回的ILGenerator类实例.

最终实现的调用效果：

```csharp
Mock<IAssessmentAopAdviceProvider> mocker = new Mock<IAssessmentAopAdviceProvider>();

mocker.Setup(t => t.Before(3)).Returns("Hello World!");

Console.WriteLine(mocker.Obj.Before(2));

Console.Read();

```

接收一个接口，初始化一个Mock类的实例，然后通过Setup和Returns扩展方法设定实现

该接口的实例在指定方法上的返回值。这里我们先不考虑对不同参数的处理逻辑。

Mock类定义如下：

```csharp
public class Mock<T> where T : IAssessmentAopAdviceProvider
{
    public T Obj
    {
        get; set;
    }

    public SetupContext Contex { get; set; }

    public Mock()
    {
    }
}
```

Mock类的Obj属性是特定接口的实例。Contex属性是上下文信息

Contex类定义如下：

```csharp
public class SetupContext
{
    public MethodInfo MethodInfo { get; set; }
}
```

只包含一个MethodInfo属性

扩展方法：

```csharp
public static class MockExtention
{
    public static Mock<T> Setup<T>(this Mock<T> mocker, Expression<Action<T>> expression) where T : IAssessmentAopAdviceProvider
    {
        mocker.Contex = new SetupContext();

        mocker.Contex.MethodInfo = expression.ToMethodInfo();

        return mocker;
    }

    public static void Returns<T>(this Mock<T> mocker, object returnValue) where T : IAssessmentAopAdviceProvider
    {
        if (mocker.Contex != null && mocker.Contex.MethodInfo != null)
        {
            //这里为简单起见，只考虑IAssessmentAopAdviceProvider接口
            mocker.Obj = (T)AdviceProviderFactory.GetProvider(mocker.Contex.MethodInfo.Name, (string)returnValue);
        }
    }

    public static MethodInfo ToMethodInfo(this LambdaExpression expression)
    {
        var memberExpression = expression.Body as System.Linq.Expressions.MethodCallExpression;

        if (memberExpression != null)

        {
            return memberExpression.Method;
        }

        return null;
    }

}
```

Setup是Mock类的扩展方法，配置要Mock的方法信息;

Returns扩展方法则调去对应的工厂获取接口的实例。

ToMethodInfo是LambdaExpression扩展方法，该方法从Lambda表达式中获取MethodInfo.

对应的工厂方法直接返回IAssessmentAopAdviceProvider接口实例。

在构造方法中初始化assemblyBuilder和moduleBuilder
代码：

```csharp
private static readonly AssemblyName assemblyName = new AssemblyName("EmitTest.MvcAdviceProvider");

private static AssemblyBuilder assemblyBuilder;

private static ModuleBuilder moduleBuilder;

static AdviceProviderFactory()
{
    assemblyName.Version = new Version("1.0.0.0");

    assemblyBuilder = AppDomain.CurrentDomain.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.RunAndSave);

    moduleBuilder = assemblyBuilder.DefineDynamicModule("MvcAdviceProviderModule", "test.dll", true);
}
```

CreateInstance方法负责创建类型和方法的实现：

```csharp
private static IAssessmentAopAdviceProvider CreateInstance(string instanceName, string methodName, string returnValue)
{
    TypeBuilder typeBuilder = moduleBuilder.DefineType("MvcAdviceProvider.MvcAdviceProviderType", TypeAttributes.Public, typeof(object), new Type[] { typeof(IAssessmentAopAdviceProvider) });

    // typeBuilder.AddInterfaceImplementation(typeof(IAssessmentAopAdviceProvider));

    MethodBuilder beforeMethodBuilder = typeBuilder.DefineMethod(methodName, MethodAttributes.Public | MethodAttributes.Virtual, typeof(string), new Type[] { typeof(int) });

    beforeMethodBuilder.DefineParameter(1, ParameterAttributes.None, "value");

    ILGenerator generator1 = beforeMethodBuilder.GetILGenerator();

    LocalBuilder local1 = generator1.DeclareLocal(typeof(string));

    local1.SetLocalSymInfo("param1");

    generator1.Emit(OpCodes.Nop);

    generator1.Emit(OpCodes.Ldstr, returnValue);

    generator1.Emit(OpCodes.Stloc_0);

    generator1.Emit(OpCodes.Ldloc_0);

    generator1.Emit(OpCodes.Ret);

    Type providerType = typeBuilder.CreateType();

    assemblyBuilder.Save("test.dll");

    IAssessmentAopAdviceProvider provider = Activator.CreateInstance(providerType) as IAssessmentAopAdviceProvider;

    return provider;
}
```

在CreateInstance方法中，首先定义类型：

```csharp
TypeBuilder typeBuilder = moduleBuilder.DefineType("MvcAdviceProvider.MvcAdviceProviderType", TypeAttributes.Public, typeof(object), new Type[] { typeof(IAssessmentAopAdviceProvider) });
```

第三个和第四个参数，分别是该类型大的基类型和实现的接口列表。

```csharp
MethodBuilder beforeMethodBuilder = typeBuilder.DefineMethod(methodName, MethodAttributes.Public | MethodAttributes.Virtual, typeof(string), new Type[] { typeof(int) });
```

DefineMethod方法中，传人的第一个参数是方法的名称，第二个参数是访问类型，第三个参数是修饰符，第四个参数是方法的参数类型列表。这里需要注意第二个参数，也就是方法的修饰符，因为接口中的方法定义都是virtual的，所以在实现接口的时候，方法也必须声明为MethodAttributes.Virtual。

定义方法的参数，使用MethodBuilder.DefineParameter方法。

```csharp
beforeMethodBuilder.DefineParameter(1, ParameterAttributes.None, "value");
```

DefineParameter方法的第一参数指定当前定义的参数在方法参数列表中的顺序，从1开始，如果设置为0则代表方法的返回参数。第二个参数是设置参数的特性，如输入参数，输出参数等等。第三个参数是指定该参数的名称。

方法定义完毕，接下来就是发出Opcode，返回一个指定的字符串。先获取ILGenerator实例，如下：

```csharp
ILGenerator generator1 = beforeMethodBuilder.GetILGenerator();
```

我们若想返回一个字符串，必须先为该字符串声明一个局部变量，可以使用LocalBuilder.DeclareLocal方法，如下：

```csharp
LocalBuilder local1 = generator1.DeclareLocal(typeof(string));

local1.SetLocalSymInfo("param1");
```

ocal1.SetLocalSymInfo("param1")指定局部变量的名称。

需要注意，如果模块定义时不允许发出符号信息，是无法使用SetLocalSymInfo方法的，AdviceProviderFactory的构造函数中，我们定义模块的代码

```csharp
moduleBuilder = assemblyBuilder.DefineDynamicModule("MvcAdviceProviderModule", "test.dll",true);
```

最后一个参数是指定是否允许发出符号信息的。

发出的Opcode:

```csharp
generator1.Emit(OpCodes.Nop);

generator1.Emit(OpCodes.Ldstr, returnValue);

generator1.Emit(OpCodes.Stloc_0);

generator1.Emit(OpCodes.Ldloc_0);

generator1.Emit(OpCodes.Ret);
```

第一条指令不执行任何操作。

第二条指令加载一个字符串到计算堆栈中。

第三条指令赋值计算堆栈顶部的数据到局部变量列表中的第一个变量。

第四条指令加载局部变量列表中的第一个变量的数据引用到计算堆栈。

第五条指令方法返回。

整个Emit的过程结束了，接下来要创建实例：

```csharp
Type providerType = typeBuilder.CreateType();

assemblyBuilder.Save("test.dll");

IAssessmentAopAdviceProvider provider = Activator.CreateInstance(providerType) as IAssessmentAopAdviceProvider;
```

控制台调用：

```csharp
Mock<IAssessmentAopAdviceProvider> mocker = new Mock<IAssessmentAopAdviceProvider>();

mocker.Setup(t => t.Before(3)).Returns("Hello World!");

Console.WriteLine(mocker.Obj.Before(2));

Console.Read();
```

[代码](https://github.com/syxdevcode/EmitTest "github代码")

参考：

[http://www.cnblogs.com/xuanhun/archive/2012/06/22/2558698.html](http://www.cnblogs.com/xuanhun/archive/2012/06/22/2558698.html "Emit详解")