---
title: Python3-JSON操作
date: 2021-04-29 13:31:46
tags:
---

Python3 中可以使用 json 模块来对 JSON 数据进行编解码，它包含了两个函数：

* json.dumps(): 对数据进行编码。
* json.loads(): 对数据进行解码。

json 模块主要提供了 `dump`、`dumps`、`load`、`loads` 方法对 JSON 数据进行编解码。

<!--more-->
## JSON操作

### dumps

json 模块的 dumps 方法可以将 Python 对象转为 JSON 格式字符串。

```py
import json

d = {'id':'001', 'name':'张三', 'age':'20'}
j = json.dumps(d, ensure_ascii=False)
print(j)
```

dumps 方法对数据进行格式化

```py
import json

d = {'id': '001', 'name': '张三', 'age': '20'}
j = json.dumps(d, ensure_ascii=False, sort_keys=True,
               indent=4, separators=(',', ': '))
print(j)
```

将 JSON 数据写入文件：

```py
import json

d = {'id':'001', 'name':'张三', 'age':'20'}
j = json.dumps(d, ensure_ascii=False, sort_keys=True, indent=4, separators=(',', ': '))
with open('test.json', 'w', encoding='utf-8') as f:
    f.write(j)
```

### dump

json 模块的 dump 方法可以将 Python 对象序列化为 JSON 格式化流形式的文件类对象。

```py
import json

d = {'id': '001', 'name': '张三', 'age': '20'}
with open('test.json', 'w', encoding='utf-8') as f:
    json.dump(d, f, indent=4, ensure_ascii=False)
```

### loads

JSON 格式数据转为 Python 对象:

```py
import json

j = '{"id":"001", "name":"张三", "age":"20"}'
d = json.loads(j)
print(d)
```

读取 test.json 中数据并将其转为 Python 对象：

```py
import json

with open('test.json', encoding='utf-8') as f:
    data = f.read()
    print(json.loads(data))
```

### load

json 模块的 load 方法将文件类对象转为 Python 对象:

```py
import json

with open('test.json', encoding='utf-8') as f:
    print(json.load(f))
```