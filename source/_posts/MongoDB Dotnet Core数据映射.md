---
title: MongoDB Dotnet Core数据映射
date: 2020-09-28 13:56:36
tags:
- DotNetCore
- CSharp
- MongoDB
categories:
- MongoDB
---

## 简介

作为一个数据库，基本的操作就是 `CRUD`。`MongoDB` 的 `CRUD`，不使用 `SQL` 来写，而是提供了更简单的方式。

**BsonDocument方式**

`BsonDocument` 方式，适合能熟练使用 `MongoDB Shell` 的开发者。`MongoDB Driver` 提供了完全覆盖 `Shell` 命令的各种方式，来处理用户的 `CRUD` 操作。

这种方法自由度很高，可以在不需要知道完整数据集结构的情况下，完成数据库的CRUD操作。

**数据映射方式**

数据映射是最常用的一种方式。准备好需要处理的数据类，直接把数据类映射到 `MongoDB`，并对数据集进行 `CRUD` 操作。

## 字段映射

### ID字段

`MongoDB` 数据集中存放的数据，称之为文档（`Document`）。每个文档在存放时，都需要有一个ID，而这个 `ID` 的名称，固定叫 `_id`，类型是 `MongoDB.Bson.ObjectId`。

当建立映射时，如果给出 `_id` 字段，则 `MongoDB` 会采用这个 `ID` 做为这个文档的 `ID` ，如果不给出，`MongoDB` 会自动添加一个 `_id` 字段。在使用上是完全一样的。唯一的区别是，如果映射类中不写 `_id`，则 `MongoDB` 自动添加 `_id` 时，会用 `ObjectId` 作为这个字段的数据类型。

`ObjectId` 是一个全局唯一的数据。

`MongoDB` 允许使用其它类型的数据作为 `ID`，例如：`string`，`int`，`long`，`GUID` 等，但这就需要你自己去保证这些数据不超限并且唯一。

可以在类中修改 `_id` 名称为别的名称，但需要加一个描述属性 `BsonId`，`BsonId` 属性会告诉映射，`topic_id` 就是这个文档数据的`ID` 。`MongoDB`在保存时，会将这个 `topic_id` 转成 `_id` 保存到数据集中。

```c#
public class CollectionModel
{
    [BsonId]
    public ObjectId topic_id { get; set; }
    public string title { get; set; }
    public string content { get; set; }
}
```

注：在 `MongoDB` 数据集中，`ID` 字段的名称固定叫 `_id`。为了代码的阅读方便，可以在类中改为别的名称，但这不会影响 `MongoDB` 中存放的 `ID` 名称。

## 2. 特殊类型 - Decimal

`MongoDB` 在早期，是不支持 `Decimal` 的。直到 `MongoDB v3.4` 开始，数据库才正式支持 `Decimal`。

所以，如果使用的是 `v3.4` 以后的版本，可以直接使用，而如果是以前的版本，需要用以下的方式：

```c#
[BsonRepresentation(BsonType.Double, AllowTruncation = true)]
public decimal price { get; set; }
```

其实就是把 `Decimal` 通过映射，转为 `Double` 存储。

## 类字段

添加两个类 `Contact` 和 `Author`：

```c#
public class Contact
{
    public string mobile { get; set; }
}
public class Author
{
    public string name { get; set; }
    public List<Contact> contacts { get; set; }
}

public class CollectionModel
{
    [BsonId]
    public ObjectId topic_id { get; set; }
    public string title { get; set; }
    public string content { get; set; }
    public int favor { get; set; }
    public Author author { get; set; }
}

/*
调用 代码:
*/
private static async Task Demo()
{
    CollectionModel new_item = new CollectionModel()
    {
        title = "Demo",
        content = "Demo content",
        favor = 100,
        author = new Author
        {
            name = "WangPlus",
            contacts = new List<Contact>(),
        }
    };

    Contact contact_item1 = new Contact()
    {
        mobile = "13800000000",
    };
    Contact contact_item2 = new Contact()
    {
        mobile = "13811111111",
    };
    new_item.author.contacts.Add(contact_item1);
    new_item.author.contacts.Add(contact_item2);

    await _collection.InsertOneAsync(new_item);
}
```

文档结构：

```C#
{ 
    "_id" : ObjectId("5ef1e635ce129908a22dfb5e"), 
    "title" : "Demo", 
    "content" : "Demo content", 
    "favor" : NumberInt(100),
    "author" : {
        "name" : "WangPlus", 
        "contacts" : [
            {
                "mobile" : "13800000000"
            }, 
            {
                "mobile" : "13811111111"
            }
        ]
    }
}
```

## 枚举字段

创建一个枚举 `TagEnumeration`:

```c#
public enum TagEnumeration
{
    CSharp = 1,
    Python = 2,
}
```

加到 `CollectionModel` 中:

```c#
public class CollectionModel
{
    [BsonId]
    public ObjectId topic_id { get; set; }
    public string title { get; set; }
    public string content { get; set; }
    public int favor { get; set; }
    public Author author { get; set; }
    public TagEnumeration tag { get; set; }
}
```

Demo代码：

```c#
private static async Task Demo()
{
    CollectionModel new_item = new CollectionModel()
    {
        title = "Demo",
        content = "Demo content",
        favor = 100,
        author = new Author
        {
            name = "WangPlus",
            contacts = new List<Contact>(),
        },
        tag = TagEnumeration.CSharp,
    };
    
    Contact contact_item1 = new Contact()
    {
        mobile = "13800000000",
    };
    Contact contact_item2 = new Contact()
    {
        mobile = "13811111111",
    };
    new_item.author.contacts.Add(contact_item1);
    new_item.author.contacts.Add(contact_item2);

    await _collection.InsertOneAsync(new_item);
}
```

保存后的文档：注意，`tag` 保存了枚举的值。

```c#
{ 
    "_id" : ObjectId("5ef1eb87cbb6b109031fcc31"), 
    "title" : "Demo", 
    "content" : "Demo content", 
    "favor" : NumberInt(100), 
    "author" : {
        "name" : "WangPlus", 
        "contacts" : [
            {
                "mobile" : "13800000000"
            }, 
            {
                "mobile" : "13811111111"
            }
        ]
    }, 
    "tag" : NumberInt(1)
}
```

可以保存枚举的字符串。只要在 `CollectionModel` 中，`tag` 声明上加个属性：

```c#
[BsonRepresentation(BsonType.String)]
public TagEnumeration tag { get; set; }
```

数据会变成：

```c#
{ 
    "_id" : ObjectId("5ef1ec448f1d540919d15904"), 
    "title" : "Demo", 
    "content" : "Demo content", 
    "favor" : NumberInt(100), 
    "author" : {
        "name" : "WangPlus", 
        "contacts" : [
            {
                "mobile" : "13800000000"
            }, 
            {
                "mobile" : "13811111111"
            }
        ]
    }, 
    "tag" : "CSharp"
}
```

## 日期字段

`CollectionModel` 中增加一个时间字段：

```c#
public DateTime post_time { get; set; }
```

`MongoDB` 的 `datetime` 存储的是 `unixtimestamp` ，所以默认只能是 `utc0` 时区的，它问题出在 `C#` 的 `DateTime` 对时区的处理上遗留的问题，可以换成 `DateTimeOffset`。

如果只是保存（像上边这样），或者查询时使用时间作为条件（例如查询 `post_time < DateTime.Now` 的数据）时，是可以使用的，不会出现问题。

但是，如果是查询结果中有时间字段，那这个字段，会被 `DateTime` 默认设置为 `DateTimeKind.Unspecified` 类型。而这个类型，是无时区信息的，输出显示时，会造成混乱。

为了避免这种情况，在进行时间字段的映射时，需要加上属性：

```c#
[BsonDateTimeOptions(Kind = DateTimeKind.Local)]
public DateTime post_time { get; set; }
```

这样做，会强制 `DateTime` 类型的字段为 `DateTimeKind.Local` 类型。这时候，从显示到使用就正确了。

数据集中存放的是 `UTC` 时间，跟我们正常的时间有8小时时差，如果我们需要按日统计，比方每天的销售额/点击量。

**按年月日时分秒拆开存放**

```c#
class Post_Time
{
    public int year { get; set; }
    public int month { get; set; }
    public int day { get; set; }
    public int hour { get; set; }
    public int minute { get; set; }
    public int second { get; set; }
}
```

**MyDateTimeSerializer**

```c#
public class MyDateTimeSerializer : DateTimeSerializer
{
    public override DateTime Deserialize(BsonDeserializationContext context, BsonDeserializationArgs args)
    {
        var obj = base.Deserialize(context, args);
        return new DateTime(obj.Ticks, DateTimeKind.Unspecified);
    }
    public override void Serialize(BsonSerializationContext context, BsonSerializationArgs args, DateTime value)
    {
        var utcValue = new DateTime(value.Ticks, DateTimeKind.Utc);
        base.Serialize(context, args, utcValue);
    }
}
```

注意：使用这个方法，一定不要添加时间的属性 `[BsonDateTimeOptions(Kind = DateTimeKind.Local)]`

对某个特定映射的特定字段使用，比方只对 `CollectionModel` 的 `post_time` 字段来使用，可以这么写：

```c#
[BsonSerializer(typeof(MyDateTimeSerializer))]
public DateTime post_time { get; set; }

//或者全局使用：

BsonSerializer.RegisterSerializer(typeof(DateTime), new MongoDBDateTimeSerializer());

// BsonSerializer是 MongoDB.Driver 的全局对象,可以放到使用数据库前的任何地方。例如在Demo中，放在Main里：
static async Task Main(string[] args)
{
    BsonSerializer.RegisterSerializer(typeof(DateTime), new MyDateTimeSerializer());

    await Demo();
    Console.ReadKey();
}
```

## Dictionary字段

数据声明很简单：

`public Dictionary<string, int> extra_info { get; set; }`

`MongoDB` 定义了三种保存属性：`Document`、`ArrayOfDocuments`、`ArrayOfArrays`，默认是 `Document`。

属性写法是这样的：

```c#
[BsonDictionaryOptions(DictionaryRepresentation.ArrayOfDocuments)]
public Dictionary<string, int> extra_info { get; set; }
```

这三种属性下，保存在数据集中的数据结构有区别。多数情况用 `DictionaryRepresentation.ArrayOfDocuments`。

```json
// DictionaryRepresentation.Document：
{ 
    "extra_info" : {
        "type" : NumberInt(1), 
        "mode" : NumberInt(2)
    }
}


// DictionaryRepresentation.ArrayOfDocuments：
{ 
    "extra_info" : [
        {
            "k" : "type", 
            "v" : NumberInt(1)
        }, 
        {
            "k" : "mode", 
            "v" : NumberInt(2)
        }
    ]
}

// DictionaryRepresentation.ArrayOfArrays：
{ 
    "extra_info" : [
        [
            "type", 
            NumberInt(1)
        ], 
        [
            "mode", 
            NumberInt(2)
        ]
    ]
}
```

这三种方式，从数据保存上并没有什么区别，但从查询来讲，如果这个字段需要进行查询，那三种方式区别很大。

如果采用 `BsonDocument` 方式查询，`DictionaryRepresentation.Document` 无疑是写着最方便的。

如果用 `Builder` 方式查询，`DictionaryRepresentation.ArrayOfDocuments` 是最容易写的。

`DictionaryRepresentation.ArrayOfArrays` 就算了。数组套数组，查询条件写死人。

## BsonElement属性

用来改数据集中的字段名称。

```c#
[BsonElement("pt")]
public DateTime post_time { get; set; }
```

在不加 `BsonElement` 的情况下，通过数据映射写到数据集中的文档，字段名就是变量名，上面这个例子，字段名就是 `post_time` 。

加上 `BsonElement` 后，数据集中的字段名会变为 `pt`。

## BsonDefaultValue属性

设置字段的默认值。
当写入的时候，如果映射中不传入值，则数据库会把这个默认值存到数据集中。

```c#
[BsonDefaultValue("This is a default title")]
public string title { get; set; }
```

## BsonRepresentation属性 

用来在映射类中的数据类型和数据集中的数据类型做转换。

```c#
[BsonRepresentation(BsonType.String)]
public int favor { get; set; }
```

表示，在映射类中，`favor` 字段是 `int` 类型的，而存到数据集中，会保存为 `string` 类型。

前边 `Decimal` 转换和枚举转换，就是用的这个属性。

## BsonIgnore属性

用来忽略某些字段。
忽略的意思是：映射类中某些字段，不希望被保存到数据集中。

```c#
[BsonIgnore]
public string ignore_string { get; set; }
```

这样，在保存数据时，字段 `ignore_string` 就不会被保存到数据集中。

参考：

[MongoDB via Dotnet Core数据映射详解](https://mp.weixin.qq.com/s/M3zdBDc2ci5xfL-fazTH7A)
