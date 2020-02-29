---
title: CSharp equal() 和 == 的区别
date: 2017-05-02 08:35:10
tags: 
- CSharp
- DotNet
- CSharp基础
categories: 
- CSharp基础
---
# CSharp中equal与==的区别

C#中，判断相等有两种方式，一种是传统的==操作，一种是object提供的Equals方法。

二者的区别在于：

* 一、==操作符判断的是堆栈中的值，Equlas判断的是堆中的值。

    C#提供值类型和引用类型，值类型存储在栈上，故用==判断是直接判断其值是否相等，因为值类型不存在堆中的数据，因此值类型的Equals也是判断数据。
    即，对于值类型而言，==与Equals相同，均是判断其值是否相等。

    对于引用类型而言，其栈中存储的是对象的地址，那么==就是比较两个地址是否相等，即是否指向同一个对象；Equals函数则是比较两个对象在堆中的数据
    是否一样，即两个引用类型是否是对同一个对象的引用。

* 二、String类型特殊

    String类型虽然是引用类型，但是对String对象的赋值却按照值类型操作。

    例如：
    ````csharp
    String s1="hello";
    String s2="hello";
    ````
    对s2初始化的时候，并没有重新开辟内存，而是直接将其地址指向s1的内容“hello”。这样一来，string类型虽然是引用类型，但是其==操作和Equals操作
    都是一样的，均比较值是否相等。

* 三、与GetHashCode()的关系

    若两对象Equals相等，那么其GetHashCode()必定相等；但是反过来，若GetHashCode()相等，那么这两个对象Equals方法比较结果不一定相同。

   （为了获取最佳性能，hash函数为对象内容生成的数字是随机分布的，这就意味着，内容不同的对象，有可能生成的数字是一样的，但可以认为这种概率非常小）。

    下面示例说明：

    ````csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;

    namespace ConsoleApplication1
    {
        class People
        {
            private string name;

            public string Name
            {
                get { return name; }
                set { name = value; }
            }

            public People(string name)
            {
                this.name = name;
            }
        }

        class Program
        {
            static void Main(string[] args)
            {
                string a = "hello";
                string b = new string(new char[] { 'h', 'e', 'l', 'l', 'o' });
                Console.WriteLine(a == b);
                Console.WriteLine(a.Equals(b));
                Console.WriteLine("\n");

                Int32 m = 3;
                Int32 n = 3;
                Console.WriteLine(n == m);
                Console.WriteLine(n.Equals(m));
                Console.WriteLine("\n");

                object g = a;
                object h = b;
                Console.WriteLine(g == h);
                Console.WriteLine(g.Equals(h));
                Console.WriteLine(g.GetHashCode() + "   " + h.GetHashCode());
                Console.WriteLine("\n");

                People p1 = new People("Jimmy");
                People p2 = new People("Jimmy");
                Console.WriteLine(p1 == p2);
                Console.WriteLine(p1.Equals(p2));
                Console.WriteLine(p1.GetHashCode() + "  " + p2.GetHashCode());
                Console.WriteLine("\n");

                People p3 = new People("Jimmy");
                People p4 = p3;
                Console.WriteLine(p3 == p4);
                Console.WriteLine(p3.Equals(p4));

            }
        }
    }
    ````

    运行结果：
    ![运行结果](/img/equals.png)