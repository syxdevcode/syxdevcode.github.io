---
title: CSharp-CodeDOM生成类
date: 2019-01-21 09:24:11
tags:
- DotNet
- CSharp
- CodeDOM
categories: 
- CodeDOM
---
# CSharp-CodeDOM生成类

CodeDOM:代码文档对象模型。

参考：[动态源代码生成和编译](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/dynamic-source-code-generation-and-compilation)

.NET Framework 包括一个名为代码文档对象模型 (CodeDOM) 的机制。通过该机制，发出源代码的程序开发者可基于一种表示要呈现的代码的单一模型，在运行时以多种编程语言生成源代码。
为表示源代码，CodeDOM 元素相互链接，构成一个名为 CodeDOM 图的数据结构。该数据结构对某些源代码的结构建模。

`System.CodeDom` 命名空间定义某些类型，这些类型可以表示源代码的逻辑结构，且不依赖特定编程语言。 `System.CodeDom.Compiler` 命名空间定义某些类型，用于从 CodeDOM 图中生成源代码，并管理使用支持语言的源代码的编译工作。 编译器供应商或开发人员可以扩展支持语言。

.NET Framework 包括适用于 `CSharpCodeProvider`、`JScriptCodeProvider` 和 `VBCodeProvider` 的代码生成器和代码编译器。

## 使用CodeDOM创建类

CodeDOM 提供表示多种常见源代码元素的类型。 可以设计一个程序，它使用 CodeDOM 元素生成源代码模型来组合对象图。 对于支持的编程语言，可使用 CodeDOM 代码生成器将此对象图呈现为源代码。 还可使用 CodeDOM 将源代码编译为二进制程序集。

CodeDOM 的常见用途包括：

* 模板化代码生成：生成适用于 ASP.NET、XML Web 服务客户端代理、代码向导、设计器或其他代码发出机制的代码。
* 动态编译：支持一种或多种语言的代码编译。

参考：[使用 CodeDom](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/using-the-codedom)

`System.CodeDom` 命名空间提供用于表示源代码逻辑结构的类，独立于语言语法。

CodeDOM 图的结构类似于一个容器树。 每个可编译 CodeDOM 图的顶端容器或根部容器是 `CodeCompileUnit`。 源代码模型中的每个元素都必须通过图中 `CodeObject` 的属性链接至该图。

CodeDOM 定义名为 `CodeCompileUnit` 的对象，可引用 CodeDOM 对象图，该对象图塑造要编译的源代码的模型。 `CodeCompileUnit` 具有属性可用于存储对特性、命名空间和程序集的引用。

派生自 `CodeDomProvider` 类的 CodeDom 提供程序包含处理 CodeCompileUnit 所引用的对象图的方法。

详细请参考：[如何：使用 CodeDOM 创建类](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-create-a-class-using-codedom)

[CodeStatement Class](https://docs.microsoft.com/zh-cn/dotnet/api/system.codedom.codestatement?view=netframework-4.7.2)

```cs
namespace ConsoleApp1
{
    using System;
    using System.CodeDom;
    using System.CodeDom.Compiler;
    using System.IO;
    using System.Reflection;

    /// <summary>
    /// This code example creates a graph using a CodeCompileUnit and
    /// generates source code for the graph using the CSharpCodeProvider.
    /// </summary>
    internal class Sample
    {
        /// <summary>
        /// Define the compile unit to use for code generation.
        /// </summary>
        private CodeCompileUnit targetUnit;

        /// <summary>
        /// The only class in the compile unit. This class contains 2 fields,
        /// 3 properties, a constructor, an entry point, and 1 simple method.
        /// </summary>
        private CodeTypeDeclaration targetClass;

        /// <summary>
        /// The name of the file to contain the source code.
        /// </summary>
        private const string outputFileName = "SampleCode.cs";

        /// <summary>
        /// Define the class.
        /// </summary>
        public Sample()
        {
            // 创建编译单元
            targetUnit = new CodeCompileUnit();

            // 定义命名空间
            CodeNamespace samples = new CodeNamespace("CodeDOMSample");

            // 导入命名空间
            samples.Imports.Add(new CodeNamespaceImport("System"));

            // 定义类型
            targetClass = new CodeTypeDeclaration("CodeDOMCreatedClass");
            targetClass.IsClass = true;
            targetClass.TypeAttributes =
                TypeAttributes.Public | TypeAttributes.Sealed;

            // 向命名空间添加类型
            samples.Types.Add(targetClass);

            // 将代码元素链接至对象图
            targetUnit.Namespaces.Add(samples);
        }

        /// <summary>
        /// Adds two fields to the class.
        /// </summary>
        public void AddFields()
        {
            // Declare the widthValue field.
            CodeMemberField widthValueField = new CodeMemberField();
            widthValueField.Attributes = MemberAttributes.Private;
            widthValueField.Name = "widthValue";
            widthValueField.Type = new CodeTypeReference(typeof(System.Double));
            widthValueField.Comments.Add(new CodeCommentStatement(
                "The width of the object."));
            targetClass.Members.Add(widthValueField);

            // Declare the heightValue field
            CodeMemberField heightValueField = new CodeMemberField();
            heightValueField.Attributes = MemberAttributes.Private;
            heightValueField.Name = "heightValue";
            heightValueField.Type =
                new CodeTypeReference(typeof(System.Double));
            heightValueField.Comments.Add(new CodeCommentStatement(
                "The height of the object."));
            targetClass.Members.Add(heightValueField);
        }

        /// <summary>
        /// Add three properties to the class.
        /// </summary>
        public void AddProperties()
        {
            // Declare the read-only Width property.
            CodeMemberProperty widthProperty = new CodeMemberProperty();
            widthProperty.Attributes =
                MemberAttributes.Public | MemberAttributes.Final;
            widthProperty.Name = "Width";
            widthProperty.HasGet = true;
            widthProperty.Type = new CodeTypeReference(typeof(System.Double));
            widthProperty.Comments.Add(new CodeCommentStatement(
                "The Width property for the object."));
            widthProperty.GetStatements.Add(new CodeMethodReturnStatement(
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "widthValue")));
            targetClass.Members.Add(widthProperty);

            // Declare the read-only Height property.
            CodeMemberProperty heightProperty = new CodeMemberProperty();
            heightProperty.Attributes =
                MemberAttributes.Public | MemberAttributes.Final;
            heightProperty.Name = "Height";
            heightProperty.HasGet = true;
            heightProperty.Type = new CodeTypeReference(typeof(System.Double));
            heightProperty.Comments.Add(new CodeCommentStatement(
                "The Height property for the object."));
            heightProperty.GetStatements.Add(new CodeMethodReturnStatement(
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "heightValue")));
            targetClass.Members.Add(heightProperty);

            // Declare the read only Area property.
            CodeMemberProperty areaProperty = new CodeMemberProperty();
            areaProperty.Attributes =
                MemberAttributes.Public | MemberAttributes.Final;
            areaProperty.Name = "Area";
            areaProperty.HasGet = true;
            areaProperty.Type = new CodeTypeReference(typeof(System.Double));
            areaProperty.Comments.Add(new CodeCommentStatement(
                "The Area property for the object."));

            // Create an expression to calculate the area for the get accessor
            // of the Area property.
            CodeBinaryOperatorExpression areaExpression =
                new CodeBinaryOperatorExpression(
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "widthValue"),
                CodeBinaryOperatorType.Multiply,
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "heightValue"));
            areaProperty.GetStatements.Add(
                new CodeMethodReturnStatement(areaExpression));
            targetClass.Members.Add(areaProperty);
        }

        /// <summary>
        /// Adds a method to the class. This method multiplies values stored
        /// in both fields.
        /// </summary>
        public void AddMethod()
        {
            // Declaring a ToString method
            CodeMemberMethod toStringMethod = new CodeMemberMethod();
            toStringMethod.Attributes =
                MemberAttributes.Public | MemberAttributes.Override;
            toStringMethod.Name = "ToString";
            toStringMethod.ReturnType =
                new CodeTypeReference(typeof(System.String));

            CodeFieldReferenceExpression widthReference =
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "Width");
            CodeFieldReferenceExpression heightReference =
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "Height");
            CodeFieldReferenceExpression areaReference =
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "Area");

            // Declaring a return statement for method ToString.
            CodeMethodReturnStatement returnStatement =
                new CodeMethodReturnStatement();

            // This statement returns a string representation of the width,
            // height, and area.
            string formattedOutput = "The object:" + Environment.NewLine +
                " width = {0}," + Environment.NewLine +
                " height = {1}," + Environment.NewLine +
                " area = {2}";
            returnStatement.Expression =
                new CodeMethodInvokeExpression(
                new CodeTypeReferenceExpression("System.String"), "Format",
                new CodePrimitiveExpression(formattedOutput),
                widthReference, heightReference, areaReference);
            toStringMethod.Statements.Add(returnStatement);
            targetClass.Members.Add(toStringMethod);
        }

        /// <summary>
        /// Add a constructor to the class.
        /// </summary>
        public void AddConstructor()
        {
            // Declare the constructor
            CodeConstructor constructor = new CodeConstructor();
            constructor.Attributes =
                MemberAttributes.Public | MemberAttributes.Final;

            // Add parameters.
            constructor.Parameters.Add(new CodeParameterDeclarationExpression(
                typeof(System.Double), "width"));
            constructor.Parameters.Add(new CodeParameterDeclarationExpression(
                typeof(System.Double), "height"));

            // Add field initialization logic
            CodeFieldReferenceExpression widthReference =
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "widthValue");
            constructor.Statements.Add(new CodeAssignStatement(widthReference,
                new CodeArgumentReferenceExpression("width")));
            CodeFieldReferenceExpression heightReference =
                new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "heightValue");
            constructor.Statements.Add(new CodeAssignStatement(heightReference,
                new CodeArgumentReferenceExpression("height")));
            targetClass.Members.Add(constructor);
        }

        /// <summary>
        /// Add an entry point to the class.
        /// </summary>
        public void AddEntryPoint()
        {
            // 为可执行程序定义代码入口点方法
            CodeEntryPointMethod start = new CodeEntryPointMethod();
            CodeObjectCreateExpression objectCreate =
                new CodeObjectCreateExpression(
                new CodeTypeReference("CodeDOMCreatedClass"),
                new CodePrimitiveExpression(5.3),
                new CodePrimitiveExpression(6.9));

            // Add the statement:
            // "CodeDOMCreatedClass testClass =
            //     new CodeDOMCreatedClass(5.3, 6.9);"
            start.Statements.Add(new CodeVariableDeclarationStatement(
                new CodeTypeReference("CodeDOMCreatedClass"), "testClass",
                objectCreate));

            // Creat the expression:
            // "testClass.ToString()"
            CodeMethodInvokeExpression toStringInvoke =
                new CodeMethodInvokeExpression(
                new CodeVariableReferenceExpression("testClass"), "ToString");

            // Add a System.Console.WriteLine statement with the previous
            // expression as a parameter.
            start.Statements.Add(new CodeMethodInvokeExpression(
                new CodeTypeReferenceExpression("System.Console"),
                "WriteLine", toStringInvoke));
            targetClass.Members.Add(start);
        }

        /// <summary>
        /// Generate CSharp source code from the compile unit.
        /// </summary>
        /// <param name="filename">Output file name</param>
        public void GenerateCSharpCode(string fileName)
        {
            CodeDomProvider provider = CodeDomProvider.CreateProvider("CSharp");
            CodeGeneratorOptions options = new CodeGeneratorOptions();
            options.BracingStyle = "C";
            using (StreamWriter sourceWriter = new StreamWriter(fileName))
            {
                provider.GenerateCodeFromCompileUnit(
                    targetUnit, sourceWriter, options);
            }
        }

        /// <summary>
        /// Create the CodeDOM graph and generate the code.
        /// </summary>
        private static void Main()
        {
            Sample sample = new Sample();
            sample.AddFields();
            sample.AddProperties();
            sample.AddMethod();
            sample.AddConstructor();
            sample.AddEntryPoint();
            sample.GenerateCSharpCode(outputFileName);
        }
    }
}
```

生成类的代码：

```cs
//------------------------------------------------------------------------------
// <auto-generated>
//     此代码由工具生成。
//     运行时版本:4.0.30319.42000
//
//     对此文件的更改可能会导致不正确的行为，并且如果
//     重新生成代码，这些更改将会丢失。
// </auto-generated>
//------------------------------------------------------------------------------

namespace CodeDOMSample
{
    using System;


    public sealed class CodeDOMCreatedClass
    {

        // The width of the object.
        private double widthValue;

        // The height of the object.
        private double heightValue;

        public CodeDOMCreatedClass(double width, double height)
        {
            this.widthValue = width;
            this.heightValue = height;
        }

        // The Width property for the object.
        public double Width
        {
            get
            {
                return this.widthValue;
            }
        }

        // The Height property for the object.
        public double Height
        {
            get
            {
                return this.heightValue;
            }
        }

        // The Area property for the object.
        public double Area
        {
            get
            {
                return (this.widthValue * this.heightValue);
            }
        }

        public override string ToString()
        {
            return string.Format("The object:\r\n width = {0},\r\n height = {1},\r\n area = {2}", this.Width, this.Height, this.Area);
        }

        public static void Main()
        {
            CodeDOMCreatedClass testClass = new CodeDOMCreatedClass(5.3D, 6.9D);
            System.Console.WriteLine(testClass.ToString());
        }
    }
}
```































































































