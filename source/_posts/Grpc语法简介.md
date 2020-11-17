---
title: Grpc语法简介
date: 2020-11-16 16:23:11
tags:
- DotNetCore
- Grpc
- Protobuf
categories:
- Grpc
---

## 定义消息类型

先看一个 proto3 的查找请求参数的消息格式的例子，这个请求参数例子模仿分页查找请求，他有一个请求参数字符串，有一个当前页的参数还有一个每页返回数据大小的参数，proto 文件内容如下：

```c#
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

* 第一行的含义是限定该文件使用的是 proto3 的语法，如果没有，则默认为 syntax = "proto3";
* SearchRequest 定义有三个承载消息的属性，每一个被定义在 SearchRequest 消息体中的字段，都是由 数据类型 和 属性名称 组成。

### 指定字段类型 

在上面的例子中，所有的属性都是标量，两个整型 ( page_number 、 result_per_page )和一个字符串( query )，你还可以在指定复合类型，包括 枚举类型 或者 其他的消息类型。

### 分配标量

每一个被定义在消息中的字段都会被分配给一个唯一的标量，这些标量用于标识你定义在二进制消息格式中的属性，标量一旦被定义就不允许在使用过程中再次被改变。标量的值在1～15的这个范围里占一个字节编码

### 指定属性规则

消息属性规则如下：

* singular: 一个正确的消息可以有零个或者多个这样的消息属性(但是不要超过一个).
* repeated: 这个属性可以在一个正确的消息格式中重复任意次数(包括零次),

在 proto3 中，标量数字类型的重复字段默认使用压缩编码。

### 添加更多的消息类型

在一个 proto 文件中可以定义多个消息类型，你可以在一个文件中定义一些相关的消息类型，上面的例子 proto 文件中只有一个请求查找的消息类型，现在可以为他多添加一个响应的消息类型，具体如下：

```c#
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
    ....
}
```

### 添加注释

proto 文件中的注释使用的是 /c++ 中的单行注释 // 语法风格。

如下：

```C#
message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // 当前页数
  int32 result_per_page = 3;  // 每页数据返回的数据量
```

### 保留属性

为了避免在加载相同的 .proto 的旧版本，包括数据损坏，隐含的错误等，这可能会导致严重的问题的方法是指定删除的字段的字段标签（和/或名称，也可能导致 JSON 序列化的问题）被保留。 如果将来的用户尝试使用这些字段标识符，协议缓冲区编译器将会报错。

保留字段的使用例子:

```C#
message Foo {
  reserved 2;
  reserved "foo", "bar";
}
```

上述例子定义保留属性为 "foo", "bar" ，定义保留属性位置为2，即在2这个位置上不可以定义属性，如: string name=2; 是不允许的，编译器在编译 proto 文件的时候如果发现，2这个位置上有属性被定义则会报错。

## 数据类型

.proto文件的值类型和 java 的值类型的对照表:

```C#
.proto Type	Java Type
double	    double
float	    float
int32	    int
int64	    long
uint32	    int
uint64	    long
sint32	    int
sint64	    long
fixed32	    int
fixed64	    long
sfixed32	int
sfixed64	long
bool	    boolean
string	    String
bytes	    ByteString
```

## 默认值

当 proto 消息被解析成具体的语言的时候，如果消息编码没包含特定的元素，则消息对象中的属性会被设置默认值，这些默认值具体如下：

* string 类型,默认值是空字符串,注意不是null
* bytes 类型,默认值是空bytes
* bool 类型，默认值是false
* 数字 类型,默认值是0
* 枚举 类型,默认值是第一个枚举值,即0
* repeated 修饰的属性，默认值是空（在相对应的编程语言中通常是一个空的list）.

## 枚举

proto 允许你在定义的消息类型的时候定义枚举类型，如下例,在消息类型中定义并使用枚举类型：

```c#
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

如上例中所示， Corpus 枚举类型的第一个枚举值是0，每一个枚举值定义都会与一个常量映射，而这些常量的第一个常量值必须为0，原因如下：

必须有一个0作为值，以至于我们可是使用0作为默认值
第一个元素的值取0，用于与第一个元素枚举值作为默认值的proto2语义兼容
枚举类型允许你定义别名，别名的作用是分配不中的标量，使用相同的常量值，使用别名只需要在定义枚举类型的第一行中添加 allow_alias 选项，并将值设置为 true 即可，如果没有设置该值就是用别名，在编译的时候会报错。

官网例子如下：

```c#
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  //如果解除这个注释编译器在编译该proto文的时候会报错
  // RUNNING = 1;  
}
```

proto 支持的枚举值的范围是 32 位的整形，即 Java 中的 int 类型,其他请参看官网。

## 引用其他的消息类型

你可以在定义消息类型的时候饮用其他已经定义好的消息类型作为新消息类型的属性，官网例子如下：

```c#
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

在上面的消息例子中，SearchResponse 这个响应消息类型的属性 results，返回的是一个 Result 类型的消息列表。

###  导入其他proto中定义的消息

在上面的例子中，Result 和 SearchResponse 消息类型被定义在同一个 .proto 文件中，如果把他们分成两个文件定义，应该如何引用呢？

proto 中为我们提供了 import 关键字用于引入不同 .proto 文件中的消息类型,你可以在你的 .proto 文件的顶部加入如下语句因为其他 .proto 文件的消息类型：`import "myproject/other_protos.proto"`;

例子：

* 文件名称 search_response.proto

```C#
syntax = "proto3";
import "test/result.proto";
package test1;

message SearchResponse {
  //包名.消息名
  repeated test2.Result results = 1;
}
```

* 文件名称 result.proto,在与search_response.proto同级目录的test下

```C#
syntax = "proto3";
package test2;

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

如果两个 .proto 文件在同一个目录下直接这样 import "result.proto"; 倒入即可。

## 内嵌类型

我们还可以在消息类型中定义消息，例子如下：

```C#
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

在上面的例子中在 SearchResponse 消息体中定义了一个 Result 消息并使用。

如果想在其他的消息体引用 Result 这个消息，可以 Parent.Type 这样引用，例子：

```c#
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

消息还可以深层的嵌套定义，如下例子：

```C#
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## Map

proto 支持 map 属性类型的定义，语法如下：

```C#
map<key_type,value_type> map_field = N;
```

key_type 可以是任何整数或字符串类型（除浮点类型和字节之外的任何标量类型,枚举类型也是不合法的key类型），value_type可以是任何类型的数据。

## 包

可以为 proto 文件指定包名，防止消息命名冲突。

例子如下：

```C#
package foo.bar;
message Open { ... }
```

当你在为消息类型定义属性的时候，你可以通过 命名.类型 的形式来使用已经定义好的消息类型，如下：

```C#
Message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

## 服务定义

如果你想在 RPC 中使用已经定义好的消息类型，你可以在 .proto 文件中定一个消息服务接口, protocol buffer 编译器会生成对应语言的接口代码。

接口定义例子：

```C#
service SearchService {
    //  方法名  方法参数                 返回值
    rpc Search(SearchRequest) returns (SearchResponse); 
}
```

## 选项

* file_extension：文件扩展
* base_namespace：命名空间

参考：

[https://developers.google.com/protocol-buffers/docs/reference/csharp-generated#compiler_options](https://developers.google.com/protocol-buffers/docs/reference/csharp-generated#compiler_options)