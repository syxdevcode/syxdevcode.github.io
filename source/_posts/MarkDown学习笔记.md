---
title: MarkDown学习笔记
date: 2017-02-08 11:03:52
tags: MarkDown
categories: MarkDown
---

不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。不需要额外标注这是 HTML 或是 Markdown；只要直接加标签就可以了。
要制约的只有一些 HTML 区块元素――比如 `<div>、<table>、<pre>、<p>` 等标签，必须在前后加上空行与其它内容区隔开，
还要求它们的开始标签与结尾标签不能用制表符或空格来缩进。Markdown 的生成器有足够智能，不会在 HTML 区块标签外加上不必要的 `<p>` 标签。

## 区块元素

### 字体颜色

```html
<font color=#ff0000 size=4 face="黑体">最大消息长度</font>
<font color=#0400ff size=4 face="黑体">最大消息长度</font>
<table><tr><td bgcolor=yellow>最大消息长度正好是 IP 中不会被分片处理的最大数据长度</td></tr></table>
```

<font color=#ff0000 size=4 face="黑体">最大消息长度</font>
<font color=#0400ff size=4 face="黑体">最大消息长度</font>
<table><tr><td bgcolor=yellow>最大消息长度</td></tr></table>

### 标题

* 使用#号表示标题，一级标题使用一个#，二级标题使用两个#，以此类推，共有六级标题；
* 使用=====表示高阶标题，使用---------表示次阶标题；

``` MarkDown
# 这是一级标题
## 这是二级标题
### 这是三级标题
###### 这是六级标题
这是高阶标题（效果和一级标题一样 ）
========
这是次阶标题（效果和二级标题一样）
--------------
```

`#` 和标题之间需要加一个空格。
===和---表示标题时，大于等于2个都可以标识。

### 区块引用

使用>表示引用，>>表示引用里面再套一层引用，依次类推

### 列表

* Markdown 支持有序列表和无序列表
* 无序列表使用星号、加号或是减号作为列表标记
* 有序列表则使用数字接着一个英文句点
* 如果要放代码区块的话，该区块就需要缩进两次，也就是 8 个空格或是 2 个制表符
* 在行首出现数字-句点-空白，要避免这样的状况，你可以在句点前面加上反斜杠

1986\. What a great season.

'1986\. What a great season.'

### 代码区块

* 要在 Markdown 中建立代码区块很简单，只要简单地缩进 4 个空格或是 1 个制表符就可以
* 使用`tab`上面的按键英文输入；

```MarkDown
Here is an example of AppleScript:
    tell application "Foo"
        beep
    end tell
```

### 分割线

* 可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。

例如：

```MarkDown
* * *
***
*****
- - -
---------------------------------------
```

## 区段元素

### 链接

* Markdown 支持两种形式的链接语法： 行内式和参考式两种形式。
* 不管是哪一种，链接文字都是用 [方括号] 来标记。
* 要建立一个行内式的链接，只要在方块括号后面紧接着圆括号并插入网址链接即可，如果你还想要加上链接的 title 文字，只要在网址后面，用双引号把 title 文字包起来即可，例如：

``` javascript
This is [an example](http://example.com/ "Title") inline link.
[This link](http://example.net/) has no title attribute.
```

会产生：

```MarkDown
<p>This is <a href="http://example.com/" title="Title">
an example</a> inline link.</p>
<p><a href="http://example.net/">This link</a> has no
title attribute.</p>
```

```MarkDown
This is [an example](http://example.com/ "Title") inline link.
This is [an example][foo] reference-style link.
[foo]: <http://example.com/>  "Optional Title Here"
```

### 强调

`*`  Markdown 使用星号（*）和底线（_）作为标记强调字词的符号，被 * 或 _ 包围的字词会被转成用`<em>` 标签包围，
用两个 * 或 _ 包起来的话，则会被转成 `<strong>`，例如

```MarkDown
*single asterisks*
_single underscores_
**double asterisks**
__double underscores__
```

效果：

```MarkDown
*single asterisks*

_single underscores_

**double asterisks**

__double underscores__
```

* 如果你的 * 和 _ 两边都有空白的话，它们就只会被当成普通的符号
* 如果要在文字前后直接插入普通的星号或底线，你可以用反斜线

### 代码

要标记一小段行内代码，你可以用反引号把它包起来（`）
如果要在代码区段内插入反引号，你可以用多个反引号来开启和结束代码区段：

```MarkDown
``There is a literal backtick (`) here.``
```

``There is a literal backtick (`) here.``

代码区段的起始和结束端都可以放入一个空白，起始端后面一个，结束端前面一个，这样你就可以在区段的一开始就插入反引号：

```MarkDown
A single backtick in a code span: `` ` ``
A backtick-delimited string in a code span: `` `foo` ``
```

A single backtick in a code span: `` ` ``

A backtick-delimited string in a code span: `` `foo` ``

### 图片

* Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式： 行内式和参考式。

行内式的图片语法看起来像是：

代码：

```MarkDown
![Alt text](http://img.1985t.com/uploads/attaches/2014/07/19716-qd52IG.jpg)

![Alt text](http://img.1985t.com/uploads/attaches/2016/11/108284-D2AwRri.jpg "Optional title")

![Alt text][id]

[id]: http://img.1985t.com/uploads/attaches/2016/11/106985-c9YZrBt.jpg  "Optional title attribute"
```

* 到目前为止， Markdown 还没有办法指定图片的宽高，如果你需要的话，你可以使用普通的 <img> 标签

### 表格

```
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
```

设置表格的对齐方式：

* -: 设置内容和标题栏居右对齐。
* :- 设置内容和标题栏居左对齐。
* :-: 设置内容和标题栏居中对齐。

示例：

**echo输出的字符串总结：**
 
```sh
|符号| 能否引用变量  |  能否引用转移符  |  能否引用文本格式符(如：换行符、制表符)|
|  ----  | ----  | ----| ---|
|  单引号  |  否  |  否 |  否  |
|  双引号  |  能  |  能 |  能  |
|  无引号  |   能 |  能 |  否  |     
```

输出：

|符号| 能否引用变量  |  能否引用转移符  |  能否引用文本格式符(如：换行符、制表符)|
|  ----  | ----  | ----| ---|
|  单引号  |  否  |  否 |  否  |
|  双引号  |  能  |  能 |  能  |
|  无引号  |   能 |  能 |  否  | 

## 其它

### 自动链接

Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用尖括号包起来，
Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样;

`<http://example.com/>`
<http://example.com/>
邮址的自动链接也很类似，只是 Markdown 会先做一个编码转换的过程，把文字字符转成 16 进
位码的 HTML 实体，这样的格式可以糊弄一些不好的邮址收集机器人:

`<address@example.com>`

 ```MarkDown
    <a href="&#x6D;&#x61;i&#x6C;&#x74;&#x6F;:&#x61;&#x64;&#x64;&#x72;&#x65;
&#115;&#115;&#64;&#101;&#120;&#x61;&#109;&#x70;&#x6C;e&#x2E;&#99;&#111;
&#109;">&#x61;&#x64;&#x64;&#x72;&#x65;&#115;&#115;&#64;&#101;&#120;&#x61;
&#109;&#x70;&#x6C;e&#x2E;&#99;&#111;&#109;</a>
```

Markdown 语法说明

参考：

[http://wowubuntu.com/markdown/][markdownlink]

[https://www.zybuluo.com/AntLog/note/63228][blogref]

[markdownlink]:http://wowubuntu.com/markdown/ "语法详情"
[blogref]:https://www.zybuluo.com/AntLog/note/63228 "参考博客"