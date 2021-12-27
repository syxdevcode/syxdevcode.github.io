---
title: Object.keys与Object.values
date: 2021-12-07 10:59:11
tags:
- Nodejs
- JavaScript
- ES5
categories: 
- Nodejs
---

## Object.keys()

ES5 引入了Object.keys方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（ enumerable ）属性的键名。

返回值：一个表示给定对象的所有可枚举属性的字符串数组

```js
var obj = { foo: "bar", baz: 42 };
Object.keys(obj)
// ["foo", "baz"]

let arr = [1,2,3,4,5,6]
Object.keys(arr) // ["0", "1", "2", "3", "4", "5"]

let str = "saasd字符串"
Object.keys(str) // ["0", "1", "2", "3", "4", "5", "6", "7"]

// 常用
let person = {name:"张三",age:25,address:"深圳",getName:function(){}}
Object.keys(person).map((key)=>{
　　person[key] // 获取到属性对应的值，做一些处理
}) 
```

## Object.values()

Object.values 方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（ enumerable ）属性的键值。

Object.values只返回对象自身的可遍历属性。

```js
var obj = { foo: "bar", baz: 42 };
Object.values(obj)
// ["bar", 42]

// 返回数组的成员顺序
// 属性名为数值的属性，是按照数值大小，从小到大遍历的，因此返回的顺序是b、c、a。
var obj = { 100: 'a', 2: 'b', 7: 'c' };
Object.values(obj)
// ["b", "c", "a"]
```
