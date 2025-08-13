---
title: ABP框架学习记录（13）- Entity解析
date: 2019-07-22 11:19:17
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（13）- Entity解析

`Entity` 在ABP项目中的目录位置：

![QQ截图20190722112057.png](/img/QQ截图20190722112057.png)

## Entity

`IEntity<TPrimaryKey>`：封装了PrimaryKey：Id,这是一个泛型类型，所有的实体都必须实现此接口；

`IEntity`: 继承 `IEntity<int>` 泛型接口；

`Entity<TPrimaryKey>`：继承 `IEntity<TPrimaryKey>` 接口的基础实现，支持主键是泛型的实体，该类重写了`Equals`,`GetHashCode`,`ToString` 方法和`==`,`!=` 运算符。

`Entity`：主键是int类型的Entity;

`IEntityTranslation`:实体语言转化；

`IEntityTranslation<TEntity, TPrimaryKeyOfMultiLingualEntity>`: 实体语言转换泛型类，`TPrimaryKeyOfMultiLingualEntity` 为主键类型；

`IEntityTranslation<TEntity>`：实体语言转换泛型类，继承 `IEntityTranslation<TEntity, int>` 接口；

`IMultiLingualEntity<TTranslation> `：多语言实体，其中 `TTranslation` 类型约束为 `IEntityTranslation`；

`IEntityTranslation<>` 接口的实现，表明当前实体可以被翻译：

![QQ截图20190722150243.png](/img/QQ截图20190722150243.png)

`IMultiLingualEntity<>` 接口的实现，表示有多语言的实现：

![QQ截图20190722150428.png](/img/QQ截图20190722150428.png)

使用：

![QQ截图20190722150703.png](/img/QQ截图20190722150703.png)

![QQ截图20190722150813.png](/img/QQ截图20190722150813.png)

`IExtendableObject`：使用 Json 格式的字符串属性，以扩展对象/实体。

`ExtendableObjectExtensions`：针对扩展字段属性的扩展类；

`IAggregateRoot`：聚合根接口

`AggregateRoot`：聚合根实现

![QQ截图20190722153654.png](/img/QQ截图20190722153654.png)

![QQ截图20190722153720.png](/img/QQ截图20190722153720.png)

`IMayHaveTenant`：为可能选择具有TenantId的实体实现此接口。

`IMustHaveTenant`：为必须具有TenantId的实体实现此接口。

`IPassivable`：此接口用于使实体主动/被动。

`ISoftDelete`：用于标准化软删除实体。

## EntityCache

`IEntityCache<TCacheItem, TPrimaryKey>`：定义实体缓存接口，此接口为泛型类型；

`IEntityCache<TCacheItem>`：继承 `IEntityCache<TCacheItem, int>` 接口；

`EntityCache<TEntity, TCacheItem, TPrimaryKey>`：实体缓存类；

![QQ截图20190722171203.png](/img/QQ截图20190722171203.png)

## Auditing

`IAudited`：定义实体的审计接口，保存/更新时自动设置相关属性

`IAudited<TUser>`：审计接口。

`ICreationAudited`： 审计创建者接口，保存到数据库时，会自动设置创建时间和创建者用户。

`IDeletionAudited`: 删除时审计信息

`IFullAudited`：完全审核信息。

`IHasCreationTime`：如果必须存储实体，则可以实现此接口，自动设置；

`IHasDeletionTime`：如果必须删除实体，则可以实现此接口，自动设置；

`IHasModificationTime`：更新实体时间；

`IModificationAudited`：实体更新人信息；

`AuditedEntity`：审计实体；

`CreationAuditedEntity`：创建者审计实体；

`FullAuditedEntity`：全部审计信息实体；

`AuditedAggregateRoot`：审计聚合根

`CreationAuditedAggregateRoot`：创建者审计 聚合根；

`FullAuditedAggregateRoot`：全部审计信息 聚合根；

`EntityAuditingHelper`：实体审计帮助类；

![QQ截图20190722174813.png](/img/QQ截图20190722174813.png)

参考：

[ABP源码分析十四：Entity的设计](https://www.cnblogs.com/1zhk/p/5329393.html)