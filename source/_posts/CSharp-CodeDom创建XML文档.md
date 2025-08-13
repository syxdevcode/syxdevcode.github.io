---
title: CSharp-CodeDom创建XML文档
date: 2019-01-22 09:44:26
tags:
- DotNet
- CSharp
- CodeDOM
categories: 
- CodeDOM
---
# CSharp-CodeDom创建XML文档

可使用 CodeDOM 创建生成 XML 文档的代码。 该进程包括创建包含 XML 文档注释的 CodeDOM 图、生成代码和通过创建 XML 文档输出的编译器选项编译生成的代码。

参考：[如何：使用 CodeDOM 创建 XML 文档文件](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-create-an-xml-documentation-file-using-codedom)

```cs
namespace ConsoleApp1
{
    using System;
    using System.CodeDom;
    using System.CodeDom.Compiler;
    using System.IO;
    using System.Text.RegularExpressions;

    class Program
    {
        static string providerName = "cs";
        static string sourceFileName = "test.cs";
        static void Main(string[] args)
        {
            CodeDomProvider provider =
                CodeDomProvider.CreateProvider(providerName);

            LogMessage("Building CodeDOM graph...");

            CodeCompileUnit cu = new CodeCompileUnit();

            cu = BuildHelloWorldGraph();

            StreamWriter sourceFile = new StreamWriter(sourceFileName);
            provider.GenerateCodeFromCompileUnit(cu, sourceFile, null);
            sourceFile.Close();

            CompilerParameters opt = new CompilerParameters(new string[]{
                                      "System.dll" });
            opt.GenerateExecutable = true;
            opt.OutputAssembly = "HelloWorld.exe";
            opt.TreatWarningsAsErrors = true;
            opt.IncludeDebugInformation = true;
            opt.GenerateInMemory = true;
            opt.CompilerOptions = "/doc:HelloWorldDoc.xml";

            CompilerResults results;

            LogMessage("Compiling with " + providerName);
            results = provider.CompileAssemblyFromFile(opt, sourceFileName);

            OutputResults(results);
            if (results.NativeCompilerReturnValue != 0)
            {
                LogMessage("");
                LogMessage("Compilation failed.");
            }
            else
            {
                LogMessage("");
                LogMessage("Demo completed successfully.");
            }
            //File.Delete(sourceFileName);

            Console.ReadKey();
        }

        // Build a Hello World program graph using 
        // System.CodeDom types.
        public static CodeCompileUnit BuildHelloWorldGraph()
        {
            // Create a new CodeCompileUnit to contain 
            // the program graph.
            CodeCompileUnit compileUnit = new CodeCompileUnit();

            // Declare a new namespace called Samples.
            CodeNamespace samples = new CodeNamespace("Samples");
            // Add the new namespace to the compile unit.
            compileUnit.Namespaces.Add(samples);

            // Add the new namespace import for the System namespace.
            samples.Imports.Add(new CodeNamespaceImport("System"));

            // Declare a new type called Class1.
            CodeTypeDeclaration class1 = new CodeTypeDeclaration("Class1");

            class1.Comments.Add(new CodeCommentStatement("<summary>", true));
            class1.Comments.Add(new CodeCommentStatement(
                "Create a Hello World application.", true));
            class1.Comments.Add(new CodeCommentStatement(
                @"<seealso cref=" + '"' + "Class1.Main" + '"' + "/>", true));
            class1.Comments.Add(new CodeCommentStatement("</summary>", true));

            // Add the new type to the namespace type collection.
            samples.Types.Add(class1);

            // Declare a new code entry point method.
            CodeEntryPointMethod start = new CodeEntryPointMethod();
            start.Comments.Add(new CodeCommentStatement("<summary>", true));
            start.Comments.Add(new CodeCommentStatement(
                "Main method for HelloWorld application.", true));
            start.Comments.Add(new CodeCommentStatement(
                @"<para>Add a new paragraph to the description.</para>", true));
            start.Comments.Add(new CodeCommentStatement("</summary>", true));

            // Create a type reference for the System.Console class.
            CodeTypeReferenceExpression csSystemConsoleType =
                new CodeTypeReferenceExpression("System.Console");

            // Build a Console.WriteLine statement.
            CodeMethodInvokeExpression cs1 = new CodeMethodInvokeExpression(
                csSystemConsoleType, "WriteLine",
                new CodePrimitiveExpression("Hello World!"));

            // Add the WriteLine call to the statement collection.
            start.Statements.Add(cs1);

            // Build another Console.WriteLine statement.
            CodeMethodInvokeExpression cs2 = new CodeMethodInvokeExpression(
                csSystemConsoleType, "WriteLine", new CodePrimitiveExpression(
                "Press the ENTER key to continue."));

            // Add the WriteLine call to the statement collection.
            start.Statements.Add(cs2);

            // Build a call to System.Console.ReadLine.
            CodeMethodInvokeExpression csReadLine =
                new CodeMethodInvokeExpression(csSystemConsoleType, "ReadLine");

            // Add the ReadLine statement.
            start.Statements.Add(csReadLine);

            // Add the code entry point method to
            // the Members collection of the type.
            class1.Members.Add(start);

            return compileUnit;
        }
        static void LogMessage(string text)
        {
            Console.WriteLine(text);
        }

        static void OutputResults(CompilerResults results)
        {
            LogMessage("NativeCompilerReturnValue=" +
                results.NativeCompilerReturnValue.ToString());
            foreach (string s in results.Output)
            {
                LogMessage(s);
            }
        }
    }
}
```
