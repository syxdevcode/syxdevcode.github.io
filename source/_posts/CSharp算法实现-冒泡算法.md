---
title: CSharp算法实现-冒泡算法
date: 2019-03-21 14:01:03
tags:
- CSharp基础
- 算法
- DotNet面试题解析
categories: 
- 算法
description: CSharp算法实现-冒泡算法的实现
---
比较相邻的元素。如果第一个比第二个大（小），就交换他们两个。

根据时间复杂度的计算原则，冒泡排序为：O(n^2 )。

```cs
// 从小到大排序
public static void BubbleSort(List<int> list)
{
    var temp = 0;
    for (var j = list.Count; j > 0; j--)
    {
        for (int i = 0; i < j - 1; i++)
        {
            if (list[i] > list[i + 1])
            {
                temp = list[i + 1];
                list[i + 1] = list[i];
                list[i] = temp;
            }
        }
    }
}

public static void BulleSort1(List<int> list)
{
    var temp = 0;
    for (var j = 0; j < list.Count; j++)
    {
        for (int i = 0; i < list.Count - j - 1; i++)
        {
            if (list[i] > list[i + 1])
            {
                temp = list[i + 1];
                list[i + 1] = list[i];
                list[i] = temp;
            }
            //PrintList(list);
        }
    }
}

private static void PrintList(List<int> list)
{
    foreach (var item in list)
    {
        Console.Write(string.Format("{0} ", item));
    }
    Console.WriteLine();
}
```
