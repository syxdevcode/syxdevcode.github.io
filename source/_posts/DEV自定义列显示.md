---
title: DEV自定义列显示
date: 2021-08-23 18:01:18
tags:
- DotNet
- CSharp
- GridView
- DevExpress
categories: 
- DevExpress
---

![微信截图_20210823180630.png](/img/微信截图_20210823180630.png)

![微信截图_20210823180722.png](/img/微信截图_20210823180722.png)

```cs
/// <summary>
/// 自定义 列名显示
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void dgvGroup_CustomColumnDisplayText(object sender, DevExpress.XtraGrid.Views.Base.CustomColumnDisplayTextEventArgs e)
{
    if (e.Column.FieldName == "gender" && e.ListSourceRowIndex != DevExpress.XtraGrid.GridControl.InvalidRowHandle)
    {
        if (e.Value.ToString() == "0")
            e.DisplayText = "男";
        else if (e.Value.ToString() == "1")
            e.DisplayText = "女";
        else
            e.DisplayText = "未知";
    }
}
```