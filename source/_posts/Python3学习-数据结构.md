---
title: Python3学习-数据结构
date: 2021-04-13 22:22:52
tags:
- Python3
- Python
categories: Python3
---

## 列表

Python中列表是可变的，这是它区别于字符串和元组的最重要的特点，一句话概括即：列表可以修改，而字符串和元组不能。

Python 中列表的方法：注意：类似 insert, remove 或 sort 等修改列表的方法没有返回值。

* list.append(x)：把一个元素添加到列表的结尾，相当于 a[len(a):] = [x]。
* list.extend(L)：通过添加指定列表的所有元素来扩充列表，相当于 a[len(a):] = L。
* list.insert(i, x)：在指定位置插入一个元素。第一个参数是准备插入到其前面的那个元素的索引，例如 a.insert(0, x) 会插入到整个列表之前，而 a.insert(len(a), x) 相当于 a.append(x) 。
* list.remove(x)：删除列表中值为 x 的第一个元素。如果没有这样的元素，就会返回一个错误。
* list.pop([i])：从列表的指定位置移除元素，并将其返回。如果没有指定索引，a.pop()返回最后一个元素。元素随即从列表中被移除。（方法中 i 两边的方括号表示这个参数是可选的，而不是要求你输入一对方括号，你会经常在 Python 库参考手册中遇到这样的标记。）
* list.clear()：移除列表中的所有项，等于del a[:]。
* list.index(x)：返回列表中第一个值为 x 的元素的索引。如果没有匹配的元素就会返回一个错误。
* list.count(x)：返回 x 在列表中出现的次数。
* list.sort()：对列表中的元素进行排序。
* list.reverse()：倒排列表中的元素。
* list.copy()：返回列表的浅复制，等于a[:]。

## 将列表当做堆栈使用

堆栈作为特定的数据结构，最先进入的元素最后一个被释放（后进先出）。用 append() 方法可以把一个元素添加到堆栈顶。用不指定索引的 pop() 方法可以把一个元素从堆栈顶释放出来。例如：

```py
>>> stack = [3, 4, 5]
>>> stack.append(6)
>>> stack.append(7)
>>> stack
[3, 4, 5, 6, 7]
>>> stack.pop()
7
>>> stack
[3, 4, 5, 6]
>>> stack.pop()
6
>>> stack.pop()
5
>>> stack
[3, 4]
```

## 将列表当作队列使用

在队列里第一加入的元素，第一个取出来；但是拿列表用作这样的目的效率不高。在列表的最后添加或者弹出元素速度快，然而在列表里插入或者从头部弹出速度却不快（因为所有其他的元素都得一个一个地移动）。

```py
>>> from collections import deque
>>> queue = deque(["Eric", "John", "Michael"])
>>> queue.append("Terry")           # Terry arrives
>>> queue.append("Graham")          # Graham arrives
>>> queue.popleft()                 # The first to arrive now leaves
'Eric'
>>> queue.popleft()                 # The second to arrive now leaves
'John'
>>> queue                           # Remaining queue in order of arrival
deque(['Michael', 'Terry', 'Graham'])
```

## 列表推导式











参考：

[Python3 数据结构](https://www.runoob.com/python3/python3-data-structure.html)