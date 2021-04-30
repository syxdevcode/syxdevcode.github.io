---
title: Python3-Excel操作
date: 2021-04-29 16:32:35
tags:
- Python3
- Python
categories: Python3
---

Python 中常用 Excel 操作库如下：

* pandas：数据处理是 pandas 的立身之本，Excel 作为 pandas 输入/输出数据的容器。[https://www.pypandas.cn/docs/](https://www.pypandas.cn/docs/)
* xlutils：提供了一些 Excel 的实用操作，比如复制、拆分、过滤等，通常与 xlrd、xlwt 一起使用。缺点是仅支持 xls 文件。[https://pypi.org/project/xlutils/](https://pypi.org/project/xlutils/)
* XlsxWriter：拥有丰富的特性，支持图片/表格/图表/筛选/格式/公式等，功能与openpyxl相似，优点是相比 openpyxl 还支持 VBA 文件导入，迷你图等功能，缺点是不能打开/修改已有文件，意味着使用 xlsxwriter 需要从零开始。[https://xlsxwriter.readthedocs.io/](https://xlsxwriter.readthedocs.io/)
* openpyxl：简单易用，功能广泛，单元格格式/图片/表格/公式/筛选/批注/文件保护等等功能应有尽有，图表功能是其一大亮点，缺点是对 VBA 支持的不够好。[https://openpyxl.readthedocs.io/en/latest/tutorial.html](https://openpyxl.readthedocs.io/en/latest/tutorial.html)
* xlwings：可结合 VBA 实现对 Excel 编程，强大的数据输入分析能力，同时拥有丰富的接口，结合 pandas/numpy/matplotlib 轻松应对 Excel 数据处理工作。[https://docs.xlwings.org/en/stable/](https://docs.xlwings.org/en/stable/)

<!--more-->

## 写入

### 使用 xlwt

通过 `pip install xlwt` 命令安装。

```py
import xlwt

# 创建工作簿
wb = xlwt.Workbook()
# 创建表单
sh = wb.add_sheet('test')
# 创建字体对象
font = xlwt.Font()
# 字体加粗
font.bold = True
alm = xlwt.Alignment()
# 设置左对齐
alm.horz = 0x01
# 创建样式对象
style1 = xlwt.XFStyle()
style2 = xlwt.XFStyle()
style1.font = font
style2.alignment = alm
# write 方法参数1：行，参数2：列，参数3：内容
sh.write(0, 1, '姓名', style1)
sh.write(0, 2, '年龄', style1)
sh.write(1, 1, '张三')
sh.write(1, 2, 50, style2)
sh.write(2, 1, '李四')
sh.write(2, 2, 30, style2)
sh.write(3, 1, '王五')
sh.write(3, 2, 40, style2)
sh.write(4, 1, '赵六')
sh.write(4, 2, 60, style2)
sh.write(5, 0, '平均年龄', style1)
# 保存
wb.save('test.xls')
```

### 使用 XlsxWriter

通过 `pip install XlsxWriter` 命令安装

```py
import xlsxwriter

# 创建工作簿
workbook = xlsxwriter.Workbook('test.xlsx')
# 创建表单
sh = workbook.add_worksheet('test')
fmt1 = workbook.add_format()
fmt2 = workbook.add_format()
# 字体加粗
fmt1.set_bold(True)
# 设置左对齐
fmt2.set_align('left')
# 数据
data = [
    ['', '姓名', '年龄'],
    ['', '张三', 50],
    ['', '李四', 30],
    ['', '王五', 40],
    ['', '赵六', 60],
    ['平均年龄', '', ]
]
sh.write_row('A1', data[0], fmt1)
sh.write_row('A2', data[1], fmt2)
sh.write_row('A3', data[2], fmt2)
sh.write_row('A4', data[3], fmt2)
sh.write_row('A5', data[4], fmt2)
sh.write_row('A6', data[5], fmt1)
chart = workbook.add_chart({'type': 'line'})
workbook.close()
```

XlsxWriter生成图表

```py
import xlsxwriter

# 创建工作簿
wk = xlsxwriter.Workbook('test.xlsx')
# 创建表单
sh = wk.add_worksheet('test')
fmt1 = wk.add_format()
fmt2 = wk.add_format()
# 字体加粗
fmt1.set_bold(True)
# 设置左对齐
fmt2.set_align('left')
# 数据
data = [
    ['', '姓名', '年龄'],
    ['', '张三', 50],
    ['', '李四', 30],
    ['', '王五', 40],
    ['', '赵六', 60],
    ['平均年龄', '', ]
]
sh.write_row('A1', data[0], fmt1)
sh.write_row('A2', data[1], fmt2)
sh.write_row('A3', data[2], fmt2)
sh.write_row('A4', data[3], fmt2)
sh.write_row('A5', data[4], fmt2)
sh.write_row('A6', data[5], fmt1)
'''
area：面积图
bar：直方图
column：柱状图
line：折线图
pie：饼图
doughnut：环形图
radar：雷达图
'''
chart = wk.add_chart({'type': 'line'})
# 创建图表
chart.add_series(
    {
        'name': '=test!$B$1',
        'categories': '=test!$B$2:$B$5',
        'values':   '=test!$C$2:$C$5'
    }
)
chart.set_title({'name': '用户年龄折线图'})
chart.set_x_axis({'name': '姓名'})
chart.set_y_axis({'name': '年龄'})
sh.insert_chart('A9', chart)
wk.close()
```

## 读取

使用 `xlrd` 读取数据，使用 `pip install xlrd` 命令安装。

```py
import xlrd

# 打开
wb = xlrd.open_workbook('test.xls')
print('sheet名称:', wb.sheet_names())
print('sheet数量:', wb.nsheets)
# 根据 sheet 索引获取 sheet
sh = wb.sheet_by_index(0)
# 根据 sheet 名称获取 sheet
# sh = wb.sheet_by_name('test')
print(u'sheet %s 有 %d 行' % (sh.name, sh.nrows))
print(u'sheet %s 有 %d 列' % (sh.name, sh.ncols))
print('第二行内容:', sh.row_values(1))
print('第三列内容:', sh.col_values(2))
print('第二行第三列的值为:', sh.cell_value(1, 2))
print('第二行第三列值的类型为:', type(sh.cell_value(1, 2)))
```

## 修改

使用 `xlutils` 模块，使用 `pip install xlutils` 安装。

```py
import xlrd
import xlwt
from xlutils.copy import copy

def avg(list):
    sumv = 0
    for i in range(len(list)):
        sumv += list[i]
    return int(sumv / len(list))

# formatting_info 为 True 表示保留原格式
wb = xlrd.open_workbook('test.xls', formatting_info=True)
# 复制
wbc = copy(wb)
sh = wb.sheet_by_index(0)
age_list = sh.col_values(2)
age_list = age_list[1:len(age_list)-1]
avg_age = avg(age_list)
sh = wbc.get_sheet(0)
# 设置左对齐
alm = xlwt.Alignment()
alm.horz = 0x01
style = xlwt.XFStyle()
style.alignment = alm
sh.write(5, 2, avg_age, style)
wbc.save('test.xls')
```

参考：

[Python-Excel 模块哪家强？](https://zhuanlan.zhihu.com/p/23998083)

[Python 进阶（六）： Excel 基本操作](https://ityard.blog.csdn.net/article/details/104454512)