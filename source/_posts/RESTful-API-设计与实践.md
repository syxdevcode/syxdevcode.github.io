---
title: RESTful API 设计与实践
date: 2019-03-18 13:51:49
tags:
- RESTful
categories: 
- RESTful
---
# RESTful API 设计与实践

## 协议

API与用户的通信协议，总是使用HTTPs协议。

## 域名

应该尽量将API部署在专用域名之下。

```url
https://api.example.com
```

如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。

```url
https://example.org/api/
```

## 版本（Versioning）

应该将API的版本号放入URL。

```url
https://api.example.com/v1/
```

另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观。

## 路径（Endpoint）

路径又称"终点"（endpoint），表示API的具体网址。

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。

例如：

```url
https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees
```

## HTTP动词

对于资源的具体操作类型，由HTTP动词表示。

常用的HTTP动词有下面五个（括号里是对应的SQL命令）。

根据 HTTP 规范，动词一律大写。

```text
GET（SELECT）：读取（Read）
POST（CREATE）：新建（Create）
PUT（UPDATE）：更新（Update）
PATCH（UPDATE）：更新（Update），通常是部分更新
DELETE（DELETE）：删除（Delete）
```

还有两个不常用的HTTP动词。

```text
HEAD：获取资源的元数据。
OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
```

例子：

```text
GET /zoos：列出所有动物园
POST /zoos：新建一个动物园
GET /zoos/ID：获取某个指定动物园的信息
PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID：删除某个动物园
GET /zoos/ID/animals：列出某个指定动物园的所有动物
DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
```

## URL 设计

### 动词 + 宾语

RESTful 的核心思想就是，客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，`GET/articles`这个命令，`GET`是动词，`/articles`是宾语。

### 动词的覆盖

有些客户端只能使用`GET`和`POST`这两种方法。服务器必须接受`POST`模拟其他三个方法（`PUT`、`PATCH`、`DELETE`）。

这时，客户端发出的 HTTP 请求，要加上`X-HTTP-Method-Override`属性，告诉服务器应该使用哪一个动词，覆盖`POST`方法。

```html
POST /api/Person/4 HTTP/1.1  
X-HTTP-Method-Override: PUT
```

`X-HTTP-Method-Override`指定本次请求的方法是`PUT`，而不是`POST`。

### 宾语必须是名词

宾语就是 API 的 URL，是 HTTP 动词作用的对象。它应该是名词，不能是动词。比如，/articles这个 URL 就是正确的。

`/getAllCars` 是错误的。

### 复数 URL

建议都使用复数 URL，比如`GET/articles/2`要好于`GET/article/2`。

### 避免多级 URL

```url
// URL 不利于扩展，语义也不明确
GET /authors/12/categories/2

// 更好的写法
GET /authors/12?categories=2
```

## 过滤信息（Filtering）

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数：

```txt
?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&per_page=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?animal_type_id=1：指定筛选条件
```

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，`GET /zoo/ID/animals` 与 `GET /animals?zoo_id=ID` 的含义是相同的。

## 状态码（Status Codes）

### 状态码必须精确

客户端的每一次请求，服务器都必须给出回应。回应包括 HTTP 状态码和数据两部分。

HTTP 状态码就是一个三位数，分成五个类别。API 不需要1xx状态码。

```url
1xx：相关信息
2xx：操作成功
3xx：重定向
4xx：客户端错误
5xx：服务器错误
```

详细请参考：[www.w3.org](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

### 2xx 状态码

200状态码表示操作成功，但是不同的方法可以返回更精确的状态码。

```html
GET: 200 OK
POST: 201 Created
PUT: 200 OK
PATCH: 200 OK
DELETE: 204 No Content
```

`POST`返回201状态码，表示生成了新的资源；`DELETE`返回204状态码，表示资源已经不存在。

`202 Accepted`状态码表示服务器已经收到请求，但还未进行处理，会在未来再处理，通常用于异步操作。

### 3xx 状态码

API 用不到`301`状态码（永久重定向）和`302`状态码（暂时重定向，307也是这个含义），因为它们可以由应用级别返回，浏览器会直接跳转，API 级别可以不考虑这两种情况。

API 用到的`3xx`状态码，主要是`303 See Other`，表示参考另一个 URL。它与`302`和`307`的含义一样，也是"暂时重定向"，区别在于`302`和`307`用于`GET`请求，而`303`用于`POST`、`PUT`和`DELETE`请求。收到`303`以后，浏览器不会自动跳转，而会让用户自己决定下一步怎么办。下面是一个例子。

```html
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

### 4xx 状态码

4xx状态码表示客户端错误，主要有下面几种。

**400 Bad Request**：服务器不理解客户端的请求，未做任何处理。

**401 Unauthorized**：用户未提供身份验证凭据，或者没有通过身份验证。

**403 Forbidden**：用户通过了身份验证，但是不具有访问资源所需的权限。

**404 Not Found**：所请求的资源不存在，或不可用。

**405 Method Not Allowed**：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。

**410 Gone**：所请求的资源已从这个地址转移，不再可用。

**415 Unsupported Media Type**：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式。

**422 Unprocessable Entity**：客户端上传的附件无法处理，导致请求失败。

**429 Too Many Requests**：客户端的请求次数超过限额。

### 5xx 状态码

`5xx状态码表示服务端错误。一般来说，API 不会向用户透露服务器的详细信息，所以只要两个状态码就够了。

**500 Internal Server Error**：客户端请求有效，服务器处理时发生了意外。

**503 Service Unavailable**：服务器无法处理请求，一般用于网站维护状态。

## 服务器回应

### 不要返回纯本文

API 返回的数据格式，不应该是纯文本，而应该是一个 JSON 对象，因为这样才能返回标准的结构化数据。所以，服务器回应的 HTTP 头的`Content-Type`属性要设为`application/json`。

客户端请求时，也要明确告诉服务器，可以接受 JSON 格式，即请求的 HTTP 头的`ACCEPT`属性也要设成`application/json`。

```html
GET /orders/2 HTTP/1.1 
Accept: application/json
```

### 错误处理（Error handling）

发生错误时，不要返回 200 状态码，状态码反映发生的错误，具体的错误信息放在数据体里面返回。

```html
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid payoad.",
  "detail": {
     "surname": "This field is required."
  }
}
```

### 返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范。

```html
GET /collection：返回资源对象的列表（数组）
GET /collection/resource：返回单个资源对象
POST /collection：返回新生成的资源对象
PUT /collection/resource：返回完整的资源对象
PATCH /collection/resource：返回完整的资源对象
DELETE /collection/resource：返回一个空文档
```

### Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法。

Hypermedia API的设计被称为HATEOAS。

例子：

```html
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
```

参考：

[RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)