---
title: JS中三个点的用法(扩展运算符)
date: 2021-10-11 11:56:56
tags:
- JavaScript
- TypeScript
- ES6
categories: 
- JavaScript
---

扩展运算符，是在ES6中新增加的内容，它可以在函数调用/数组构造时，将`数组表达式`或者string在语法层面展开；还可以在构造字面量对象时将对象表达式按照`key-value`的方式展开。用于取出参数对象的所有可遍历属性，然后拷贝到当前对象之中。

字面量一般指[1,2,3]或者{name:'chuichui'}这种简洁的构造方式,多层嵌套的数组和对象三个点就无能为力了。

## 复制

```js
// 对象的复制
let person = {name: "Amy", age: 15}
let someone = { ...person }
someone  // {name: "Amy", age: 15}

// 数组的复制
let personarr = ["hello"]
let someonearr = { ...personarr }
someonearr  // ["hello"]

let foo = { ...['a', 'b', 'c'] };
foo
// {0: "a", 1: "b", 2: "c"}
```

### 空对象

如果扩展运算符后面是一个空对象，则没有任何效果。

```js
let a = {...{}, a: 1}
a // { a: 1 }
```

### Int类型、Boolen类型、undefined、null

如果扩展运算符后面是上面这几种类型，都会返回一个空对象，因为它们没有自身属性。

```js
// 等同于 {...Object(1)}
{...1} // {}

// 等同于 {...Object(true)}
{...true} // {}

// 等同于 {...Object(undefined)}
{...undefined} // {}

// 等同于 {...Object(null)}
{...null} // {}
```

### 字符串数组

如果扩展运算符后面是字符串，它会自动转成一个类似数组的对象

```js
{...'hello'}
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```

## 合并

### 对象合并

```js
let age = {age: 15}
let name = {name: "Amy"}
let person = {...age, ...name}
person; // {age: 15, name: "Amy"}
```

**注意事项：**

自定义的属性和拓展运算符对象里面属性的相同的时候：

**自定义的属性在拓展运算符后面，则拓展运算符对象内部同名的属性将被覆盖掉。**

```js
let person = {name: "Amy", age: 15};
let someone = { ...person, name: "Mike", age: 17};
someone;  //{name: "Mike", age: 17}
```

**自定义的属性在拓展运算度前面，则变成设置新对象默认属性值。**

```js
let person = {name: "Amy", age: 15};
let someone = {name: "Mike", age: 17, ...person};
someone;  //{name: "Amy", age: 15}
```

### 数组合并

```js
var arr1=['hello'];
var arr2=['world'];
var mergeArr=[...arr1,...arr2];
mergeArr //['hello','world'];
```
