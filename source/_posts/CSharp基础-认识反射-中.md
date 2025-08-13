---
title: CSharp基础-认识反射-中
date: 2019-01-12 15:52:24
tags:
- DotNet
- CSharp
- 反射
- CSharp基础
categories: 
- 反射
---
# CSharp基础-认识反射-中

本文包括动态加载和使用类型，将程序集加载到仅反射上下文中；

## 动态加载和使用类型

### 自定义绑定

参考：[自定义绑定](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/dynamically-loading-and-using-types#custom-binding)

反射可在代码中显式用于完成后期绑定。在通过反射实现的后期绑定中，必须由自定义绑定来控制该绑定。 `Binder` 类提供对成员选择和调用的自定义控制。

使用自定义绑定可以在运行时加载程序集、获取该程序集中有关类型的信息、指定所需类型，并在之后调用该类型上的方法或访问该类型上的字段或属性。 如果在编译时不知道对象的类型，则此方法非常有用，例如当对象类型取决于用户输入时。

实例：

```cs
namespace ClassLibrary1
{
    public class MySimpleClass
    {
        public void MyMethod(string str, int i)
        {
            Console.WriteLine("MyMethod parameters: {0}, {1}", str, i);
        }

        public void MyMethod(string str, int i, int j)
        {
            Console.WriteLine("MyMethod parameters: {0}, {1}, {2}",
                str, i, j);
        }
    }
}

using ClassLibrary1;
using System;
using System.Globalization;
using System.Reflection;

namespace ConsoleApp1
{
    internal class MyMainClass
    {
        private static void Main()
        {
            // Get the type of MySimpleClass.
            Type myType = typeof(MySimpleClass);

            // Get an instance of MySimpleClass.
            MySimpleClass myInstance = new MySimpleClass();
            MyCustomBinder myCustomBinder = new MyCustomBinder();

            // Get the method information for the particular overload
            // being sought.
            MethodInfo myMethod = myType.GetMethod("MyMethod",
                BindingFlags.Public | BindingFlags.Instance,
                myCustomBinder, new Type[] {typeof(string),
                typeof(int)}, null);
            Console.WriteLine(myMethod.ToString());

            // Invoke the overload.
            myType.InvokeMember("MyMethod", BindingFlags.InvokeMethod,
                myCustomBinder, myInstance,
                new Object[] { "Testing...", (int)32 });

            Console.ReadKey();
        }
    }

    // ****************************************************
    //  A simple custom binder that provides no
    //  argument type conversion.
    // ****************************************************
    internal class MyCustomBinder : Binder
    {
        public override MethodBase BindToMethod(
            BindingFlags bindingAttr,
            MethodBase[] match,
            ref object[] args,
            ParameterModifier[] modifiers,
            CultureInfo culture,
            string[] names,
            out object state)
        {
            if (match == null)
            {
                throw new ArgumentNullException("match");
            }
            // Arguments are not being reordered.
            state = null;
            // Find a parameter match and return the first method with
            // parameters that match the request.
            foreach (MethodBase mb in match)
            {
                ParameterInfo[] parameters = mb.GetParameters();

                if (ParametersMatch(parameters, args))
                {
                    return mb;
                }
            }
            return null;
        }

        public override FieldInfo BindToField(BindingFlags bindingAttr,
            FieldInfo[] match, object value, CultureInfo culture)
        {
            if (match == null)
            {
                throw new ArgumentNullException("match");
            }
            foreach (FieldInfo fi in match)
            {
                if (fi.GetType() == value.GetType())
                {
                    return fi;
                }
            }
            return null;
        }

        public override MethodBase SelectMethod(
            BindingFlags bindingAttr,
            MethodBase[] match,
            Type[] types,
            ParameterModifier[] modifiers)
        {
            if (match == null)
            {
                throw new ArgumentNullException("match");
            }

            // Find a parameter match and return the first method with
            // parameters that match the request.
            foreach (MethodBase mb in match)
            {
                ParameterInfo[] parameters = mb.GetParameters();
                if (ParametersMatch(parameters, types))
                {
                    return mb;
                }
            }

            return null;
        }

        public override PropertyInfo SelectProperty(
            BindingFlags bindingAttr,
            PropertyInfo[] match,
            Type returnType,
            Type[] indexes,
            ParameterModifier[] modifiers)
        {
            if (match == null)
            {
                throw new ArgumentNullException("match");
            }
            foreach (PropertyInfo pi in match)
            {
                if (pi.GetType() == returnType &&
                    ParametersMatch(pi.GetIndexParameters(), indexes))
                {
                    return pi;
                }
            }
            return null;
        }

        public override object ChangeType(
            object value,
            Type myChangeType,
            CultureInfo culture)
        {
            try
            {
                object newType;
                newType = Convert.ChangeType(value, myChangeType);
                return newType;
            }
            // Throw an InvalidCastException if the conversion cannot
            // be done by the Convert.ChangeType method.
            catch (InvalidCastException)
            {
                return null;
            }
        }

        public override void ReorderArgumentArray(ref object[] args,
            object state)
        {
            // No operation is needed here because BindToMethod does not
            // reorder the args array. The most common implementation
            // of this method is shown below.

            // ((BinderState)state).args.CopyTo(args, 0);
        }

        // Returns true only if the type of each object in a matches
        // the type of each corresponding object in b.
        private bool ParametersMatch(ParameterInfo[] a, object[] b)
        {
            if (a.Length != b.Length)
            {
                return false;
            }
            for (int i = 0; i < a.Length; i++)
            {
                if (a[i].ParameterType != b[i].GetType())
                {
                    return false;
                }
            }
            return true;
        }

        // Returns true only if the type of each object in a matches
        // the type of each corresponding entry in b.
        private bool ParametersMatch(ParameterInfo[] a, Type[] b)
        {
            if (a.Length != b.Length)
            {
                return false;
            }
            for (int i = 0; i < a.Length; i++)
            {
                if (a[i].ParameterType != b[i])
                {
                    return false;
                }
            }
            return true;
        }
    }
}
```

### InvokeMember 和 CreateInstance

使用 `Type.InvokeMember` 调用类型的成员。`InvokeMember` 可以创建指定类型的新实例，而各种类的 `CreateInstance` 方法（例如 `Activator.CreateInstance` 和 `Assembly.CreateInstance`）是 `InvokeMember` 的专用形式。 `Binder` 类可在这些方法中用于重载解析和参数强制转换。

实例：

```cs
public class CustomBinderDriver
{
    public static void Main()
    {
        Type t = typeof(CustomBinderDriver);
        CustomBinder binder = new CustomBinder();
        BindingFlags flags = BindingFlags.InvokeMethod | BindingFlags.Instance |
            BindingFlags.Public | BindingFlags.Static;
        object[] args;

        // Case 1. Neither argument coercion nor member selection is needed.
        args = new object[] {};
        t.InvokeMember("PrintBob", flags, binder, null, args);

        // Case 2. Only member selection is needed.
        args = new object[] {42};
        t.InvokeMember("PrintValue", flags, binder, null, args);

        // Case 3. Only argument coercion is needed.
        args = new object[] {"5.5"};
        t.InvokeMember("PrintNumber", flags, binder, null, args);
    }

    public static void PrintBob()
    {
        Console.WriteLine("PrintBob");
    }

    public static void PrintValue(long value)
    {
        Console.WriteLine("PrintValue({0})", value);
    }

    public static void PrintValue(string value)
    {
        Console.WriteLine("PrintValue\"{0}\")", value);
    }

    public static void PrintNumber(double value)
    {
        Console.WriteLine("PrintNumber ({0})", value);
    }
}
```

## 将程序集加载到仅反射上下文中

参考：[如何：将程序集加载到仅反射上下文中](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-load-assemblies-into-the-reflection-only-context)

通过仅反射加载上下文，可检查为其他平台或 .NET Framework 的其他版本编译的程序集。 只能检查，不能执行加载到此上下文中的代码。 这意味着无法创建对象，因为无法执行构造函数。 因为该代码无法执行，所以不会自动加载依赖项。 如果该程序集具有依赖项，`ReflectionOnlyLoad` 方法不会加载这些依赖项。如果需要对依赖项进行检查，必须自行加载。使用 `CustomAttributeData.GetCustomAttributes` 方法的适当重载获取表示应用于程序集、成员、模块或参数的特性的 `CustomAttributeData` 对象。

```cs
using System;
using System.Reflection;
using System.Collections.Generic;
using System.Collections.ObjectModel;

// The example attribute is applied to the assembly.
[assembly:Example(ExampleKind.ThirdKind, Note="This is a note on the assembly.")]

// An enumeration used by the ExampleAttribute class.
public enum ExampleKind
{
    FirstKind, 
    SecondKind, 
    ThirdKind, 
    FourthKind
};

// An example attribute. The attribute can be applied to all
// targets, from assemblies to parameters.
//
[AttributeUsage(AttributeTargets.All)]
public class ExampleAttribute : Attribute
{
    // Data for properties.
    private ExampleKind kindValue;
    private string noteValue;
    private string[] arrayStrings;
    private int[] arrayNumbers;

    // Constructors. The parameterless constructor (.ctor) calls
    // the constructor that specifies ExampleKind and an array of 
    // strings, and supplies the default values.
    //
    public ExampleAttribute(ExampleKind initKind, string[] initStrings)
    {
        kindValue = initKind;
        arrayStrings = initStrings;
    }
    public ExampleAttribute(ExampleKind initKind) : this(initKind, null) {}
    public ExampleAttribute() : this(ExampleKind.FirstKind, null) {}

    // Properties. The Note and Numbers properties must be read/write, so they
    // can be used as named parameters.
    //
    public ExampleKind Kind { get { return kindValue; }}
    public string[] Strings { get { return arrayStrings; }}
    public string Note
    {
        get { return noteValue; }
        set { noteValue = value; }
    }
    public int[] Numbers
    {
        get { return arrayNumbers; }
        set { arrayNumbers = value; }
    }
}

// The example attribute is applied to the test class.
//
[Example(ExampleKind.SecondKind, 
         new string[] { "String array argument, line 1", 
                        "String array argument, line 2", 
                        "String array argument, line 3" }, 
         Note="This is a note on the class.",
         Numbers = new int[] { 53, 57, 59 })] 
public class Test
{
    // The example attribute is applied to a method, using the
    // parameterless constructor and supplying a named argument.
    // The attribute is also applied to the method parameter.
    //
    [Example(Note="This is a note on a method.")]
    public void TestMethod([Example] object arg) { }

    // Main() gets objects representing the assembly, the test
    // type, the test method, and the method parameter. Custom
    // attribute data is displayed for each of these.
    //
    public static void Main()
    {
        Assembly asm = Assembly.ReflectionOnlyLoad("Source");
        Type t = asm.GetType("Test");
        MethodInfo m = t.GetMethod("TestMethod");
        ParameterInfo[] p = m.GetParameters();

        Console.WriteLine("\r\nAttributes for assembly: '{0}'", asm);
        ShowAttributeData(CustomAttributeData.GetCustomAttributes(asm));
        Console.WriteLine("\r\nAttributes for type: '{0}'", t);
        ShowAttributeData(CustomAttributeData.GetCustomAttributes(t));
        Console.WriteLine("\r\nAttributes for member: '{0}'", m);
        ShowAttributeData(CustomAttributeData.GetCustomAttributes(m));
        Console.WriteLine("\r\nAttributes for parameter: '{0}'", p);
        ShowAttributeData(CustomAttributeData.GetCustomAttributes(p[0]));
    }

    private static void ShowAttributeData(
        IList<CustomAttributeData> attributes)
    {
        foreach( CustomAttributeData cad in attributes )
        {
            Console.WriteLine("   {0}", cad);
            Console.WriteLine("      Constructor: '{0}'", cad.Constructor);

            Console.WriteLine("      Constructor arguments:");
            foreach( CustomAttributeTypedArgument cata 
                in cad.ConstructorArguments )
            {
                ShowValueOrArray(cata);
            }

            Console.WriteLine("      Named arguments:");
            foreach( CustomAttributeNamedArgument cana 
                in cad.NamedArguments )
            {
                Console.WriteLine("         MemberInfo: '{0}'", 
                    cana.MemberInfo);
                ShowValueOrArray(cana.TypedValue);
            }
        }
    }

    private static void ShowValueOrArray(CustomAttributeTypedArgument cata)
    {
        if (cata.Value.GetType() == typeof(ReadOnlyCollection<CustomAttributeTypedArgument>))
        {
            Console.WriteLine("         Array of '{0}':", cata.ArgumentType);

            foreach (CustomAttributeTypedArgument cataElement in 
                (ReadOnlyCollection<CustomAttributeTypedArgument>) cata.Value)
            {
                Console.WriteLine("             Type: '{0}'  Value: '{1}'",
                    cataElement.ArgumentType, cataElement.Value);
            }
        }
        else
        {
            Console.WriteLine("         Type: '{0}'  Value: '{1}'", 
                cata.ArgumentType, cata.Value);
        }
    }
}

/* This code example produces output similar to the following:

Attributes for assembly: 'source, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null'
   [System.Runtime.CompilerServices.CompilationRelaxationsAttribute((Int32)8)]
      Constructor: 'Void .ctor(Int32)'
      Constructor arguments:
         Type: 'System.Int32'  Value: '8'
      Named arguments:
   [System.Runtime.CompilerServices.RuntimeCompatibilityAttribute(WrapNonExceptionThrows = True)]
      Constructor: 'Void .ctor()'
      Constructor arguments:
      Named arguments:
         MemberInfo: 'Boolean WrapNonExceptionThrows'
         Type: 'System.Boolean'  Value: 'True'
   [ExampleAttribute((ExampleKind)2, Note = "This is a note on the assembly.")]
      Constructor: 'Void .ctor(ExampleKind)'
      Constructor arguments:
         Type: 'ExampleKind'  Value: '2'
      Named arguments:
         MemberInfo: 'System.String Note'
         Type: 'System.String'  Value: 'This is a note on the assembly.'

Attributes for type: 'Test'
   [ExampleAttribute((ExampleKind)1, new String[3] { "String array argument, line 1", "String array argument, line 2", "String array argument, line 3" }, Note = "This is a note on the class.", Numbers = new Int32[3] { 53, 57, 59 })]
      Constructor: 'Void .ctor(ExampleKind, System.String[])'
      Constructor arguments:
         Type: 'ExampleKind'  Value: '1'
         Array of 'System.String[]':
             Type: 'System.String'  Value: 'String array argument, line 1'
             Type: 'System.String'  Value: 'String array argument, line 2'
             Type: 'System.String'  Value: 'String array argument, line 3'
      Named arguments:
         MemberInfo: 'System.String Note'
         Type: 'System.String'  Value: 'This is a note on the class.'
         MemberInfo: 'Int32[] Numbers'
         Array of 'System.Int32[]':
             Type: 'System.Int32'  Value: '53'
             Type: 'System.Int32'  Value: '57'
             Type: 'System.Int32'  Value: '59'

Attributes for member: 'Void TestMethod(System.Object)'
   [ExampleAttribute(Note = "This is a note on a method.")]
      Constructor: 'Void .ctor()'
      Constructor arguments:
      Named arguments:
         MemberInfo: 'System.String Note'
         Type: 'System.String'  Value: 'This is a note on a method.'

Attributes for parameter: 'System.Object arg'
   [ExampleAttribute()]
      Constructor: 'Void .ctor()'
      Constructor arguments:
      Named arguments:
*/
```