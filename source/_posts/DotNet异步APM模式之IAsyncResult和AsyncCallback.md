---
title: DotNet异步APM模式之IAsyncResult和AsyncCallback
date: 2018-07-24 14:21:17
tags:
- DotNet
- CSharp
- .net异步与并行
categories: 
- .net异步与并行
---

IAsyncResult 异步设计模式通过名为 BeginOperationName 和 EndOperationName 的两个方法来实现原同步方法的异步调用，如 FileStream 类提供了 BeginRead 和 EndRead 方法来从文件异步读取字节，它们是 Read 方法的异步版本。

**Begin方法包含同步方法签名中的任何参数**，此外还包含另外**两个参数**：
**一个AsyncCallback 委托**
**一个用户定义的状态对象**。

委托用来调用回调方法，状态对象是用来向回调方法传递状态信息。该方法返回一个实现 IAsyncResult 接口的对象。

End 方法用于结束异步操作并返回结果，因此包含同步方法签名中的 ref 和 out 参数，**返回值类型也与同步方法相同**。该方法还包括一个**IAsyncResult** 参数，用于获取异步操作是否完成的信息，当然在使用时就必须传入对应的 Begin 方法返回的对象实例。

开始异步操作后如果要阻止应用程序，可以直接调用 End 方法，这会阻止应用程序直到异步操作完成后再继续执行。也可以使用 `IAsyncResult` 的 `AsyncWaitHandle` 属性，调用其中的`WaitOne`等方法来阻塞线程。
这两种方法的区别不大，只是前者必须一直等待而后者可以设置等待超时。

如果不阻止应用程序，则可以通过轮循 `IAsyncResult` 的 `IsCompleted` 状态来判断操作是否完成，或使用 `AsyncCallback` 委托来结束异步操作。
AsyncCallback 委托包含一个 `IAsyncResult` 的签名，回调方法内部再调用 End 方法来获取操作执行结果。

代码：

```csharp
namespace ConsoleApplication1
{
    class Program
    {
        static AsyncDemo demo = new AsyncDemo();

        static void Main(string[] args)
        {
            //bool state = false;

            //IAsyncResult ar = demo.BeginRun("zhangshan", new AsyncCallback(outPut), state);

            //ar.AsyncWaitHandle.WaitOne(1000, false);

            IAsyncResult ar = demo.BeginRun("zhangshan", null, null);

            if (ar.IsCompleted)
            {
                string demoName = demo.EndRun(ar);
                Console.WriteLine(demoName);
            }
            else
            {
                Console.WriteLine("Sorry,can't get demoName, the time is over");
            }

            System.Threading.Thread.Sleep(1000);

            Console.ReadKey();
        }

        static void outPut(IAsyncResult ar)
        {
            bool state = (bool)ar.AsyncState;
            string demoName = demo.EndRun(ar);

            if (state)
            {
                Console.WriteLine(demoName);
            }
            else
            {
                Console.WriteLine(demoName);
            }
        }
    }

    public class AsyncDemo
    {
        private delegate string runDelegate(string name);

        private runDelegate m_Delegate;

        public AsyncDemo()
        {
            m_Delegate = new runDelegate(Run);
        }

        /// ﹤summary﹥  
        /// Synchronous method  
        /// ﹤/summary﹥  
        /// ﹤returns﹥﹤/returns﹥  
        public string Run(string name)
        {
            return "My name is " + name;
        }

        public IAsyncResult BeginRun(string name, AsyncCallback callBack, Object stateObject)
        {
            try
            {
                return m_Delegate.BeginInvoke(name, callBack, stateObject);
            }
            catch (Exception e)
            {
                throw e;
            }
        }

        public string EndRun(IAsyncResult ar)
        {
            if (ar == null)
                throw new NullReferenceException("Arggument ar can't be null");

            try
            {
                return m_Delegate.EndInvoke(ar);
            }
            catch (Exception e)
            {
                throw e;
            }
        }
    }

}
```

参考：

[C#异步编程模式IAsyncResult浅析](http://developer.51cto.com/art/200908/145362.htm)

[使用 IAsyncResult 进行 .NET 异步编程](https://blog.csdn.net/powertoolsteam/article/details/6284035)

[异步编程(AsyncCallback委托,IAsyncResult接口,BeginInvoke方法,EndInvoke方法的使用小总结)](http://www.cnblogs.com/panjun-Donet/archive/2009/03/03/1284700.html)