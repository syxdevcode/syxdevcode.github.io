---
title: MongoDB 高级操作
date: 2020-09-25 09:53:19
tags:
- Linux
- NoSQL
- CentOS7
- Windows
- MongoDB
categories:
- MongoDB
---

## ObjectId

ObjectId 是一个12字节 `BSON` 类型数据，有以下格式：

* 前4个字节表示时间戳
* 接下来的3个字节是机器标识码
* 紧接的两个字节由进程id组成（PID）
* 最后三个字节是随机数。

`MongoDB` 中存储的文档必须有一个 `_id` 键。这个键的值可以是任何类型的，默认是个 `ObjectId` 对象。

在一个集合里面，每个文档都有唯一的 `_id` 值，来确保集合里面每个文档都能被唯一标识。

`MongoDB` 采用 `ObjectId`，而不是其他比较常规的做法（比如自动增加的主键）的主要原因，因为在多个服务器上同步自动增加主键值既费力还费时。

```sh
# 1.创建新的ObjectId
newObjectId = ObjectId()
# 输出
ObjectId("5f6d62d8b7424d4010318ac6")

# 也可以使用生成的id来取代MongoDB自动生成的ObjectId：
myObjectId = ObjectId("5f6d62d8b7424d4010318ac6")

# 2.创建文档的时间戳
# 由于 ObjectId 中存储了4个字节的时间戳，所以你不需要为你的文档保存时间戳字段，
# 可以通过 getTimestamp 函数来获取文档的创建时间:
ObjectId("5f6d62d8b7424d4010318ac6").getTimestamp()
# 输出
ISODate("2020-09-25T03:24:08Z")

# 3.ObjectId 转换为字符串
new ObjectId().str
# 输出
5f6d6374b7424d4010318ac7
```

## 文档关系

`MongoDB` 中的关系分为：`嵌入式关系` 和 `引用式关系`。

### 嵌入式关系

```json
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "phone": "123345345",
   "name": "test",
   "address": [
      {
         "pincode": 123456,
         "city": "beijing",
         "state": "California"
      },
      {
         "pincode": 456789,
         "city": "shanghai",
         "state": "Illinois"
      }]
}
```

查询：

`db.users.findOne({"name":"test"},{"address":1})`

这种数据结构的缺点是，如果用户和用户地址在不断增加，数据量不断变大，会影响读写性能。

### 引用式关系

```json
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "test",
   "address_ids": [
      ObjectId("52ffc4a5d85242602e000000"),
      ObjectId("52ffc4a5d85242602e000001")
   ]
}
```

查询：

```sh
var result = db.users.findOne({"name":"test"},{"address_ids":1})
var addresses = db.address.find({"_id":{"$in":result["address_ids"]}})

# 或
var result = db.users.find({"name":"test"},{"address_ids":1})
var addresses = db.address.find({"_id":{"$in":result[0]["address_ids"]}})
```

注意：`find` 返回的数据类型是数组，`findOne` 返回的数据类型是对象。

## 引用关系

`MongoDB` 引用有两种：

* 手动引用（Manual References）
* DBRefs

### DBRefs

```sh
# $ref：集合名称
# $id：引用的id
# $db:数据库名称，可选参数
{ $ref : , $id : , $db :  }
```

实例：

```json
{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "runoob"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "test1"
}
```

查询：

```sh
var user = db.users.findOne({"name":"test1"})
var dbRef = user.address

db[dbRef.$ref].findOne({"_id":(dbRef.$id)})

# MongoDB4.0 版本写法
db[dbRef.$ref].findOne({"_id":ObjectId(dbRef.$id)})
```

## 覆盖索引查询

官方的 `MongoDB` 的文档中说明，覆盖查询是以下的查询：

* 所有的查询字段是索引的一部分
* 所有的查询返回字段在同一个索引中

由于所有出现在查询中的字段是索引的一部分， `MongoDB` 无需在整个数据文档中检索匹配查询条件和返回使用相同索引的查询结果。
因为索引存在于 `RAM` 中，从索引中获取数据比通过扫描文档读取数据要快得多。

### 使用覆盖索引查询

实例：

```json
{
   "_id": ObjectId("53402597d852426020000002"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "gender": "M",
   "name": "test1",
   "user_name": "tombenzamin"
}
```

在 `users` 集合中创建联合索引，字段为 `gender` 和 `user_name` :

`db.users.createIndex({gender:1,user_name:1})`

该索引会覆盖以下查询：

`db.users.find({gender:"M"},{user_name:1,_id:0})`

上述查询，`MongoDB` 的不会去数据库文件中查找。它会从索引中提取数据，这是非常快速的数据查询。

由于索引中不包括 `_id` 字段，`_id` 在查询中会默认返回，我们可以在 `MongoDB` 的查询结果集中排除它。

下面的实例没有排除 `_id`，查询就不会被覆盖：

`db.users.find({gender:"M"},{user_name:1})`

最后，如果是以下的查询，不能使用覆盖索引查询：

* 所有索引字段是一个数组
* 所有索引字段是一个子文档

## 查询分析

`MongoDB` 查询分析常用函数有：`explain()` 和 `hint()`。

### explain()

`explain` 操作提供了查询信息，使用索引及查询统计等。有利于我们对索引的优化。

```sh
# users 集合中创建 gender 和 user_name 的索引：
db.users.createIndex({gender:1,user_name:1})

# 在查询语句中使用 explain ：
db.users.find({gender:"M"},{user_name:1,_id:0}).explain()
```

 explain() 查询返回:

```json
{
   "cursor" : "BtreeCursor gender_1_user_name_1",
   "isMultiKey" : false,
   "n" : 1,
   "nscannedObjects" : 0,
   "nscanned" : 1,
   "nscannedObjectsAllPlans" : 0,
   "nscannedAllPlans" : 1,
   "scanAndOrder" : false,
   "indexOnly" : true,
   "nYields" : 0,
   "nChunkSkips" : 0,
   "millis" : 0,
   "indexBounds" : {
      "gender" : [
         [
            "M",
            "M"
         ]
      ],
      "user_name" : [
         [
            {
               "$minElement" : 1
            },
            {
               "$maxElement" : 1
            }
         ]
      ]
   }
}
```

结果集的字段：

* indexOnly: 字段为 true ，表示我们使用了索引。
* cursor：因为这个查询使用了索引，`MongoDB` 中索引存储在B树结构中，所以这是也使用了 `BtreeCursor` 类型的游标。如果没有使用索引，游标的类型是 `BasicCursor`。这个键还会给出你所使用的索引的名称，你通过这个名称可以查看当前数据库下的 `system.indexes` 集合（系统自动创建，由于存储索引信息，这个稍微会提到）来得到索引的详细信息。
* n：当前查询返回的文档数量。
* nscanned/nscannedObjects：表明当前这次查询一共扫描了集合中多少个文档，我们的目的是，让这个数值和返回文档的数量越接近越好。
* millis：当前查询所需时间，毫秒数。
* indexBounds：当前查询具体使用的索引。

### hint()

`hint()` 来强制 `MongoDB` 使用一个指定的索引。

```sh
db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1})

# 可以使用 explain() 函数来分析以上查询：
db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1}).explain()
```

## 原子操作

`MongoDB` 提供了许多原子操作，比如文档的保存，修改，删除等，都是原子操作。

所谓原子操作就是要么这个文档保存到 `MongoDB`，要么没有保存到 `MongoDB`，不会出现查询到的文档没有保存完整的情况。

`db.collection.findAndModify()` 方法来判断书籍是否可结算并更新新的结算信息。

```sh
db.books.findAndModify ( {
   query: {
            _id: 123456789,
            available: { $gt: 0 }
          },
   update: {
             $inc: { available: -1 },
             $push: { checkout: { by: "abc", date: new Date() } }
           }
} )
```

### 原子操作常用命令

**$set**

用来指定一个键并更新键值，若键不存在并创建。

`{ $set : { field : value } }`

**$unset**

用来删除一个键。

`{ $unset : { field : 1} }`

**$inc**

$inc可以对文档的某个值为数字型（只能为满足要求的数字）的键进行增减的操作。

`{ $inc : { field : value } }`

**$push**

把value追加到field里面去，field一定要是数组类型才行，如果field不存在，会新增一个数组类型加进去。

`{ $push : { field : value } }`

**$pushAll**

同 `$push`,只是一次可以追加多个值到一个数组字段内。

`{ $pushAll : { field : value_array } }`

**$pull**

从数组field内删除一个等于value值。

`{ $pull : { field : _value } }`

**$addToSet**

增加一个值到数组内，而且只有当这个值不在数组内才增加。

**$pop**

删除数组的第一个或最后一个元素

`{ $pop : { field : 1 } }`

**$rename**

修改字段名称

`{ $rename : { old_field_name : new_field_name } }`

**$bit**

位操作，integer类型

`{$bit : { field : {and : 5}}}`

**偏移操作符**

```sh
t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 3 }, { "by" : "jane", "votes" : 7 } ] }
 
# 重点
t.update( {'comments.by':'joe'}, {$inc:{'comments.$.votes':1}}, false, true )
 
t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 4 }, { "by" : "jane", "votes" : 7 } ] }
```

## 高级索引

实例 文档集合（users）包含了 `address` 子文档和 `tags` 数组：

```json
{
   "address": {
      "city": "Los Angeles",
      "state": "California",
      "pincode": "123"
   },
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}
```

### 索引数组字段

在数组中创建索引，需要对数组中的每个字段依次建立索引。所以在我们为数组 `tags` 创建索引时，会为 `music、cricket、blogs` 三个值建立单独的索引。

```sh
# 创建数组索引：

db.users.createIndex({"tags":1})

# 创建索引后，我们可以这样检索集合的 tags 字段：
db.users.find({tags:"cricket"})

# 为了验证我们使用使用了索引，可以使用 explain 命令：
# 执行结果中会显示 "cursor" : "BtreeCursor tags_1" ，则表示已经使用了索引。
db.users.find({tags:"cricket"}).explain()
```

### 索引子文档字段

```sh
# 为子文档的三个字段创建索引，命令如下：
db.users.createIndex({"address.city":1,"address.state":1,"address.pincode":1})

# 一旦创建索引，我们可以使用子文档的字段来检索数据：
db.users.find({"address.city":"Los Angeles"})   

# 查询表达不一定遵循指定的索引的顺序，mongodb 会自动优化。所以上面创建的索引将支持以下查询：
db.users.find({"address.state":"California","address.city":"Los Angeles"}) 

# 同样支持以下查询：
db.users.find({"address.city":"Los Angeles","address.state":"California","address.pincode":"123"})
```

## 索引限制

**额外开销**

每个索引占据一定的存储空间，在进行插入，更新和删除操作时也需要对索引进行操作。所以，如果你很少对集合进行读取操作，建议不使用索引。

**内存(RAM)使用**

由于索引是存储在内存( `RAM` )中,你应该确保该索引的大小不超过内存的限制。

如果索引的大小大于内存的限制，`MongoDB` 会删除一些索引，这将导致性能下降。

**查询限制**

索引不能被以下的查询使用：

正则表达式及非操作符，如 `$nin`, `$not`, 等。
算术运算符，如 `$mod`, 等。
`$where` 子句
所以，检测你的语句是否使用索引是一个好的习惯，可以用 `explain` 来查看。

**索引键限制**

从2.6版本开始，如果现有的索引字段的值超过索引键的限制，`MongoDB` 中不会创建索引。

**插入文档超过索引键限制**

如果文档的索引字段值超过了索引键的限制，`MongoDB` 不会将任何文档转换成索引的集合。与 `mongorestore` 和 `mongoimport` 工具类似。

**最大范围**

集合中索引不能超过64个
索引名的长度不能超过128个字符
一个复合索引最多可以有31个字段

## Map Reduce

`Map-Reduce` 是一种计算模型，简单的说就是将大批量的工作（数据）分解（`MAP`）执行，然后再将结果合并成最终结果（`REDUCE`）。

`MongoDB` 提供的 `Map-Reduce` 非常灵活，对于大规模数据分析也相当实用。

`MapReduce 命令基本语法`

```sh
db.collection.mapReduce(
   function() {emit(key,value);},  //map 函数
   function(key,values) {return reduceFunction},   //reduce 函数
   {
      out: collection,
      query: document,
      sort: document,
      limit: number
   }
)
```

使用 `MapReduce` 要实现两个函数 `Map` 函数和 `Reduce` 函数,`Map` 函数调用 `emit(key, value)`, 遍历 `collection` 中所有的记录, 将 `key` 与 `value` 传递给 `Reduce` 函数进行处理。

`Map` 函数必须调用 `emit(key, value)` 返回键值对。

参数说明:

* `map` ：映射函数 (生成键值对序列,作为 `reduce` 函数参数)。
* `reduce` 统计函数，`reduce` 函数的任务就是将 `key-values` 变成 `key-value` ，也就是把 `values` 数组变成一个单一的值 `value`。。
* `out` 统计结果存放集合 (不指定则使用临时集合，在客户端断开后自动删除)。
* `query` 一个筛选条件，只有满足条件的文档才会调用 `map` 函数。（ `query`,`limit`，`sort`可以随意组合）
* `sort` 和 `limit` 结合的 `sort` 排序参数（也是在发往 `map` 函数前给文档排序），可以优化分组机制
* `limit` 发往 `map` 函数的文档数量的上限（要是没有 `limit`，单独使用 `sort` 的用处不大）

以下实例在集合 `orders` 中查找 `status:"A"` 的数据，并根据 `cust_id` 来分组，并计算 `amount` 的总和。

![map-reduce.bakedsvg.svg](/img/map-reduce.bakedsvg.svg)

**使用MapReduce**

在 `posts` 集合中使用 `mapReduce` 函数来选取已发布的文章(status:"active")，并通过 `user_name` 分组，计算每个用户的文章数：

```sh
db.posts.mapReduce( 
   function() { emit(this.user_name,1); }, 
   function(key, values) {return Array.sum(values)}, 
      {  
         query:{status:"active"},  
         out:"post_total" 
      }
)
db.post_total.find()

# 输出
{ "_id" : "runoob", "value" : 1 }
{ "_id" : "mark", "value" : 4 }
```

## 全文检索

全文检索对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。MongoDB 从 2.4 版本开始支持全文检索，MongoDB 从3.2 版本以后添加了对中文索引的支持。

**启用全文检索**

MongoDB 在 2.6 版本以后是默认开启全文检索的，如果你使用之前的版本，你需要使用以下代码来启用全文检索:

```sh
db.adminCommand({setParameter:true,textSearchEnabled:true})

# 或者使用命令：
mongod --setParameter textSearchEnabled=true
```

**创建全文索引**

实例：posts 集合的文档数据，包含了文章内容（post_text）及标签(tags)：

```sh
{
   "post_text": "enjoy the mongodb articles on Runoob",
   "tags": [
      "mongodb",
      "runoob"
   ]
}
```

对 post_text 字段建立全文索引，这样我们可以搜索文章内的内容：

`db.posts.createIndex({post_text:"text"})`

**使用全文索引**

```sh
db.posts.find({$text:{$search:"runoob"}})

# 结果：
{ 
   "_id" : ObjectId("53493d14d852429c10000002"), 
   "post_text" : "enjoy the mongodb articles on Runoob", 
   "tags" : [ "mongodb", "runoob" ]
}
```

旧版本的 MongoDB，可以使用以下命令：

`db.posts.runCommand("text",{search:"runoob"})`

使用全文索引可以提高搜索效率。

**删除全文索引**

```sh
# 删除已存在的全文索引，可以使用 find 命令查找索引名：
db.posts.getIndexes()

# 执行以下命令来删除索引：
db.posts.dropIndex("post_text_text")
```