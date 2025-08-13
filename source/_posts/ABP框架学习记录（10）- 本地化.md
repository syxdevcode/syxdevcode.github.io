---
title: ABP框架学习记录（10）- 本地化
date: 2019-06-11 17:26:12
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（10）- 本地化

ABP中的本地化主要涉及两个方面：一个是语言（Language）的管理（相对简单），另一个是语言对应得本地化资源（Localization）的管理（稍显复杂）。

目录结构：

![QQ截图20190618095421.png](/img/QQ截图20190618095421.png)

## 引用

在 `AbpKernelModule` 完成对本地化资源的引用，设置的初始化： 

`PreInitialize` 方法调用 `AddLocalizationSources` 方法，添加资源：

![QQ截图20190618131540.png](/img/QQ截图20190618131540.png)

`PreInitialize` 方法调用 `AddSettingProviders` 方法，添加设置：

![QQ截图20190618111856.png](/img/QQ截图20190618111856.png)

`PostInitialize` 方法初始化：

![QQ截图20190618111740.png](/img/QQ截图20190618111740.png)

## LocalizationManager/ILocalizationManager

遍历 `LocalizationConfiguration` 中的 `ILocalizationSourceList` 实例，通过 `ILocalizationSource` 的`ILocalizationDictionaryProvider` 实例完成本地化资源的初始化。

![QQ截图20190618135216.png](/img/QQ截图20190618135216.png)

`ILocalizationSource` 接口：

![QQ截图20190618140712.png](/img/QQ截图20190618140712.png)

`ILocalizationConfiguration/LocalizationConfiguration`: 用于配置支持本地化的语言的一个 `LanguageInfo` 集合，以及这些语言所对应的本地化资源。这两者分别对应 `ILocalizationConfiguration` 中的 `Lanugages` 和 `Sources` 属性。注意这个Sources是一个 `ILocalizationSourceList` 实例。

## DictionaryBasedLocalizationSource

`IDictionaryBasedLocalizationSource` : 继承 `ILocalizationSource` 接口，`DictionaryBasedLocalizationSource` 实现 `IDictionaryBasedLocalizationSource` 接口，用于构建本地化源，适用于基于内存的词典以查找字符串。

![QQ截图20190618140440.png](/img/QQ截图20190618140440.png)

子类的实现：

![QQ截图20190618142054.png](/img/QQ截图20190618142054.png)

应用：

![QQ截图20190618142251.png](/img/QQ截图20190618142251.png)

`LocalizationSourceExtensionInfo`：用于扩展本地化资源。ABP在 `LocalizationManager` 初始化的过程中将`LocalizationSourceExtensionInfo` 所对应的本地化资源扩充到 `ILocalizationSource` 对象的相应本地化资源字典中。

## LocalizationDictionaryProviderBase

`LocalizationDictionaryProviderBase` 实现 `ILocalizationDictionaryProvider` 接口，它封装了一个`IDictionary<string, ILocalizationDictionary>` 实例(这是ABP在runtime时候，唯一持有本地化资源的对象)，其中key就是sourceName。并且提供了一个方法Initialize来初始化本地化这个Dictionary。可以通过实现这个接口来提供其他类型的本地化资源。比如 Abp.Zero 就实现了数据库的本地化资源。

![QQ截图20190618143125.png](/img/QQ截图20190618143125.png)

`Initialize` 方法：

![QQ截图20190618143206.png](/img/QQ截图20190618143206.png)

`LocalizationDictionaryProviderBase`：实现了 `ILocalizationDictionaryProvider` 的抽象类，实现了extend本地化Dictionary的方法，这个方法主要用于初始化完成以后，用于扩展相应的ILocalizationDictionary对象。

`XmlFileLocalizationDictionaryProvider`，`JsonFileLocalizationDictionaryProvider`，`JsonEmbeddedFileLocalizationDictionaryProvider`，`XmlEmbeddedFileLocalizationDictionaryProvider` 类，继承 `LocalizationDictionaryProviderBase`;

## LocalizationDictionary

`ILocalizationDictionary`：提供了索引器this[]方法的接口，该方法接受一个string返回的是本地化的string。当`LocalizationManager` 初始化动作结束后，每一种本地化语言的都对应有且仅有的一个 `ILocalizationDictionary` 对象，这个对象用于保存该语言的所有本地化信息。

`LocalizationDictionary`：实现了 `ILocalizationDictionary` 和 `IEnumerable` 两个接口，他本身就是一个具有集合操作的类。其内部封装了一个Dictionary的实例，用于提供真正的集合操作。这个基类只提供了从其内部的Dictionary中根据原string查找返回本地化的string。 其本身并没有将本地化资源文件中的数据加载到其内部的Dictionary的功能，这部分是在其子类中实现的。

![QQ截图20190618143919.png](/img/QQ截图20190618143919.png)

`XmlLocalizationDictionary`：实现 `BuildFomFile` 和 `BuildFomXmlString` 方法用于从XML文件读取本地化数据。

`JsonLocalizationDictionary`：实现 `BuildFromFile` 和 `BuildFromJsonString` 方法用于从Json文件读取本地化数据。

`JsonLocalizationFile`: 反序列化Json字符串到 `JsonLocalizationFile` 对象。

## LanguageManager

`ILanguageManager/LanguageManager`：通过调用 `ILanguageProvider` 接口返回 `LanguageInfo` 的一个集合。以及返回服务器的当前语言设置，如果服务器的当前语言不在 `LocalizationConfiguration` 的本地化语言集合中，则返回`LocalizationConfiguration` 的本地化语言集合中的第一项。

`ILanguageProvider`：接口定义一个返回本地化语言集合的方法。这里使用接口做隔离是有必要的，因为ABP底层框架的`DefaultLanguageProvider` 只是返回通过代码 `hardcode` (硬编码) 到系统中的 `LanguageInfo` 信息。如果需要从其他source(比如数据库)中获取配置的 `LanguageInfo` 信息，那么我们就必须实现自定义的 `LanguageProvider`。

`DefaultLanguageProvider`：从 `LocalizationConfiguration` 读取 `LanguageInfo` 的集合。

`LanguageInfo`：用于封装language的基本信息。

参考：

[ABP源码分析十二：本地化](https://www.cnblogs.com/1zhk/p/5320751.html)