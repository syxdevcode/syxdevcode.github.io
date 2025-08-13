---
title: CSharp集合-HashSet与List区别
date: 2018-12-27 09:59:34
tags:
- DotNet
- CSharp
- 集合与泛型
- CSharp基础
categories: 
- 集合与泛型
---

`HashSet<T>` 是把对象保存在一个Hash表中的，查找时根据对象的Hash值直接定位对象；
`List<T>` 是把对象放到数组里，查找时是一个一个元素遍历的。

`List<T>` 里面是线序集，`HashSet<T>` 里面是散列表。与 `Dictionary<K,V>` 相比，`List<T>` 可以看成下标到值的映射，`HashSet<T>` 可以看成值自己到自己的映射。
判断一个值是否存在，`List<T>` 相当于是用值去找下标，要遍历一遍容器；
`HashSet<T>` 相当于用映射前的值去找映射后的值，只需要计算出来值的hash，然后直接访问就可以。

`HashSet<T>` 时间复杂度O(1)，
`List<T>` 时间复杂度O(n)。

而且 `HashSet<T>` 无序不重，和 `List<T>` 完全不同。


按特定顺序存储项的集合 `List<T>`。

快速确定集合中是否包含某个对象 `HashSet<T>`。
