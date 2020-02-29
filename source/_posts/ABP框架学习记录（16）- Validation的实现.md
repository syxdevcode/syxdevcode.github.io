---
title: ABP框架学习记录（16）- Validation的实现
date: 2019-08-12 13:35:19
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（16）- Validation的实现

`Validation` 在ABP项目中的位置：

![QQ截图20190812145325.png](/img/QQ截图20190812145325.png)

## 定义

`ValidatorAttribute`：自定义属性，继承 `Attribute`；

`IValueValidator`：定义值验证接口；

`ValueValidatorBase`：默认实现 `IValueValidator` 接口，提供抽象基类；

![QQ截图20190820165721.png](/img/QQ截图20190820165721.png)

`StringValueValidator`：验证字符串类型；

`BooleanValueValidator`：验证布尔类型；

`NumericValueValidator`：验证数字类型；

`DisableValidationAttribute`：禁止自动验证属性，适用于方法，类，属性；

`EnableValidationAttribute`：启用验证，可以在禁止验证的类中的方法添加，以启动自动验证；

`ICustomValidate`：提供自定义验证接口，自定义验证类必须实现此接口；

![QQ截图20190820172950.png](/img/QQ截图20190820172950.png)

`CustomValidationContext`：自定义验证上下文；

![QQ截图20190820181037.png](/img/QQ截图20190820181037.png)

`IMethodParameterValidator`：定义验证方法参数的接口；

![QQ截图20190821135444.png](/img/QQ截图20190821135444.png)

`CustomValidator`：实现 `IMethodParameterValidator` 接口；自定义验证

![QQ截图20190821140040.png](/img/QQ截图20190821140040.png)

`ValidatableObjectValidator`：对象验证器。确定指定的对象是否有效。

![QQ截图20190821141036.png](/img/QQ截图20190821141036.png)

`DataAnnotationsValidator`：数据注释验证器

![QQ截图20190821142048.png](/img/QQ截图20190821142048.png)

`IShouldNormalize`：此接口用于在方法执行之前规范化输入。

![QQ截图20190821144206.png](/img/QQ截图20190821144206.png)

![QQ截图20190821144655.png](/img/QQ截图20190821144655.png)

## 初始化

`AbpBootstrapper` 的 `AddInterceptorRegistrars` 方法初始化验证拦截器：

![QQ截图20190821142450.png](/img/QQ截图20190821142450.png)

`ValidationInterceptorRegistrar`:验证拦截器注册。

![QQ截图20190821142924.png](/img/QQ截图20190821142924.png)

`ValidationInterceptor`：验证拦截器。

![QQ截图20190821143004.png](/img/QQ截图20190821143004.png)

`MethodInvocationValidator`：验证方法参数。

![QQ截图20190821143536.png](/img/QQ截图20190821143536.png)

`AbpKernelModule` 的预初始化方法 `PreInitialize`，调用 `AddMethodParameterValidators`

![QQ截图20190821142706.png](/img/QQ截图20190821142706.png)