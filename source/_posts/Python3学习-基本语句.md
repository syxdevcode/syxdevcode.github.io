---
title: Python3学习-基本语句
date: 2021-01-16 11:35:24
tags:
- Python3
- Python
categories: Python3
---

## 条件语句

if、elif、else 来进行逻辑判断

格式：

```py
if 判断条件1:
    执行语句1...
elif 判断条件2:
    执行语句2...
elif 判断条件3:
    执行语句3...
else:
    执行语句4...
```

## 循环语句

### for 循环

for 循环可以遍历任何序列

```py
str = 'Python'
for s in str:
    print(s)

# 或
for i in range(5,9) :
    print(i)
```

输出：

```py
P
y
t
h
o
n
```

###  while 循环

```py
sum = 0
m = 10
while m > 0:
    sum = sum + m
    m = m - 1
print(sum)
```

**while 循环使用 else 语句**

在 while … else 在条件语句为 false 时执行 else 的语句块。

```py
while <expr>:
    <statement(s)>
else:
    <additional_statement(s)>
```

### break

break 用在 for 循环和 while 循环语句中，用来终止整个循环。

### continue

continue 用在 for 循环和 while 循环语句中，用来终止本次循环。

### pass 语句

pass 是空语句，它不做任何事情，一般用做占位语句，作用是保持程序结构的完整性。

```py
if True:
    pass
```