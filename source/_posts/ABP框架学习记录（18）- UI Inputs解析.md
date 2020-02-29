---
title: ABP框架学习记录（18）- UI Inputs解析
date: 2019-08-23 10:34:46
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（18）- UI Inputs解析

UI Inputs 在 ABP 项目中的路径：

![QQ截图20190823104812.png](/img/QQ截图20190823104812.png)

`IInputType` ：定义输入类型接口；

![QQ截图20190823105038.png](/img/QQ截图20190823105038.png)

`InputTypeBase`：默认实现 `IInputType` 接口；

![QQ截图20190823105319.png](/img/QQ截图20190823105319.png)

`InputTypeAttribute`：定义输入类型属性；

`SingleLineStringInputType`：单行字符串；

```cs
public override void SetFeatures(IFeatureDefinitionContext context)
{
    var contacts = context.Create(Names.Contacts, "false");
    contacts.CreateChildFeature(Names.MaxContactCount, "100", inputType: new SingleLineStringInputType(new NumericValueValidator(1, 10000)));

    contacts.CreateChildFeature(Names.ChildFeatureToOverride, "ChildFeature");
    contacts.RemoveChildFeature(Names.ChildFeatureToOverride);
    contacts.CreateChildFeature(Names.ChildFeatureToOverride, "ChildFeatureToOverride");

    contacts.CreateChildFeature(Names.ChildFeatureToDelete, "ChildFeatureToDelete");
    contacts.RemoveChildFeature(Names.ChildFeatureToDelete);
}
```

`CheckboxInputType`：复选框类型；

`ComboboxInputType`：下拉列表；

`InputType` 验证：`Feature` 对象的 `InputType` 对象，`InputType` 的 `Validator` 对象的 `IsValid`方法；

![QQ截图20190823104156.png](/img/QQ截图20190823104156.png)