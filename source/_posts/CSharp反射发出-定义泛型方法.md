---
title: CSharp反射发出-定义泛型方法
date: 2019-01-17 15:29:32
tags:
- DotNet
- CSharp
- Emit(反射发出)
categories: 
- Emit(反射发出)
---
<font color=#ff0000 size=4 face="黑体">某方法只要属于泛型类型，且使用该类型的类型参数，就不是泛型方法。 只有当方法有属于自己的类型参数列表时才是泛型方法。</font>

参考：[用反射发出定义泛型方法](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-define-a-generic-method-with-reflection-emit)

[动态类型生成的可回收程序集](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/collectible-assemblies)

<font color=#ff0000 size=4 face="黑体">如果发出对泛型类型方法的调用，并且这些类型的类型参数是泛型方法的类型参数，则必须使用 TypeBuilder 类的 staticGetConstructor(Type, ConstructorInfo)、GetMethod(Type, MethodInfo) 和 GetField(Type, FieldInfo) 方法重载来获取方法的构造窗体。</font>

```cs
namespace ConsoleApp1
{
    using System;
    using System.Collections.Generic;
    using System.Reflection;
    using System.Reflection.Emit;

    // Declare a generic delegate that can be used to execute the 
    // finished method.
    //
    public delegate TOut D<TIn, TOut>(TIn[] input);

    class GenericMethodBuilder
    {
        // This method shows how to declare, in Visual Basic, the generic
        // method this program emits. The method has two type parameters,
        // TInput and TOutput, the second of which must be a reference type
        // (class), must have a parameterless constructor (new()), and must
        // implement ICollection<TInput>. This interface constraint
        // ensures that ICollection<TInput>.Add can be used to add
        // elements to the TOutput object the method creates. The method 
        // has one formal parameter, input, which is an array of TInput. 
        // The elements of this array are copied to the new TOutput.
        // 1,该方法具有两个类型参数：TInput 和 TOutput。
        // 其中第二个参数必须具备以下条件：是引用类型 (class)；具有无参数构造函数 (new)；
        // 实现 ICollection(Of TInput)（在 C# 中为 ICollection<TInput>）。
        // 此接口约束确保可使用 ICollection<T>.Add 方法将元素添加到该方法
        // 创建的 TOutput 集合。 该方法具有一个形参 input，它是 TInput 的数组。
        // 该方法将创建一个 TOutput 类型的集合，并将 input 的元素复制到该集合。
        public static TOutput Factory<TInput, TOutput>(TInput[] tarray)
            where TOutput : class, ICollection<TInput>, new()
        {
            TOutput ret = new TOutput();
            ICollection<TInput> ic = ret;

            foreach (TInput t in tarray)
            {
                ic.Add(t);
            }
            return ret;
        }

        public static void Main()
        {
            // The following shows the usage syntax of the C#
            // version of the generic method emitted by this program.
            // Note that the generic parameters must be specified 
            // explicitly, because the compiler does not have enough 
            // context to infer the type of TOutput. In this case, TOutput
            // is a generic List containing strings.
            // 
            string[] arr = { "a", "b", "c", "d", "e" };
            List<string> list1 =
                GenericMethodBuilder.Factory<string, List<string>>(arr);
            Console.WriteLine("The first element is: {0}", list1[0]);


            // Creating a dynamic assembly requires an AssemblyName
            // object, and the current application domain.
            // 2,定义动态程序集和动态模块，以包含泛型方法所属类型。
            AssemblyName asmName = new AssemblyName("DemoMethodBuilder1");
            AppDomain domain = AppDomain.CurrentDomain;
            AssemblyBuilder demoAssembly =
                domain.DefineDynamicAssembly(asmName,
                    AssemblyBuilderAccess.RunAndSave);

            // Define the module that contains the code. For an 
            // assembly with one module, the module name is the 
            // assembly name plus a file extension.
            ModuleBuilder demoModule =
                demoAssembly.DefineDynamicModule(asmName.Name,
                    asmName.Name + ".dll");

            // Define a type to contain the method.
            // 3,定义泛型方法所属类型。 该类型不一定是泛型类型。
            // 泛型方法可以属于泛型或非泛型类型。
            TypeBuilder demoType =
                demoModule.DefineType("DemoType", TypeAttributes.Public);

            // Define a public static method with standard calling
            // conventions. Do not specify the parameter types or the
            // return type, because type parameters will be used for 
            // those types, and the type parameters have not been
            // defined yet.
            // 4,定义泛型方法。 
            MethodBuilder factory =
                demoType.DefineMethod("Factory",
                    MethodAttributes.Public | MethodAttributes.Static);

            // Defining generic type parameters for the method makes it a
            // generic method. To make the code easier to read, each
            // type parameter is copied to a variable of the same name.
            // 4,通过将包含参数名称的字符串数组传递给 MethodBuilder.DefineGenericParameters
            // 方法来定义 DemoMethod 的泛型类型参数。 这使该方法成为泛型方法。
            string[] typeParameterNames = { "TInput", "TOutput" };
            GenericTypeParameterBuilder[] typeParameters =
                factory.DefineGenericParameters(typeParameterNames);

            GenericTypeParameterBuilder TInput = typeParameters[0];
            GenericTypeParameterBuilder TOutput = typeParameters[1];

            // Add special constraints.
            // The type parameter TOutput is constrained to be a reference
            // type, and to have a parameterless constructor. This ensures
            // that the Factory method can create the collection type.
            // 6,可以选择向类型参数添加特殊约束。
            // 使用 SetGenericParameterAttributes 方法添加特殊约束。
            // 约束 TOutput 作为引用类型并且具有无参数构造函数。
            TOutput.SetGenericParameterAttributes(
                GenericParameterAttributes.ReferenceTypeConstraint |
                GenericParameterAttributes.DefaultConstructorConstraint);

            // Add interface and base type constraints.
            // The type parameter TOutput is constrained to types that
            // implement the ICollection<T> interface, to ensure that
            // they have an Add method that can be used to add elements.
            //
            // To create the constraint, first use MakeGenericType to bind 
            // the type parameter TInput to the ICollection<T> interface,
            // returning the type ICollection<TInput>, then pass
            // the newly created type to the SetInterfaceConstraints
            // method. The constraints must be passed as an array, even if
            // there is only one interface.
            // 7,可以选择向类型参数添加类约束和接口约束。
            // 约束类型参数 TOutput 为实现 ICollection<TInput>）接口的类型。
            Type icoll = typeof(ICollection<>);
            Type icollOfTInput = icoll.MakeGenericType(TInput);
            Type[] constraints = { icollOfTInput };
            TOutput.SetInterfaceConstraints(constraints);

            // Set parameter types for the method. The method takes
            // one parameter, an array of type TInput.
            // 8,使用 SetParameters 方法定义方法的形参。
            Type[] parms = { TInput.MakeArrayType() };
            factory.SetParameters(parms);

            // Set the return type for the method. The return type is
            // the generic type parameter TOutput.
            // 9,使用 SetReturnType 方法定义该方法的返回类型。
            factory.SetReturnType(TOutput);

            // Generate a code body for the method. 
            // -----------------------------------
            // Get a code generator and declare local variables and
            // labels. Save the input array to a local variable.
            // 10，使用 ILGenerator 发出方法主体。

            // (1),获取代码生成器，并声明局部变量和标签。
            ILGenerator ilgen = factory.GetILGenerator();

            //用于保留该方法返回的新 TOutput
            LocalBuilder retVal = ilgen.DeclareLocal(TOutput);

            //用于在 TOutput 强制转换成 ICollection<TInput> 时进行保留；
            LocalBuilder ic = ilgen.DeclareLocal(icollOfTInput);

            // 用于保留 TInput 对象的输入数组；
            LocalBuilder input = ilgen.DeclareLocal(TInput.MakeArrayType());

            //用于循环访问数组
            LocalBuilder index = ilgen.DeclareLocal(typeof(int));

            // 用于进入循环 (enterLoop)
            Label enterLoop = ilgen.DefineLabel();

            //用于循环的顶部 (loopAgain)
            Label loopAgain = ilgen.DefineLabel();

            ilgen.Emit(OpCodes.Ldarg_0);
            ilgen.Emit(OpCodes.Stloc_S, input);//将参数存储到局部变量 input

            // Create an instance of TOutput, using the generic method 
            // overload of the Activator.CreateInstance method. 
            // Using this overload requires the specified type to have
            // a parameterless constructor, which is the reason for adding 
            // that constraint to TOutput. Create the constructed generic
            // method by passing TOutput to MakeGenericMethod. After
            // emitting code to call the method, emit code to store the
            // new TOutput in a local variable. 
            // (2),使用 Activator.CreateInstance 方法的泛型方法重载，
            // 发出代码，创建 TOutput 的实例。
            // TOutput ret = new TOutput();
            MethodInfo createInst =
                typeof(Activator).GetMethod("CreateInstance", Type.EmptyTypes);
            MethodInfo createInstOfTOutput =
                createInst.MakeGenericMethod(TOutput);// 创建构造泛型方法

            ilgen.Emit(OpCodes.Call, createInstOfTOutput);
            //发出代码调用方法后,将其存储在局部变量 retVal
            ilgen.Emit(OpCodes.Stloc_S, retVal);

            // Load the reference to the TOutput object, cast it to
            // ICollection<TInput>, and save it.
            // (3),发出代码，将新的 TOutput 对象强制转换为 ICollection<TInput>，
            // 并将其存储在局部变量 ic。
            // ICollection<TInput> ic = ret;
            ilgen.Emit(OpCodes.Ldloc_S, retVal);
            ilgen.Emit(OpCodes.Box, TOutput);
            ilgen.Emit(OpCodes.Castclass, icollOfTInput);
            ilgen.Emit(OpCodes.Stloc_S, ic);

            // Loop through the array, adding each element to the new
            // instance of TOutput. Note that in order to get a MethodInfo
            // for ICollection<TInput>.Add, it is necessary to first 
            // get the Add method for the generic type defintion,
            // ICollection<T>.Add. This is because it is not possible
            // to call GetMethod on icollOfTInput. The static overload of
            // TypeBuilder.GetMethod produces the correct MethodInfo for
            // the constructed type.
            // (4),获取表示 ICollection<T>.Add 方法的 MethodInfo。 
            MethodInfo mAddPrep = icoll.GetMethod("Add");
            MethodInfo mAdd = TypeBuilder.GetMethod(icollOfTInput, mAddPrep);

            // Initialize the count and enter the loop.
            // (5),通过加载 32 位整数 0 并将其存储在变量中，发出代码，初始化 index 变量。
            ilgen.Emit(OpCodes.Ldc_I4_0);
            ilgen.Emit(OpCodes.Stloc_S, index); // index=0
            ilgen.Emit(OpCodes.Br_S, enterLoop); //发出代码，分支到标签 enterLoop

            // Mark the beginning of the loop. Push the ICollection
            // reference on the stack, so it will be in position for the
            // call to Add. Then push the array and the index on the 
            // stack, get the array element, and call Add (represented
            // by the MethodInfo mAdd) to add it to the collection.
            //
            // The other ten instructions just increment the index
            // and test for the end of the loop. Note the MarkLabel
            // method, which sets the point in the code where the 
            // loop is entered. (See the earlier Br_S to enterLoop.)
            // (6),发出该循环的代码。
            ilgen.MarkLabel(loopAgain);

            ilgen.Emit(OpCodes.Ldloc_S, ic);
            ilgen.Emit(OpCodes.Ldloc_S, input); // 数组
            ilgen.Emit(OpCodes.Ldloc_S, index); // 索引

            // Ldelem 操作码从堆栈中弹出该索引和数组，
            // 然后将索引数组元素推入堆栈。
            ilgen.Emit(OpCodes.Ldelem, TInput);
            // 堆栈现已准备好调用 ICollection<T>.Add 方法，
            // 该方法从堆栈中弹出集合和新元素，并将该元素添加到该集合中。
            ilgen.Emit(OpCodes.Callvirt, mAdd);

            // index+=1;
            ilgen.Emit(OpCodes.Ldloc_S, index);
            ilgen.Emit(OpCodes.Ldc_I4_1);
            ilgen.Emit(OpCodes.Add);
            ilgen.Emit(OpCodes.Stloc_S, index);

            // 调用 MarkLabel，将该点设置为循环的入口点。
            ilgen.MarkLabel(enterLoop);
            ilgen.Emit(OpCodes.Ldloc_S, index);// 加载该索引
            ilgen.Emit(OpCodes.Ldloc_S, input);// 将输入数组推入堆栈
            ilgen.Emit(OpCodes.Ldlen); // 获取其长度
            ilgen.Emit(OpCodes.Conv_I4);
            ilgen.Emit(OpCodes.Clt); // 二者进行比较
            ilgen.Emit(OpCodes.Brtrue_S, loopAgain);

            // (7),发出代码，将 TOutput 对象推入堆栈，并从该方法返回。
            ilgen.Emit(OpCodes.Ldloc_S, retVal);
            ilgen.Emit(OpCodes.Ret);

            // Complete the type.
            // 11,完成包含该方法的类型，并保存程序集。
            Type dt = demoType.CreateType();
            // Save the assembly, so it can be examined with Ildasm.exe.
            demoAssembly.Save(asmName.Name + ".dll");

            /*调用泛型方法*/

            // To create a constructed generic method that can be
            // executed, first call the GetMethod method on the completed 
            // type to get the generic method definition. Call MakeGenericType
            // on the generic method definition to obtain the constructed
            // method, passing in the type arguments. In this case, the
            // constructed method has string for TInput and List<string>
            // for TOutput. 
            // 1，分配泛型类型参数
            MethodInfo m = dt.GetMethod("Factory");
            MethodInfo bound =
                m.MakeGenericMethod(typeof(string), typeof(List<string>));

            // Display a string representing the bound method.
            Console.WriteLine(bound);


            // Once the generic method is constructed, 
            // you can invoke it and pass in an array of objects 
            // representing the arguments. In this case, there is only
            // one element in that array, the argument 'arr'.
            // 2，若要以后期绑定的形式调用该方法，请使用 Invoke 方法。
            // Factory为静态方法
            object o = bound.Invoke(null, new object[] { arr });
            List<string> list2 = (List<string>)o;

            Console.WriteLine("The first element is: {0}", list2[0]);


            // You can get better performance from multiple calls if
            // you bind the constructed method to a delegate. The 
            // following code uses the generic delegate D defined 
            // earlier.
            //
            Type dType = typeof(D<string, List<string>>);
            D<string, List<string>> test;
            test = (D<string, List<string>>)
                Delegate.CreateDelegate(dType, bound);

            List<string> list3 = test(arr);
            Console.WriteLine("The first element is: {0}", list3[0]);

            Console.ReadKey();
        }
    }
}
```