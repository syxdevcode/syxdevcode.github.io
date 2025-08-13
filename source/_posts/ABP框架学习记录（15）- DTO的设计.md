---
title: ABP框架学习记录（15）- DTO的设计
date: 2019-08-03 14:28:39
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（15）- DTO的设计

DTO在项目中的目录位置：

![QQ截图20190812131158.png](/img/QQ截图20190812131158.png)

DTO 基础接口，类：

`IEntityDto<TPrimaryKey>`：定义公共的属性接口，此接口为泛型接口；

`IEntityDto`：以Int类型的主键类型接口；

`EntityDto<TPrimaryKey>`：实现 `IEntityDto<TPrimaryKey>`泛型接口；

`EntityDto`：实现 `EntityDto<int>`, `IEntityDto` 接口；

`ComboboxItemDto`：复选框DTO；

此外，还有审计相关DTO：

`CreationAuditedEntityDto<TPrimaryKey>`，`CreationAuditedEntityDto`,

`FullAuditedEntityDto<TPrimaryKey>`，`FullAuditedEntityDto`;

`IPagedResult`：实现 `IListResult<T>`，`IHasTotalCount`,此接口定义输出到客户端的分页标准；

`PagedResultDto<T>`：实现 `ListResultDto<T>`, `IPagedResult<T>`接口；

`IListResult<T>`：定义输出到客户端集合的接口；

`ListResultDto<T>`：实现 `IListResult<T>` 接口；