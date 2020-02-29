---
title: ABP框架学习记录（17）- Feature解析
date: 2019-08-22 09:50:14
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（17）- Feature解析

Feature：特征，功能。

Feature是什么？Feature就是对function的分类方法，其与function的关系就比如Role和User的关系一样。

ABP中Feature具有以下属性： 其中最重要的属性是name，用以表示feature的Identity,一个feature一个name. 一个Feature可以有一组子Features，从而构成Feature树。

`Feature` 设计的整体流程：

`FeatureChecker` 作用：获取 `Feaure` 的值，`FeatureChecker` 类中定义的 `FeatureValueStore` 负责获取存储的值，`IFeatureManager` 负责管理 `Feaure`。

Feature目录位置：

![QQ截图20190822100703.png](/img/QQ截图20190822100703.png)

`Feature`：定义应用程序的功能。可以在多租户应用程序中使用，根据版本和租户启用或禁用某些应用程序功能。

`Feature` 中验证：

![QQ截图20190823104156.png](/img/QQ截图20190823104156.png)

`FeatureScopes`：`Feature` 的范围；

![QQ截图20190822104856.png](/img/QQ截图20190822104856.png)

`FeatureDictionary`：`Feature` 字典;

![QQ截图20190822105858.png](/img/QQ截图20190822105858.png)

`IFeatureDefinitionContext/FeatureDefinitionContextBase`：在 `FeatureProvider.SetFeatures` 方法中用作上下文；

`FeatureProvider`：`Feature` 提供者；

![QQ截图20190822154145.png](/img/QQ截图20190822154145.png)

`IFeatureConfiguration/FeatureConfiguration`：`Feature` 配置；

![QQ截图20190822154908.png](/img/QQ截图20190822154908.png)

`IFeatureManager/FeatureManager`：`Feature` 管理器;

![QQ截图20190822155121.png](/img/QQ截图20190822155121.png)

提供 `Initialize` 初始化 `Feature`:

![QQ截图20190822161707.png](/img/QQ截图20190822161707.png)

`IFeatureValueStore`：定义获取 `Feature` 值的存储接口；

![QQ截图20190822164845.png](/img/QQ截图20190822164845.png)

`NullFeatureValueStore`：`IFeatureValueStore` 接口的默认实现；

`IFeatureChecker`：获取 `Feature` 的值。

`FeatureChecker`：默认实现 `IFeatureChecker` 接口，提供获取 `Feature` 值的接口。

![QQ截图20190822165438.png](/img/QQ截图20190822165438.png)

`IFeatureDependency`：定义功能依赖项接口；

![QQ截图20190822165603.png](/img/QQ截图20190822165603.png)

`IFeatureDependencyContext/FeatureDependencyContext`：定义功能依赖项上下文；

![QQ截图20190822165744.png](/img/QQ截图20190822165744.png)

`SimpleFeatureDependency`：`IFeatureDependency` 接口的简单实现；

![QQ截图20190822173642.png](/img/QQ截图20190822173642.png)

`RequiresFeatureAttribute`: `Feature` 启用的情况下，以声明给定的类/方法可用。

![QQ截图20190822173830.png](/img/QQ截图20190822173830.png)

参考：

[ABP源码分析二十一：Feature](https://www.cnblogs.com/1zhk/p/5351968.html)