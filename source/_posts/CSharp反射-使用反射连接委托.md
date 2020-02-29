---
title: CSharp反射-使用反射连接委托
date: 2019-01-13 16:10:21
tags:
- DotNet
- CSharp
- 反射
- 委托与事件
categories: 
- 反射
description: 使用反射加载和运行程序集时，不能使用 C# += 运算符将事件挂钩。
---
## 连接委托

参考：[使用反射连接委托](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-hook-up-a-delegate-using-reflection)

使用反射加载和运行程序集时，不能使用 C# += 运算符将事件挂钩。 以下代码示例介绍如何使用反射将现有方法挂钩到事件，以及如何使用 `DynamicMethod` 类在运行时发出方法并将其挂钩到事件。

```cs
using System;
using System.Reflection;
using System.Reflection.Emit;
using System.Windows.Forms;

namespace ConsoleApp3
{
    internal class ExampleForm : Form
    {
        public ExampleForm() : base()
        {
            this.Text = "Click me";
        }
    }

    internal class Example
    {
        public static void Main()
        {
            Example ex = new Example();
            ex.HookUpDelegate();
        }

        private void HookUpDelegate()
        {
            // Load an assembly, for example using the Assembly.Load
            // method. In this case, the executing assembly is loaded, to
            // keep the demonstration simple.
            //
            Assembly assem = typeof(Example).Assembly;

            // Assembly assem1 = Assembly.GetExecutingAssembly();

            // Get the type that is to be loaded, and create an instance
            // of it. Activator.CreateInstance has other overloads, if
            // the type lacks a default constructor. The new instance
            // is stored as type Object, to maintain the fiction that
            // nothing is known about the assembly. (Note that you can
            // get the types in an assembly without knowing their names
            // in advance.)
            //
            Type tExForm = assem.GetType("ConsoleApp3.ExampleForm");

            // Type tExForm1 = typeof(ExampleForm);

            Object exFormAsObj = Activator.CreateInstance(tExForm);

            // Get an EventInfo representing the Click event, and get the
            // type of delegate that handles the event.
            //
            EventInfo evClick = tExForm.GetEvent("Click");
            Type tDelegate = evClick.EventHandlerType;

            // If you already have a method with the correct signature,
            // you can simply get a MethodInfo for it.
            //
            MethodInfo miHandler =
                typeof(Example).GetMethod("LuckyHandler",
                    BindingFlags.NonPublic | BindingFlags.Instance);

            // Create an instance of the delegate. Using the overloads
            // of CreateDelegate that take MethodInfo is recommended.
            //
            Delegate d = Delegate.CreateDelegate(tDelegate, this, miHandler);

            // Get the "add" accessor of the event and invoke it late-
            // bound, passing in the delegate instance. This is equivalent
            // to using the += operator in C#, or AddHandler in Visual
            // Basic. The instance on which the "add" accessor is invoked
            // is the form; the arguments must be passed as an array.
            //
            MethodInfo addHandler = evClick.GetAddMethod();
            Object[] addHandlerArgs = { d };
            addHandler.Invoke(exFormAsObj,addHandlerArgs);

            // Event handler methods can also be generated at run time,
            // using lightweight dynamic methods and Reflection.Emit.
            // To construct an event handler, you need the return type
            // and parameter types of the delegate. These can be obtained
            // by examining the delegate's Invoke method.
            //
            // It is not necessary to name dynamic methods, so the empty
            // string can be used. The last argument associates the
            // dynamic method with the current type, giving the delegate
            // access to all the public and private members of Example,
            // as if it were an instance method.
            //
            Type returnType = GetDelegateReturnType(tDelegate);
            if (returnType != typeof(void))
                throw new ApplicationException("Delegate has a return type.");

            DynamicMethod handler =
                new DynamicMethod("",
                                  null,
                                  GetDelegateParameterTypes(tDelegate),
                                  typeof(Example));

            // Generate a method body. This method loads a string, calls
            // the Show method overload that takes a string, pops the
            // return value off the stack (because the handler has no
            // return type), and returns.
            //
            ILGenerator ilgen = handler.GetILGenerator();

            Type[] showParameters = { typeof(String) };
            MethodInfo simpleShow =
                typeof(MessageBox).GetMethod("Show", showParameters);

            ilgen.Emit(OpCodes.Ldstr,
                "This event handler was constructed at run time.");
            ilgen.Emit(OpCodes.Call, simpleShow);
            ilgen.Emit(OpCodes.Pop);
            ilgen.Emit(OpCodes.Ret);

            // Complete the dynamic method by calling its CreateDelegate
            // method. Use the "add" accessor to add the delegate to
            // the invocation list for the event.
            //
            Delegate dEmitted = handler.CreateDelegate(tDelegate);
            addHandler.Invoke(exFormAsObj, new Object[] { dEmitted });

            // Show the form. Clicking on the form causes the two
            // delegates to be invoked.
            //
            Application.Run((Form)exFormAsObj);
        }

        private void LuckyHandler(Object sender, EventArgs e)
        {
            MessageBox.Show("This event handler just happened to be lying around.");
        }

        private Type[] GetDelegateParameterTypes(Type d)
        {
            if (d.BaseType != typeof(MulticastDelegate))
                throw new ApplicationException("Not a delegate.");

            MethodInfo invoke = d.GetMethod("Invoke");
            if (invoke == null)
                throw new ApplicationException("Not a delegate.");

            ParameterInfo[] parameters = invoke.GetParameters();
            Type[] typeParameters = new Type[parameters.Length];
            for (int i = 0; i < parameters.Length; i++)
            {
                typeParameters[i] = parameters[i].ParameterType;
            }
            return typeParameters;
        }

        private Type GetDelegateReturnType(Type d)
        {
            if (d.BaseType != typeof(MulticastDelegate))
                throw new ApplicationException("Not a delegate.");

            MethodInfo invoke = d.GetMethod("Invoke");
            if (invoke == null)
                throw new ApplicationException("Not a delegate.");

            return invoke.ReturnType;
        }
    }
}
```

## 连接事件

参考：[EventInfo.AddEventHandler](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.eventinfo.addeventhandler?view=netframework-4.7.2)

```cs
namespace ConsoleApp1
{
    using System;
    using System.Reflection;
    using System.Reflection.Emit;

    public class Example
    {
        private static object timer;

        public static void Main()
        {
            // Get the Timer type.
            Type t = typeof(System.Timers.Timer);
            // Create an instance of the Timer type.
            timer = Activator.CreateInstance(t);

            // Use reflection to get the Elapsed event.
            EventInfo eInfo = t.GetEvent("Elapsed");

            // In order to create a method to handle the Elapsed event,
            // it is necessary to know the signature of the delegate
            // used to raise the event. Reflection.Emit can then be
            // used to construct a dynamic class with a static method
            // that has the correct signature.

            // Get the event handler type of the Elapsed event. This is
            // a delegate type, so it has an Invoke method that has
            // the same signature as the delegate. The following code
            // creates an array of Type objects that represent the
            // parameter types of the Invoke method.
            //
            Type handlerType = eInfo.EventHandlerType;
            MethodInfo invokeMethod = handlerType.GetMethod("Invoke");
            ParameterInfo[] parms = invokeMethod.GetParameters();
            Type[] parmTypes = new Type[parms.Length];
            for (int i = 0; i < parms.Length; i++)
            {
                parmTypes[i] = parms[i].ParameterType;
            }

            // Use Reflection.Emit to create a dynamic assembly that
            // will be run but not saved. An assembly must have at
            // least one module, which in this case contains a single
            // type. The only purpose of this type is to contain the
            // event handler method. (You can use also dynamic methods,
            // which are simpler because there is no need to create an
            // assembly, module, or type.)
            //
            AssemblyName aName = new AssemblyName();
            aName.Name = "DynamicTypes";
            AssemblyBuilder ab = AssemblyBuilder.DefineDynamicAssembly(aName, AssemblyBuilderAccess.Run);
            ModuleBuilder mb = ab.DefineDynamicModule(aName.Name);
            TypeBuilder tb = mb.DefineType("Handler", TypeAttributes.Class | TypeAttributes.Public);

            // Create the method that will handle the event. The name
            // is not important. The method is static, because there is
            // no reason to create an instance of the dynamic type.
            //
            // The parameter types and return type of the method are
            // the same as those of the delegate's Invoke method,
            // captured earlier.
            MethodBuilder handler = tb.DefineMethod("DynamicHandler",
                MethodAttributes.Public | MethodAttributes.Static,
                invokeMethod.ReturnType, parmTypes);

            // Generate code to handle the event. In this case, the
            // handler simply prints a text string to the console.
            //
            ILGenerator il = handler.GetILGenerator();
            il.EmitWriteLine("Timer's Elapsed event is raised.");
            il.Emit(OpCodes.Ret);

            // CreateType must be called before the Handler type can
            // be used. In order to create the delegate that will
            // handle the event, a MethodInfo from the finished type
            // is required.
            Type finished = tb.CreateType();
            MethodInfo eventHandler = finished.GetMethod("DynamicHandler");

            // Use the MethodInfo to create a delegate of the correct
            // type, and call the AddEventHandler method to hook up
            // the event.
            Delegate d = Delegate.CreateDelegate(handlerType, eventHandler);
            eInfo.AddEventHandler(timer, d);

            // Late-bound calls to the Interval and Enabled property
            // are required to enable the timer with a one-second
            // interval.
            t.InvokeMember("Interval", BindingFlags.SetProperty, null, timer, new Object[] { 1000 });
            t.InvokeMember("Enabled", BindingFlags.SetProperty, null, timer, new Object[] { true });

            Console.WriteLine("Press the Enter key to end the program.");
            Console.ReadLine();
        }
    }
}
/* This example produces output similar to the following:

Press the Enter key to end the program.
Timer's Elapsed event is raised.
Timer's Elapsed event is raised.
Timer's Elapsed event is raised.
*/
```