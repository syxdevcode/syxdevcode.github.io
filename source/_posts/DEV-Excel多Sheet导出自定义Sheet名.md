---
title: DEV-Excel多Sheet导出自定义Sheet名
date: 2021-08-31 18:43:32
tags:
- DotNet
- CSharp
- GridView
- DevExpress
categories: 
- DevExpress
---

调用方法：

```cs
/// <summary>
/// 导出
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void btnOutPut_Click(object sender, EventArgs e)
{
    try
    {
        ExportToExcel("候选人信息", true, "", PrintingSystem_XlSheetCreated, gctlGroup, gctlGroup1, gctlGroup2, gctlGroup3);
    }
    catch (Exception ex)
    {
        LoadingHelper.CloseForm();
        XtraMessageBox.Show("获取异常：" + ex.ToString());
    }
    finally { LoadingHelper.CloseForm(); }
}
```

<details>
<summary>方法定义：</summary>

**<summary>ExportToExcel方法定义：**
```cs
/// <summary>
/// </summary>
/// DevExpress控件通用导出Excel,支持多个控件同时导出在同一个Sheet表或者分不同工作薄
/// eg:ExportToXlsx("test",true,"控件",gridControl1,gridControl2);
/// 将gridControl1和gridControl2的数据一同导出到同一个文件不同的工作薄
/// eg:ExportToXlsx("test",false,"",gridControl1,gridControl2);
/// 将gridControl1和gridControl2的数据一同导出到同一个文件同一个的工作薄
/// <param name="title">文件名</param>
/// <param name="isPageForEachLink">多个打印控件是否分多个工作薄显示</param>
/// <param name="sheetName">工作薄名称</param>
/// <param name="handler">修改sheet事件名</param>
/// <param name="printables">控件集 eg:GridControl,PivotGridControl,TreeList,ChartControl...</param>
public static void ExportToExcel(string title, bool isPageForEachLink, string sheetName, XlSheetCreatedEventHandler handler, params DevExpress.XtraPrinting.IPrintable[] printables)
{
    SaveFileDialog saveFileDialog = new SaveFileDialog() { FileName = title, Title = "导出Excel", Filter = "Excel文件(*.xlsx)|*.xlsx|Excel文件(*.xls)|*.xls" };
    DialogResult dialogResult = saveFileDialog.ShowDialog();
    if (dialogResult == DialogResult.Cancel) return;
    string fileName = saveFileDialog.FileName;
    DevExpress.XtraPrintingLinks.CompositeLink link = new DevExpress.XtraPrintingLinks.CompositeLink(new DevExpress.XtraPrinting.PrintingSystem());
    foreach (var item in printables)
    {
        var plink = new DevExpress.XtraPrinting.PrintableComponentLink() { Component = item };
        link.Links.Add(plink);
    }

    if (handler != null)
    {
        // sheet 重命名
        link.PrintingSystem.XlSheetCreated += handler;
    }

    if (isPageForEachLink)
    {
        //用于每个Link生成一个Sheet,不使用此方法，则合并在一个Sheet内
        link.CreatePageForEachLink();
    }
    if (string.IsNullOrEmpty(sheetName))
        sheetName = "Sheet";//默认工作薄名称
    try
    {
        int count = 1;
        //在重复名称后加（序号）
        while (System.IO.File.Exists(fileName))
        {
            if (fileName.Contains(")."))
            {
                int start = fileName.LastIndexOf("(", StringComparison.Ordinal);
                int end = fileName.LastIndexOf(").", StringComparison.Ordinal) - fileName.LastIndexOf("(", StringComparison.Ordinal) + 2;
                fileName = fileName.Replace(fileName.Substring(start, end), $"({count}).");
            }
            else
            {
                fileName = fileName.Replace(".", $"({count}).");
            }
            count++;
        }

        if (fileName.LastIndexOf(".xlsx", StringComparison.Ordinal) >= fileName.Length - 5)
        {
            DevExpress.XtraPrinting.XlsxExportOptions options = new DevExpress.XtraPrinting.XlsxExportOptions()
            { SheetName = sheetName };
            if (isPageForEachLink)
                options.ExportMode = DevExpress.XtraPrinting.XlsxExportMode.SingleFilePageByPage;
            link.ExportToXlsx(fileName, options);
        }
        else
        {
            DevExpress.XtraPrinting.XlsExportOptions options = new DevExpress.XtraPrinting.XlsExportOptions()
            { SheetName = sheetName };
            if (isPageForEachLink) //15.Xls没有这个属性，15.2未知
                options.ExportMode = DevExpress.XtraPrinting.XlsExportMode.SingleFilePageByPage;
            link.ExportToXls(fileName, options);
        }

        if (DevExpress.XtraEditors.XtraMessageBox.Show("保存成功，是否打开文件？", "提示", MessageBoxButtons.YesNo,
            MessageBoxIcon.Information) == DialogResult.Yes)
            System.Diagnostics.Process.Start(fileName); //打开指定路径下的文件
    }
    catch (Exception ex)
    {
        DevExpress.XtraEditors.XtraMessageBox.Show(ex.Message);
    }
}

private static void PrintingSystem_XlSheetCreated(object sender, XlSheetCreatedEventArgs e)
{
    if (e.Index == 0)
        e.SheetName = "基本信息";
    else if (e.Index == 1)
        e.SheetName = "学业信息";
    else if (e.Index == 2)
        e.SheetName = "工作经历";
    else if (e.Index == 3)
        e.SheetName = "家庭成员";
}
```
</details>

参考：

[How to set the sheet names for links](https://supportcenter.devexpress.com/ticket/details/t551319/how-to-set-the-sheet-names-for-links#)