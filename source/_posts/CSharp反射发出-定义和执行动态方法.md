---
title: CSharp反射发出-定义和执行动态方法
date: 2019-01-15 09:12:26
tags:
- DotNet
- CSharp
- Emit(反射发出)
categories: 
- Emit(反射发出)
description: 详细信息，请参考https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.dynamicmethod?view=netframework-4.7.2
---
有关动态方法的更多信息，请参阅 [DynamicMethod](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.dynamicmethod?view=netframework-4.7.2) 类。

[OpCodes](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes?view=netframework-4.7.2)

## 定义和执行动态方法

```cs
using System;
using System.Reflection;
using System.Reflection.Emit;

public class Example
{
    // Declare delegates that can be used to execute the completed 
    // SquareIt dynamic method. The OneParameter delegate can be 
    // used to execute any method with one parameter and a return
    // value, or a method with two parameters and a return value
    // if the delegate is bound to an object.
    // 1,声明用于执行方法的委托类型。
    private delegate long SquareItInvoker(int input);

    private delegate TReturn OneParameter<TReturn, TParameter0>
        (TParameter0 p0);
    public static void Main()
    {
        // Example 1: A simple dynamic method.
        //
        // Create an array that specifies the parameter types for the
        // dynamic method. In this example the only parameter is an 
        // int, so the array has only one element.
        // 2,创建用于为动态方法指定参数类型的数组。
        Type[] methodArgs = { typeof(int) };

        // Create a DynamicMethod. In this example the method is
        // named SquareIt. It is not necessary to give dynamic 
        // methods names. They cannot be invoked by name, and two
        // dynamic methods can have the same name. However, the 
        // name appears in calls stacks and can be useful for
        // debugging. 
        //
        // In this example the return type of the dynamic method
        // is long. The method is associated with the module that 
        // contains the Example class. Any loaded module could be
        // specified. The dynamic method is like a module-level
        // static method.
        // 3,创建 DynamicMethod。
        DynamicMethod squareIt = new DynamicMethod(
            "SquareIt",
            typeof(long),
            methodArgs,
            typeof(Example).Module);

        // Emit the method body. In this example ILGenerator is used
        // to emit the MSIL. DynamicMethod has an associated type
        // DynamicILInfo that can be used in conjunction with 
        // unmanaged code generators.
        //
        // The MSIL loads the argument, which is an int, onto the 
        // stack, converts the int to a long, duplicates the top
        // item on the stack, and multiplies the top two items on the
        // stack. This leaves the squared number on the stack, and 
        // all the method has to do is return.
        // 4,发出方法主体。
        ILGenerator il = squareIt.GetILGenerator();
        il.Emit(OpCodes.Ldarg_0);
        il.Emit(OpCodes.Conv_I8);
        il.Emit(OpCodes.Dup);
        il.Emit(OpCodes.Mul);
        il.Emit(OpCodes.Ret);

        // Create a delegate that represents the dynamic method. 
        // Creating the delegate completes the method, and any further 
        // attempts to change the method (for example, by adding more
        // MSIL) are ignored. The following code uses a generic 
        // delegate that can produce delegate types matching any
        // single-parameter method that has a return type.
        // 5,通过调用 CreateDelegate 方法创建表示动态方法的委托（在步骤 1 中声明）的实例
        // 以下代码使用泛型委托创建委托并调用它。
        OneParameter<long, int> invokeSquareIt =
            (OneParameter<long, int>)
            squareIt.CreateDelegate(typeof(OneParameter<long, int>));
        // squared :平方
        Console.WriteLine("123456789 squared = {0}",
            invokeSquareIt(123456789));
        Console.ReadLine();
    }
}
/* This code example produces the following output:
123456789 squared = 15241578750190521
*/
```

## 定义和执行绑定到对象的动态方法

```cs
using System;
using System.Reflection;
using System.Reflection.Emit;

public class Example
{
    // The following constructor and private field are used to
    // demonstrate a method bound to an object.
    private int test;
    public Example(int test) { this.test = test; }

    // Declare delegates that can be used to execute the completed 
    // SquareIt dynamic method. The OneParameter delegate can be 
    // used to execute any method with one parameter and a return
    // value, or a method with two parameters and a return value
    // if the delegate is bound to an object.
    // 1,声明用于执行方法的委托类型。

    private delegate TReturn OneParameter<TReturn, TParameter0>
        (TParameter0 p0);
    public static void Main()
    {
        // Example 2: A dynamic method bound to an instance.
        //
        // Create an array that specifies the parameter types for a
        // dynamic method. If the delegate representing the method
        // is to be bound to an object, the first parameter must 
        // match the type the delegate is bound to. In the following
        // code the bound instance is of the Example class. 
        // 2,创建用于为动态方法指定参数类型的数组。
        Type[] methodArgs2 = { typeof(Example), typeof(int) };

        // Create a DynamicMethod. In this example the method has no
        // name. The return type of the method is int. The method 
        // has access to the protected and private data of the 
        // Example class.
        // 3,创建 DynamicMethod。
        DynamicMethod multiplyHidden = new DynamicMethod(
            "",
            typeof(int),
            methodArgs2,
            typeof(Example));

        // Emit the method body. In this example ILGenerator is used
        // to emit the MSIL. DynamicMethod has an associated type
        // DynamicILInfo that can be used in conjunction with 
        // unmanaged code generators.
        //
        // The MSIL loads the first argument, which is an instance of
        // the Example class, and uses it to load the value of a 
        // private instance field of type int. The second argument is
        // loaded, and the two numbers are multiplied. If the result
        // is larger than int, the value is truncated and the most 
        // significant bits are discarded. The method returns, with
        // the return value on the stack.
        // 4,发出方法主体。
        ILGenerator ilMH = multiplyHidden.GetILGenerator();
        ilMH.Emit(OpCodes.Ldarg_0);

        FieldInfo testInfo = typeof(Example).GetField("test",
            BindingFlags.NonPublic | BindingFlags.Instance);
        // 获取Example实例的test值，
        ilMH.Emit(OpCodes.Ldfld, testInfo);
        ilMH.Emit(OpCodes.Ldarg_1);
        ilMH.Emit(OpCodes.Mul);
        ilMH.Emit(OpCodes.Ret);

        // Create a delegate that represents the dynamic method. 
        // Creating the delegate completes the method, and any further 
        // attempts to change the method � for example, by adding more
        // MSIL � are ignored. 
        // 
        // The following code binds the method to a new instance
        // of the Example class whose private test field is set to 42.
        // That is, each time the delegate is invoked the instance of
        // Example is passed to the first parameter of the method.
        //
        // The delegate OneParameter is used, because the first
        // parameter of the method receives the instance of Example.
        // When the delegate is invoked, only the second parameter is
        // required. 
        // 5,通过调用 CreateDelegate(Type, Object) 方法重载创建表示动态方法的委托（在步骤 1 中声明）的实例。
        OneParameter<int, int> invoke = (OneParameter<int, int>)
            multiplyHidden.CreateDelegate(
                typeof(OneParameter<int, int>),
                new Example(42)
            );

        Console.WriteLine("3 * test = {0}", invoke(3));
        Console.ReadLine();
    }
}

/* This code example produces the following output:

3 * test = 126
*/
```

参考：[如何：定义和执行动态方法](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-define-and-execute-dynamic-methods)