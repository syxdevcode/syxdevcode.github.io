---
title: MongoDB在C#中使用
date: 2020-09-29 14:31:05
tags:
- DotNetCore
- CSharp
- MongoDB
categories:
- MongoDB
---

## 示例

```c#
using MongoDB.Bson;
using MongoDB.Driver;
using System;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;

namespace MongoDBTest
{
    internal class Program
    {
        private const string MongoDBConnection = "mongodb://localhost:27017/admin";

        private static IMongoClient _client = new MongoClient(MongoDBConnection);
        private static IMongoDatabase _database = _client.GetDatabase("Test");
        private static IMongoCollection<CollectionModel> _collection = _database.GetCollection<CollectionModel>("TestCollection");

        private static async Task Main(string[] args)
        {
            // 添加
            await InsertOneAsyncTest();

            Console.WriteLine("-----------FindAsyncTest输出---------------");
            await FindAsyncTest();

            Console.WriteLine("-----------Query输出---------------");
            await Query("a");

            Console.WriteLine("-----------SkipAndSortAndLimitAndProjections输出---------------");
            var request = new PageRequest {PageIndex = 1, PageSize = 2};
            await SkipAndSortAndLimitAndProjections(request);

            Console.ReadKey();
        }

        /// <summary>
        /// 查询，支持模糊查询
        /// </summary>
        /// <param name="keyword"></param>
        /// <returns></returns>
        private static async Task Query(string keyword)
        {
            FilterDefinitionBuilder<CollectionModel> builder = Builders<CollectionModel>.Filter;
            string p = keyword == null ? $".*{Regex.Escape("")}.*" : $".*{Regex.Escape(keyword)}.*";

            //1,条件为空
            //FilterDefinition<CollectionModel> filter2 = FilterDefinition<CollectionModel>.Empty;

            //2,不区分大小写
            FilterDefinition<CollectionModel> filter2 = builder.Regex("title", new BsonRegularExpression(new Regex(p, RegexOptions.IgnoreCase)));

            // 以下两种写法区分大小写
            //FilterDefinition<CollectionModel> filter2 = builder.Regex("title", new BsonRegularExpression(p));
            //BsonDocument filter2 = new BsonDocument { { "title", $"/{keyword}/" } };

            FindOptions<CollectionModel> options = new FindOptions<CollectionModel>
            {
                BatchSize = 2,
                NoCursorTimeout = false
            };
            var cursor = await _collection.FindAsync(filter2, options);
            var batch = 0;
            while (await cursor.MoveNextAsync())
            {
                batch++;
                Console.WriteLine($"Batch: {batch}");
                IEnumerable<CollectionModel> documents = cursor.Current;
                foreach (CollectionModel document in documents)
                {
                    Console.WriteLine(document.ToJson());
                    Console.WriteLine();
                }
            }
        }

        /// <summary>
        /// Skip, Sort, Limit, Projections
        /// </summary>
        /// <param name="request"></param>
        /// <returns></returns>
        private static async Task SkipAndSortAndLimitAndProjections(PageRequest request)
        {
            var count = 0;
            await _collection.Find(FilterDefinition<CollectionModel>.Empty)
                .Skip((request.PageIndex - 1) * request.PageSize)
                .Limit(request.PageSize)
                //.Sort(Builders<CollectionModel>.Sort.Descending("title"))
                //.Sort(Builders<CollectionModel>.Sort.Descending(x => x.title).Ascending(x => x.content))
                .Sort("{title: 1}")
                .Project(x => new { x.m_id, x.title, x.content, x.userinfo })
                .ForEachAsync(
                    obj =>
                    {
                        Console.WriteLine($"S/N: {count}, \t Id: {obj.m_id}, title: {obj.title}, content: {obj.content}, serinfo.name: {obj?.userinfo?.name}");
                        count++;
                    });
        }

        private static async Task InsertOneAsyncTest()
        {
            CollectionModel model = new CollectionModel();
            model.title = "A1";
            model.content = "可以测试";
            model.userinfo = new UserModel
            {
                name = "测试",
                addressList = new List<AddressModel>
                {
                    new AddressModel { address = "北京" },
                    new AddressModel { address = "上海" }
                }
            };
            await _collection.InsertOneAsync(model);
        }

        private static async Task FindAsyncTest()
        {
            using IAsyncCursor<CollectionModel> cursor = await _collection.FindAsync(new BsonDocument());
            while (await cursor.MoveNextAsync())
            {
                IEnumerable<CollectionModel> batch = cursor.Current;
                foreach (CollectionModel document in batch)
                {
                    Console.WriteLine(document.ToJson());
                    Console.WriteLine();
                }
            }
        }
    }
}
```

## 公共类

做通用查询的时候，可以使用。

```c#
/// <summary>
/// 公共页面请求参数
/// </summary>
public class PageRequest
{
    public int PageIndex { get; set; }

    public int PageSize { get; set; }

    public string Order { get; set; }
}

/// <summary>
/// 公用页面参数
/// </summary>
public class ParameterBase
{
    public string CollectName { get; set; }

    public int PageIndex { get; set; }

    public int PageSize { get; set; }
}

/// <summary>
/// 公用页面参数-MongoDB
/// </summary>
public class PageMongoParameter : ParameterBase
{
    public BsonDocument Sort { get; }
    public BsonDocument Skip { get; }
    public BsonDocument Limit { get; }
    public BsonDocument Filter2 { get; }

    public BsonDocument[] Pipeline { get; set; }

    public PageMongoParameter(PageRequest req, string collectName, BsonDocument filter = null, 
            BsonDocument sort = null, BsonDocument filter2 = null)
    {
        CollectName = collectName;
        PageIndex = req.PageIndex;
        PageSize = req.PageSize;
        Limit = new BsonDocument { { "$limit", PageSize } };
        Skip = new BsonDocument { { "$skip", (PageIndex - 1) * PageSize } };

        if (sort == null)
        {
            if (!string.IsNullOrEmpty(req.Order)) Sort = new BsonDocument { { "$sort", new BsonDocument { { req.Order, 1 } } } };
        }
        else
        {
            Sort = sort;
        }

        if (filter != null && filter2 == null)
        {
            var dom = filter.GetElement(0);
            Filter2 = BsonDocument.Parse(dom.Value.ToJson());
        }
        Filter2 ??= new BsonDocument();

        if (filter == null && Sort == null)
            Pipeline = new[] { Skip, Limit };
        else
        {
            if (filter != null && Sort != null)
                Pipeline = new[] { filter, Skip, Limit, Sort };
            else
                Pipeline = filter == null ? new[] { Skip, Limit, Sort } : new[] { filter, Skip, Limit };
        }
    }
}
```