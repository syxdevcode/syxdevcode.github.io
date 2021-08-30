---
title: Aspose.Words生成Word文档
date: 2021-08-30 15:25:46
tags:
- Aspose
- Windows
- Word
categories:
- Aspose
---

word表格设置如下：

注意，表格需要设置 开始结束标记：`TableStart`，`TableEnd`:

```cs
TableStart:GZ // 开始，GZ为表名

T_Row_QZSJ
T_Row_ZW
T_Row_GZDW
T_Row_LZYY
T_Row_ZMR

TableEnd:GZ // 结束，GZ为表名
```

word设置表格如下：

![微信截图_20210830154308.png](/img/微信截图_20210830154308.png)

![微信截图_20210830153642.png](/img/微信截图_20210830153642.png)

<details>
<summary>调用方法：</summary>

**<summary>Dev开发界面方法**
```cs
private Dictionary<string, string> familyMembersDic = new Dictionary<string, string>(8);

private void GenFamilyMembers()
{
    if (familyMembersDic.Keys.Count == 8) return;

    familyMembersDic.Add("父", "");
    familyMembersDic.Add("母", "");
    familyMembersDic.Add("配偶", "");
    familyMembersDic.Add("子女", "");
    familyMembersDic.Add("(配偶)父", "");
    familyMembersDic.Add("(配偶)母", "");
    familyMembersDic.Add("兄弟", "");
    familyMembersDic.Add("姐妹", "");
}

/// <summary>
/// 导出人员情况登记表
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void simpleButton1_Click(object sender, EventArgs e)
{
    try
    {
        FolderBrowserDialog folderDialog = new FolderBrowserDialog();
        folderDialog.Description = "选择路径";
        DialogResult dialogResult = folderDialog.ShowDialog();
        if (dialogResult == DialogResult.Cancel) return;
        string path = folderDialog.SelectedPath;

        var templateFile = Environment.CurrentDirectory + @"\Template\WordsTemp\01人员情况登记表.doc";

        GenFamilyMembers();

        //打开流转单模板
        WordHelper wh = new WordHelper(templateFile);

        bool isGen = false;
        for (int i = 0; i < this.dgvGroup.RowCount; i++)
        {
            if (this.dgvGroup.GetRow(i) is TB_PLY_OPT_EHR_ResumeInfoDtoView row )
            {
                var t_time = $"{row.createDateTime:yyyy-MM-dd}";
                wh.ReplaceText("T_name", row.name);
                wh.ReplaceText("T_jobtitle", row.jobtitle);
                wh.ReplaceText("T_date", t_time);
                wh.ReplaceText("T_gender", row.gender);
                wh.ReplaceText("T_idcard", row.idcard);
                wh.ReplaceText("T_birth", row.birth);
                wh.ReplaceText("T_mz", row.national);
                wh.ReplaceText("T_nativeplace", row.nativeplace);
                wh.ReplaceText("T_hkszd", row.censusregisterseat);
                wh.ReplaceText("T_sg", row.uheight.ToString());
                wh.ReplaceText("T_married", row.married);
                wh.ReplaceText("T_health", row.health);
                wh.ReplaceText("T_zy", row.lastSpeciality);
                wh.ReplaceText("T_school", row.school);
                wh.ReplaceText("T_bysj", row.endDate);
                wh.ReplaceText("T_xl", row.academicDegree);
                wh.ReplaceText("T_phone", row.phone);
                wh.ReplaceText("T_jtdh", row.homephone);
                wh.ReplaceText("T_zc", row.thetitle);
                wh.ReplaceText("T_currentaddress", row.currentaddress);

                DataTable dtXM = new DataTable("Family");
                dtXM.Columns.Add("T_Row_Gx", typeof(string));
                dtXM.Columns.Add("T_Row_Xm", typeof(string));
                dtXM.Columns.Add("T_Row_IdCard", typeof(string));
                dtXM.Columns.Add("T_Row_Work", typeof(string));
                dtXM.Columns.Add("T_Row_Phone", typeof(string));
                if (row.家庭成员 != null && row.家庭成员.Any())
                {
                    foreach (var dic in familyMembersDic)
                    {
                        var item = row.家庭成员.FirstOrDefault(o => o.与应聘者关系 == dic.Key);

                        DataRow dr = dtXM.NewRow();
                        if (item == null)
                        {
                            dr["T_Row_Gx"] = dic.Key;
                            dr["T_Row_Xm"] = "";
                            dr["T_Row_IdCard"] = "";
                            dr["T_Row_Work"] = "";
                            dr["T_Row_Phone"] = "";
                        }
                        else
                        {
                            dr["T_Row_Gx"] = item.与应聘者关系;
                            dr["T_Row_Xm"] = item.姓名;
                            dr["T_Row_IdCard"] = item.身份证号;
                            dr["T_Row_Work"] = item.工作单位及职务;
                            dr["T_Row_Phone"] = item.联系电话;
                        }
                        dtXM.Rows.Add(dr);
                    }
                }
                wh.doc.MailMerge.ExecuteWithRegions(dtXM);

                DataTable dtEdu = new DataTable("Edu");
                dtEdu.Columns.Add("T_Row_QZSJ", typeof(string));
                dtEdu.Columns.Add("T_Row_School", typeof(string));
                dtEdu.Columns.Add("T_Row_ZY", typeof(string));
                if (row.学业经历 != null && row.学业经历.Any())
                {
                    foreach (var item in row.学业经历)
                    {
                        DataRow dr = dtEdu.NewRow();

                        dr["T_Row_QZSJ"] = item.起止年月;
                        dr["T_Row_School"] = item.学校名称;
                        dr["T_Row_ZY"] = item.所学专业;

                        dtEdu.Rows.Add(dr);
                    }
                }
                wh.doc.MailMerge.ExecuteWithRegions(dtEdu);

                DataTable dtGZ = new DataTable("GZ");
                dtGZ.Columns.Add("T_Row_QZSJ", typeof(string));
                dtGZ.Columns.Add("T_Row_ZW", typeof(string));
                dtGZ.Columns.Add("T_Row_GZDW", typeof(string));
                dtGZ.Columns.Add("T_Row_LZYY", typeof(string));
                dtGZ.Columns.Add("T_Row_ZMR", typeof(string));
                if (row.工作经历 != null && row.工作经历.Any())
                {
                    foreach (var item in row.工作经历)
                    {
                        DataRow dr = dtGZ.NewRow();

                        dr["T_Row_QZSJ"] = item.起止年月;
                        dr["T_Row_ZW"] = item.职务;
                        dr["T_Row_GZDW"] = item.工作单位;
                        dr["T_Row_LZYY"] = item.离职原因;
                        dr["T_Row_ZMR"] = item.证明人及其联系电话;

                        dtGZ.Rows.Add(dr);
                    }
                }
                wh.doc.MailMerge.ExecuteWithRegions(dtGZ);

                wh.SaveWord($@"{path}\\{row.name}_{DateTime.Now:yyyyMMddhhmmss}.doc");
                isGen = true;
            }
        }
        if (isGen)
        {
            if (DevExpress.XtraEditors.XtraMessageBox.Show("保存成功，是否打开文件夹？", "提示", MessageBoxButtons.YesNo,
                MessageBoxIcon.Information) == DialogResult.Yes)
                System.Diagnostics.Process.Start("explorer.exe", path); //打开指定路径
        }
        else
        {
            DevExpress.XtraEditors.XtraMessageBox.Show("未能生成有效文档");
        }
    }
    catch (Exception ex)
    {
        DevExpress.XtraEditors.XtraMessageBox.Show(ex.Message);
    }
}
```
</details>

<details>
<summary>AsposeWordHelper帮助类</summary>

```cs
using Aspose.Pdf.Facades;
using Aspose.Pdf.InteractiveFeatures.Forms;
using Aspose.Words;
using Aspose.Words.Drawing;
using Aspose.Words.Saving;
using Aspose.Words.Tables;
using System;
using System.Collections;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Net;
using System.Text.RegularExpressions;

namespace Utils
{
    public class AsposeWordHelper
    {
        /// <summary>
        /// 模板
        /// </summary>
        public string templateFile { get; set; }

        public Document doc = null;
        public DocumentBuilder builder = null;

        #region 实例化模板

        public WordHelper(string templateFile)
        {
            doc = new Document(templateFile); //载入模板
            builder = new DocumentBuilder(doc);
            //doc.Protect(ProtectionType.ReadOnly); //设为只读
        }

        #endregion 实例化模板

        /// <summary>
        /// 替换书签内容，传入书签名称和要替换的值即可
        /// </summary>
        public void ReplaceBookmarks(string BookmarksName, string ReplaceBookmarksText)
        {
            if (doc.Range.Bookmarks[BookmarksName] != null)
            {
                Aspose.Words.Bookmark mark = doc.Range.Bookmarks[BookmarksName];
                mark.Text = ReplaceBookmarksText;
            }
        }

        /// <summary>
        /// 替换书签图片，传入书签名称和要替换的值及宽和高即可,会判断远程文件是否存在，适用于报告生成。
        /// </summary>
        public void ReplaceBookmarksPic(string BookmarksName, string ReplaceBookmarksPic, int picWidth, int picHeight)
        {
            if (RemoteFileExists(ReplaceBookmarksPic))
            {
                Shape shape = new Shape(doc, ShapeType.Image);
                shape.ImageData.SetImage(ReplaceBookmarksPic);
                shape.Width = picWidth;
                shape.Height = picHeight;

                builder.MoveToBookmark(BookmarksName);
                builder.InsertNode(shape);
            }
        }

        /// <summary>
        /// 替换书签图片，传入书签名称和要替换的值及宽和高即可，不用判断文件是否存在！
        /// </summary>
        public void ReplaceBookmarksPic1(string BookmarksName, string ReplaceBookmarksPic, int picWidth, int picHeight)
        {
            Shape shape = new Shape(doc, ShapeType.Image);
            shape.ImageData.SetImage(ReplaceBookmarksPic);
            shape.Width = picWidth;
            shape.Height = picHeight;
            builder.MoveToBookmark(BookmarksName);
            builder.InsertNode(shape);
        }

        public void ReplaceBookmarksPic1(string BookmarksName, byte[] imageData, int picWidth, int picHeight, double picLeft)
        {
            Shape shape = new Shape(doc, ShapeType.Image);
            shape.ImageData.SetImage(imageData);
            shape.Width = picWidth;
            shape.Height = picHeight;
            shape.Left = picLeft;
            builder.MoveToBookmark(BookmarksName);
            builder.InsertNode(shape);
        }

        public void ReplaceBookmarksPic1(string BookmarksName, string ReplaceBookmarksPic, int picWidth, int picHeight, double picLeft)
        {
            Shape shape = new Shape(doc, ShapeType.Image);
            shape.ImageData.SetImage(ReplaceBookmarksPic);
            shape.Width = picWidth;
            shape.Height = picHeight;
            shape.Left = picLeft;

            builder.MoveToBookmark(BookmarksName);
            builder.InsertNode(shape);
        }

        /// <summary>
        /// 插入图片，传入书签名称和要替换的值及左侧位置及宽和高即可，不用判断文件是否存在。未测试！！！！
        /// </summary>
        public void ReplaceBookmarksPic2(string BookmarksName, string ReplaceBookmarksPic, double picLeft, double picWidth, double picHeight)
        {
            if (RemoteFileExists(ReplaceBookmarksPic))
            {
                builder.MoveToBookmark(BookmarksName);
                builder.InsertImage(ReplaceBookmarksPic, RelativeHorizontalPosition.Page, picLeft, RelativeVerticalPosition.TopMargin, 0, picWidth, picHeight, WrapType.None);
            }
        }

        /// <summary>
        /// 插入图片(本地图片)，传入书签名称和要替换的值及左侧位置及宽和高即可，不用判断文件是否存在。未测试！！！！
        /// </summary>
        public void ReplaceBookmarksPic3(string BookmarksName, string ReplaceBookmarksPic, double picLeft, double picWidth, double picHeight)
        {
            if (LocalFileExists(ReplaceBookmarksPic))
            {
                builder.MoveToBookmark(BookmarksName);
                builder.InsertImage(ReplaceBookmarksPic, RelativeHorizontalPosition.Page, picLeft, RelativeVerticalPosition.TopMargin, 0, picWidth, picHeight, WrapType.None);
            }
        }

        /// <summary>
        /// 本地文件是否存在验证
        /// </summary>
        /// <param name="filePath"></param>
        /// <returns></returns>
        public bool LocalFileExists(string filePath)
        {
            try
            {
                if (File.Exists(filePath))
                {
                    return true;
                }
                else
                {
                    return false;
                }
            }
            catch (Exception)
            {
                return false;
            }
        }

        /// <summary>
        /// 查找替换
        /// </summary>
        public void ReplaceText(string SearchText, string ReplaceText)
        {
            ReplaceText = ReplaceText.Replace("\n", "");
            ReplaceText = ReplaceText.Replace("\r", "");
            doc.Range.Replace(SearchText, ReplaceText, true, true);
        }

        /// <summary>
        /// 设置页眉页脚，考虑内容和格式是否可以分开处理
        /// </summary>

        public void SetHeaderFooter1(bool headflag, string HeaderFooterText)
        {
            //设置移动到页面最底下

            builder.MoveToDocumentEnd();

            //设置页眉高度

            Section currentSection = builder.CurrentSection;
            Aspose.Words.PageSetup pageSetup = currentSection.PageSetup;
            pageSetup.HeaderDistance = 50;

            //移动光标至页眉页脚，设置属性

            builder.MoveToHeaderFooter(HeaderFooterType.HeaderPrimary);
            //builder.MoveToHeaderFooter(HeaderFooterType.HeaderFirst);
            builder.Underline = Underline.Single;
            //  builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;
            // Set font properties for header text.
            builder.Font.Name = "宋体";
            //  builder.RowFormat.Alignment = RowAlignment.Center;
            builder.Font.Bold = true;
            builder.Font.Size = 11;
            // Specify header title for the first page.
            if (headflag)
            {
                builder.Write(HeaderFooterText);
            }
        }

        /// <summary>
        /// 设置页眉页脚
        /// </summary>
        /// <param name="headflag"></param>
        /// <param name="HeaderFooterText"></param>
        public void SetHeaderFooter(bool headflag, string HeaderFooterText)
        {
            //设置页眉高度
            Section currentSection = builder.CurrentSection;
            Aspose.Words.PageSetup pageSetup = currentSection.PageSetup;
            pageSetup.HeaderDistance = 25;

            if (headflag)
            {
                builder.Write(HeaderFooterText);
            }
        }

        /// <summary>
        /// 设置表格
        /// </summary>
        /// <param name="headflag"></param>
        /// <param name="HeaderFooterText"></param>
        public void SetTable(bool headflag, string HeaderFooterText)
        {
            builder.MoveToDocumentEnd();
            builder.StartTable();

            builder.RowFormat.Alignment = RowAlignment.Center;
            builder.CellFormat.Borders.LineStyle = LineStyle.Single;
            builder.CellFormat.Borders.Color = Color.Black;
            //builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            //builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;
            builder.Bold = true;

            builder.InsertCell();
            builder.CellFormat.Width = 80;
            builder.Write("序号");
            //builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            //builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;

            builder.InsertCell();
            builder.CellFormat.Width = 250;
            builder.Write("产品名称");
            //builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            //builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;

            builder.InsertCell();
            builder.CellFormat.Width = 110;
            builder.Write("品牌");
            //builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            //builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;

            builder.InsertCell();
            builder.CellFormat.Width = 250;
            builder.Write("型号");
            //builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            //builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;

            builder.InsertCell();
            builder.CellFormat.Width = 170;
            builder.Write("备注");
            //builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            //builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;

            builder.EndRow();
        }

        /// <summary>
        /// 设置表格信息
        /// </summary>
        /// <param name="CellWidth"></param> 单元格宽度
        /// <param name="CellTitle"></param> 单元格标题
        public void InsertTableCell(int CellWidth, string CellTitle)
        {
            builder.InsertCell();
            builder.CellFormat.Width = CellWidth;
            builder.Write(CellTitle);
            builder.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
            builder.ParagraphFormat.Alignment = ParagraphAlignment.Center;
        }

        /// <summary>
        /// 获取表格信息
        /// </summary>
        /// <param name=""></param> 单元格宽度
        /// <param name=""></param> 单元格标题
        public void GetTableCell(int TableIndex, int RowsIndex, int ColumnIndex, int CharacterIndex, string CellText)
        {
            NodeCollection allTables = doc.GetChildNodes(NodeType.Table, true);
            Table table = allTables[3] as Aspose.Words.Tables.Table;//拿到第一个表格
            builder.MoveToCell(0, 2, 1, 0);//先跳转到第0个表格，第31行，第1列
            //builder.MoveToCell(TableIndex, RowsIndex, ColumnIndex, CharacterIndex);//先跳转到第0个表格，第31行，第1列
            builder.Write(CellText);
        }

        public void SetCellLocation(int count)
        {
            try
            {
                NodeCollection allTables = doc.GetChildNodes(NodeType.Table, true);
                //拿到第一个表格
                int rowIndex = 5;
                Table table = null;
                for (int i = 0; i < allTables.Count; i++)
                {
                    bool res = false;
                    if (!res)
                    {
                        table = allTables[i] as Aspose.Words.Tables.Table;
                        for (int z = 0; z < table.Rows.Count; z++)
                        {
                            string tempRow = table.Rows[z].Range.Text.Replace("\a", "");
                            if (tempRow == "")
                            {
                                //beforeRow = table.Rows[z];//正确配置，待复制的空白行
                                rowIndex = z;
                                res = true;
                                break;
                            }
                        }
                    }
                }
                if (count > 0 && table != null)
                {
                    int colCount = table.Rows[rowIndex + count - 1].Cells.Count;
                    if (colCount > 0)
                    {
                        for (int n = 0; n < colCount; n++)
                        {
                            Cell cell = table.Rows[rowIndex + count - 1].Cells[n];

                            cell.CellFormat.TopPadding = 50;
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                return;
            }
        }

        public void SetCellBackColor(int count)
        {
            try
            {
                NodeCollection allTables = doc.GetChildNodes(NodeType.Table, true);
                //拿到第一个表格
                int rowIndex = 5;
                Table table = null;
                for (int i = 0; i < allTables.Count; i++)
                {
                    bool res = false;
                    if (!res)
                    {
                        table = allTables[i] as Aspose.Words.Tables.Table;
                        for (int z = 0; z < table.Rows.Count; z++)
                        {
                            string tempRow = table.Rows[z].Range.Text.Replace("\a", "");
                            if (tempRow == "")
                            {
                                //beforeRow = table.Rows[z];//正确配置，待复制的空白行
                                rowIndex = z;
                                res = true;
                                break;
                            }
                        }
                    }
                }
                if (count > 0 && table != null)
                {
                    for (int m = 1; m <= count; m++)
                    {
                        if (m % 2 != 0)
                        {
                            int colCount = table.Rows[rowIndex + m].Cells.Count;
                            if (colCount > 0)
                            {
                                for (int n = 0; n < colCount; n++)
                                {
                                    Cell cell = table.Rows[rowIndex + m].Cells[n];
                                    cell.CellFormat.Shading.BackgroundPatternColor = Color.FromArgb(244, 247, 251);
                                }
                            }
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                return;
            }
        }

        public void SetCellMerge(int count)
        {
            try
            {
                NodeCollection allTables = doc.GetChildNodes(NodeType.Table, true);
                Table table = allTables[0] as Aspose.Words.Tables.Table;//拿到第一个表格
                int rowIndex = 5;
                if (table != null && table.Rows.Count > 0)
                {
                    for (int z = 0; z < table.Rows.Count; z++)
                    {
                        string tempRow = table.Rows[z].Range.Text.Replace("\a", "");
                        if (tempRow == "")
                        {
                            //beforeRow = table.Rows[z];//正确配置，待复制的空白行
                            rowIndex = z;
                            break;
                        }
                    }
                }

                if (count >= 2)
                {
                    Cell cell0 = table.Rows[rowIndex + 1].Cells[0];
                    Cell cell01 = table.Rows[rowIndex + 2].Cells[0];
                    //单元格内容
                    string cellContent0 = cell0.GetText().Replace("\a", "");
                    string cellContent01 = cell01.GetText().Replace("\a", "");
                    if (cellContent0 == cellContent01)
                    {
                        cell0.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                        cell01.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                        if (count == 3)
                        {
                            Cell cell02 = table.Rows[rowIndex + 3].Cells[0];
                            string cellContent02 = cell02.GetText().Replace("\a", "");
                            if (cellContent01 == cellContent02)
                            {
                                cell02.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                            }
                        }
                    }

                    Cell cell1 = table.Rows[rowIndex + 1].Cells[1];
                    Cell cell11 = table.Rows[rowIndex + 2].Cells[1];
                    string cellContent1 = cell1.GetText().Replace("\a", "");
                    string cellContent11 = cell11.GetText().Replace("\a", "");
                    if (cellContent1 == cellContent11)
                    {
                        cell1.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                        cell11.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                        if (count == 3)
                        {
                            Cell cell12 = table.Rows[rowIndex + 3].Cells[1];
                            string cellContent12 = cell12.GetText().Replace("\a", "");
                            if (cellContent11 == cellContent12)
                            {
                                cell12.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                            }
                        }
                    }

                    Cell cell40 = table.Rows[rowIndex - 1].Cells[4];
                    Cell cell4 = table.Rows[rowIndex + 1].Cells[4];
                    Cell cell41 = table.Rows[rowIndex + 2].Cells[4];
                    string cellContent40 = cell40.GetText().Replace("\a", "");
                    string cellContent4 = cell4.GetText().Replace("\a", "");
                    string cellContent41 = cell41.GetText().Replace("\a", "");
                    if (cellContent4 == cellContent41 && cellContent40.Contains("检出限"))
                    {
                        cell4.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                        cell41.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                        if (count == 3)
                        {
                            Cell cell42 = table.Rows[rowIndex + 3].Cells[4];
                            string cellContent42 = cell42.GetText().Replace("\a", "");
                            if (cellContent41 == cellContent42)
                            {
                                cell42.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                            }
                        }
                    }
                }
            }
            catch (Exception)
            {
                return;
            }
        }

        /// <summary>
        /// 修改表中的数据
        /// </summary>
        /// <param name="table">表名</param>
        /// <param name="doc">文档</param>
        /// <param name="row">要修改行</param>
        /// <param name="cell">要修改列</param>
        /// <param name="value">修改后的值</param>
        public Table EditCell(Table table, Document doc, int row, int cell, string value)
        {
            try
            {
                //st = cy.FirstParagraph.ParagraphFormat.Style.Font;
                Cell c = table.Rows[row].Cells[cell];
                Paragraph p = new Paragraph(doc);
                p.AppendChild(new Run(doc, value));
                p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                //p.ParagraphFormat.Style.Font.Size = 12;
                c.FirstParagraph.Remove();
                c.AppendChild(p);
                table.Rows[row].Cells[cell].Remove();
                table.Rows[row].Cells.Insert(cell, c);
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return table;
        }

        /// <summary>
        /// 修改表中的数据（带字体大小，获取列头的字体）
        /// </summary>
        /// <param name="table">表名</param>
        /// <param name="doc">文档</param>
        /// <param name="row">要修改行</param>
        /// <param name="cell">要修改列</param>
        /// <param name="value">修改后的值</param
        /// <param name="ft">替换的字体</param>
        public Table EditCell(Table table, Document doc, int row, int cell, string value, Aspose.Words.Font ft)
        {
            try
            {
                Cell c = table.Rows[row].Cells[cell];
                Paragraph p = new Paragraph(doc);
                p.AppendChild(new Run(doc, value));
                p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                p.ParagraphFormat.Style.Font.Size = ft.Size;
                c.FirstParagraph.Remove();
                c.AppendChild(p);
                table.Rows[row].Cells[cell].Remove();
                table.Rows[row].Cells.Insert(cell, c);
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return table;
        }

        //private string oldValue = "";

        /// <summary>
        /// 修改表中的数据（带字体大小，获取列头的字体）
        /// </summary>
        /// <param name="table">表名</param>
        /// <param name="doc">文档</param>
        /// <param name="row">要修改行</param>
        /// <param name="cell">要修改列</param>
        /// <param name="value">修改后的值</param
        /// <param name="ft">替换的字体</param>
        public Table EditCell(Table table, Document doc, int row, int cell, string value, Aspose.Words.Font ft, int x, ref string oldValue)
        {
            try
            {
                Cell c = table.Rows[row].Cells[cell];
                Paragraph p = new Paragraph(doc);
                Run r = null;

                if (x == 0)
                {
                    if (cell == 0)
                    {
                        oldValue = value;
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        r = new Run(doc, oldValue);
                        r.Font.Size = ft.Size;
                        //p.AppendChild(new Run(doc, oldValue));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft.Size;
                        p.AppendChild(r);
                    }
                    else
                    {
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        r = new Run(doc, value);
                        r.Font.Size = ft.Size;
                        //p.AppendChild(new Run(doc, value));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft.Size;
                        p.AppendChild(r);
                    }
                }
                else
                {
                    if (cell == 0)
                    {
                        if (oldValue == value)
                        {
                            c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                            c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        }
                        else
                        {
                            oldValue = value;
                            c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                            c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                            //p.AppendChild(new Run(doc, oldValue));
                            r = new Run(doc, oldValue);
                            r.Font.Size = ft.Size;
                            p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                            //p.ParagraphFormat.Style.Font.Size = ft.Size;
                            p.AppendChild(r);
                        }
                    }
                    else
                    {
                        //oldValue = value;
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        //p.AppendChild(new Run(doc, value));
                        r = new Run(doc, value);
                        r.Font.Size = ft.Size;
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft.Size;
                        p.AppendChild(r);
                    }
                }

                //if (x == 0)
                //{
                //    p.AppendChild(new Run(doc, value));
                //    p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                //    if (cell == 4|| cell == 0)
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                //        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                //    }
                //    else
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                //    }
                //}
                //else
                //{
                //    if (cell == 4 || cell == 0)
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                //        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                //    }
                //    else
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                //    }
                //}
                c.FirstParagraph.Remove();
                c.AppendChild(p);
                table.Rows[row].Cells[cell].Remove();
                table.Rows[row].Cells.Insert(cell, c);
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return table;
        }

        public Table EditCell(Table table, Document doc, int row, int cell, string value, double ft, int x, ref string oldValue)
        {
            try
            {
                Cell c = table.Rows[row].Cells[cell];
                Paragraph p = new Paragraph(doc);

                if (x == 0)
                {
                    if (cell == 0)
                    {
                        oldValue = value;
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        p.AppendChild(new Run(doc, oldValue));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft;
                    }
                    else
                    {
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        p.AppendChild(new Run(doc, value));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft;
                    }
                }
                else
                {
                    if (cell == 0)
                    {
                        if (oldValue == value)
                        {
                            c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                            c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        }
                        else
                        {
                            oldValue = value;
                            c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                            c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                            p.AppendChild(new Run(doc, oldValue));
                            p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                            //p.ParagraphFormat.Style.Font.Size = ft;
                        }
                    }
                    else
                    {
                        //oldValue = value;
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        p.AppendChild(new Run(doc, value));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft;
                    }
                }

                //if (x == 0)
                //{
                //    p.AppendChild(new Run(doc, value));
                //    p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                //    if (cell == 4|| cell == 0)
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                //        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                //    }
                //    else
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                //    }
                //}
                //else
                //{
                //    if (cell == 4 || cell == 0)
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                //        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                //    }
                //    else
                //    {
                //        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                //    }
                //}
                c.FirstParagraph.Remove();
                c.AppendChild(p);
                table.Rows[row].Cells[cell].Remove();
                table.Rows[row].Cells.Insert(cell, c);
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return table;
        }

        /// <summary>
        /// 处理上下标
        /// </summary>
        /// <param name="table"></param>
        /// <param name="doc"></param>
        /// <param name="row"></param>
        /// <param name="cell"></param>
        /// <param name="value"></param>
        /// <param name="ft"></param>
        /// <param name="x"></param>
        /// <param name="oldValue"></param>
        /// <returns></returns>
        public Table EditLimsCell(Table table, Document doc, int row, int cell, string value, Aspose.Words.Font ft, int x, ref string oldValue)
        {
            try
            {
                List<FontSubscript> listFont = new List<FontSubscript>();
                Cell c = table.Rows[row].Cells[cell];
                Paragraph p = new Paragraph(doc);
                Run r = null;

                if (x == 0)
                {
                    if (cell == 0)
                    {
                        oldValue = value;
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        listFont = GetFontInfoList(value);
                        //Dictionary<string, string> dicList = new Dictionary<string, string>();
                        //dicList.Add("{(.*?)}", "{{{0}}}");

                        //foreach (var dic in dicList)
                        //{
                        //    MatchCollection mc = Regex.Matches(oldValue, dic.Key);
                        //    if (mc != null && mc.Count > 0)
                        //    {
                        //        //List<FieldInfo> listF = new List<FieldInfo>();
                        //        //foreach (Match item in mc)
                        //        //{
                        //        //    if (!list.Exists(o => o.FieldName == item.Groups[1].Value))
                        //        //    {
                        //        //        fieldInfo = new FieldInfo();
                        //        //        fieldInfo.Type = "文本";
                        //        //        fieldInfo.TextReplaceStr = dic.Value;
                        //        //        fieldInfo.FieldName = item.Groups[1].Value;
                        //        //        list.Add(fieldInfo);
                        //        //    }
                        //        //}
                        //    }
                        //}

                        //foreach (KeyValuePair<string, string> kvp in dicList)
                        //{
                        //    Regex regex = new Regex(kvp.Value, RegexOptions.IgnoreCase);
                        //    wh.doc.Range.Replace(regex, new ReplaceEvaluatorFindAndHighlight(), false);
                        //}
                        r = new Run(doc, oldValue);
                        r.Font.Size = ft.Size;
                        //p.AppendChild(new Run(doc, oldValue));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft.Size;
                        p.AppendChild(r);

                        HandleFontSubscript(listFont, p);
                    }
                    else
                    {
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;

                        Dictionary<string, string> dicList = new Dictionary<string, string>();
                        dicList.Add("{(.*?)}", "{{{0}}}");
                        //dicList.Add("{(.)}", "{{{0}}}");
                        //dicList.Add("\\[(.*?)\\]", "\\[{0}\\]");
                        dicList.Add("\\[(.)\\]", "\\[{0}\\]");
                        //dicList.Add("<sub>(.*?)</sub>", "<sub>{0}</sub>");
                        //dicList.Add("<sup>(.*?)</sup>", "<sup>{0}</sup>");
                        //string[] arr = new string[] {"{2}", "{23}"};
                        //List<string> list = new List<string>();
                        listFont = GetFontInfoList(value);

                        #region 注释

                        //List<FontSubscript> listFont = new List<FontSubscript>();
                        //Dictionary<int, string> dicT = new Dictionary<int, string>();
                        //foreach (var dic in dicList)
                        //{
                        //    MatchCollection mc = Regex.Matches(value, dic.Key);
                        //    if (mc != null && mc.Count > 0)
                        //    {
                        //        FontSubscript ety;
                        //        for (int i = 0; i < mc.Count; i++)
                        //        {
                        //            ety = new FontSubscript();
                        //            if (mc[i].Value.Contains("]") && mc[i].Value.Contains("["))
                        //            {
                        //                ety.FontType = 1;//下标
                        //                                 //ety.ReplaceText = mc[i].Value.Replace("[", "");
                        //                ety.ReplaceText = "\\[" + mc[i].Groups[1].Value + "\\]";
                        //                ety.ReplaceValue = mc[i].Groups[1].Value;
                        //                listFont.Add(ety);
                        //            }
                        //            if (mc[i].Value.Contains("}") && mc[i].Value.Contains("{"))
                        //            {
                        //                ety.FontType = 0;//上标
                        //                                 //ety.ReplaceText = mc[i].Value.Replace("}", "");
                        //                ety.ReplaceText = "\\{" + mc[i].Groups[1].Value + "\\}";
                        //                ety.ReplaceValue = mc[i].Groups[1].Value;
                        //                listFont.Add(ety);
                        //            }
                        //        }
                        //    }
                        //}

                        #endregion 注释

                        r = new Run(doc, value);
                        r.Font.Size = ft.Size;
                        //p.AppendChild(new Run(doc, value));
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft.Size;
                        p.AppendChild(r);

                        HandleFontSubscript(listFont, p);

                        //if (listFont != null && listFont.Count > 0)
                        //{
                        //    foreach (var st in listFont)
                        //    {
                        //        Regex regex = new Regex(st.ReplaceText, RegexOptions.IgnoreCase);
                        //        p.Range.Replace(regex, new ReplaceEvaluatorFindAndHighlight(st.FontType), false);
                        //    }
                        //}
                    }
                }
                else
                {
                    if (cell == 0)
                    {
                        if (oldValue == value)
                        {
                            c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.Previous;
                            c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        }
                        else
                        {
                            oldValue = value;
                            c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.First;
                            c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                            listFont = GetFontInfoList(value);
                            //p.AppendChild(new Run(doc, oldValue));
                            r = new Run(doc, oldValue);
                            r.Font.Size = ft.Size;
                            p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                            //p.ParagraphFormat.Style.Font.Size = ft.Size;
                            p.AppendChild(r);

                            HandleFontSubscript(listFont, p);
                        }
                    }
                    else
                    {
                        //oldValue = value;
                        c.CellFormat.VerticalMerge = Aspose.Words.Tables.CellMerge.None;
                        c.CellFormat.VerticalAlignment = CellVerticalAlignment.Center;
                        listFont = GetFontInfoList(value);
                        //p.AppendChild(new Run(doc, value));
                        r = new Run(doc, value);
                        r.Font.Size = ft.Size;
                        p.ParagraphFormat.Alignment = ParagraphAlignment.Center;
                        //p.ParagraphFormat.Style.Font.Size = ft.Size;
                        p.AppendChild(r);

                        HandleFontSubscript(listFont, p);
                    }
                }
                c.FirstParagraph.Remove();
                c.AppendChild(p);
                table.Rows[row].Cells[cell].Remove();
                table.Rows[row].Cells.Insert(cell, c);
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return table;
        }

        /// <summary>
        /// 获取拆分上下标的字符串
        /// </summary>
        /// <param name="str"></param>
        /// <returns></returns>
        public List<FontSubscript> GetFontInfoList(string str)
        {
            List<FontSubscript> list = new List<FontSubscript>();
            //str = str + "{-3 }";
            if (!string.IsNullOrEmpty(str))
            {
                Dictionary<string, string> dicList = new Dictionary<string, string>();
                dicList.Add("{(.*?)}", "{{{0}}}");
                //dicList.Add("{(.)}", "{{{0}}}");
                //dicList.Add("\\[(.*?)\\]", "\\[{0}\\]");
                dicList.Add("\\[(.*?)\\]", "\\[{0}\\]");
                //dicList.Add("\\[(.)\\]", "\\[{0}\\]");
                //dicList.Add("<sub>(.*?)</sub>", "<sub>{0}</sub>");
                //dicList.Add("<sup>(.*?)</sup>", "<sup>{0}</sup>");
                //string[] arr = new string[] {"{2}", "{23}"};
                //List<string> list = new List<string>();
                //List<FontSubscript> listFont = new List<FontSubscript>();
                foreach (var dic in dicList)
                {
                    MatchCollection mc = Regex.Matches(str, dic.Key);
                    if (mc != null && mc.Count > 0)
                    {
                        FontSubscript ety;
                        for (int i = 0; i < mc.Count; i++)
                        {
                            ety = new FontSubscript();
                            if (mc[i].Value.Contains("]") && mc[i].Value.Contains("["))
                            {
                                ety.FontType = 1;//下标
                                                 //ety.ReplaceText = mc[i].Value.Replace("[", "");
                                ety.ReplaceText = "\\[" + mc[i].Groups[1].Value + "\\]";
                                ety.ReplaceValue = mc[i].Groups[1].Value;
                                list.Add(ety);
                            }
                            if (mc[i].Value.Contains("}") && mc[i].Value.Contains("{"))
                            {
                                ety.FontType = 0;//上标
                                                 //ety.ReplaceText = mc[i].Value.Replace("}", "");
                                ety.ReplaceText = "\\{" + mc[i].Groups[1].Value + "\\}";
                                ety.ReplaceValue = mc[i].Groups[1].Value;
                                list.Add(ety);
                            }
                        }
                    }
                }
            }
            return list;
        }

        /// <summary>
        /// 处理上下标
        /// </summary>
        /// <param name="list"></param>
        /// <param name="p"></param>
        public void HandleFontSubscript(List<FontSubscript> list, Paragraph p)
        {
            if (list != null && list.Count > 0)
            {
                foreach (var st in list)
                {
                    Regex regex = new Regex(st.ReplaceText, RegexOptions.IgnoreCase);
                    p.Range.Replace(regex, new ReplaceEvaluatorFindAndHighlight(st.FontType), false);
                }
            }
        }

        public void HandleFontSubscript(List<FontSubscript> list, WordHelper wh)
        {
            if (list != null && list.Count > 0)
            {
                foreach (var st in list)
                {
                    Regex regex = new Regex(st.ReplaceText, RegexOptions.IgnoreCase);
                    wh.doc.Range.Replace(regex, new ReplaceEvaluatorFindAndHighlight(st.FontType), false);
                }
            }
        }

        //Dictionary<string, string> dicList = new Dictionary<string, string>();
        //dicList.Add("{(.*?)}", "{{{0}}}");

        //处理文本中某个字符的样式
        private class ReplaceEvaluatorFindAndHighlight : IReplacingCallback
        {
            public int type { get; set; }

            public ReplaceEvaluatorFindAndHighlight(int type)//0:上标；1：下标
            {
                this.type = type;
            }

            /// <summary>
            /// This method is called by the Aspose.Words find and replace engine for each match.
            /// This method highlights the match string, even if it spans multiple runs.
            /// </summary>
            ReplaceAction IReplacingCallback.Replacing(ReplacingArgs e)
            {
                Node currentNode = e.MatchNode;
                if (e.MatchOffset > 0)
                    currentNode = SplitRun((Run)currentNode, e.MatchOffset);
                ArrayList runs = new ArrayList();

                int remainingLength = e.Match.Value.Length;
                while (
                    (remainingLength > 0) &&
                    (currentNode != null) &&
                    (currentNode.GetText().Length <= remainingLength))
                {
                    runs.Add(currentNode);
                    remainingLength = remainingLength - currentNode.GetText().Length;

                    do
                    {
                        currentNode = currentNode.NextSibling;
                    }
                    while ((currentNode != null) && (currentNode.NodeType != NodeType.Run));
                }

                // Split the last run that contains the match if there is any text left.
                if ((currentNode != null) && (remainingLength > 0))
                {
                    SplitRun((Run)currentNode, remainingLength);
                    runs.Add(currentNode);
                }

                // Now highlight all runs in the sequence.
                foreach (Run run in runs)
                {
                    //run.Font.HighlightColor = Color.Yellow;
                    //run.Font.Name = "宋体";
                    //run.Font.Size = 16;
                    //run.Text = "100";
                    ////变成上标
                    string aa = run.Text;

                    if (this.type == 0)
                    {
                        run.Text = run.Text.Replace("{", "").Replace("}", "");
                        run.Font.Superscript = true;//上标
                    }
                    else
                    {
                        run.Text = run.Text.Replace("[", "").Replace("]", "");
                        run.Font.Subscript = true;//下标
                        //run.Text= run.Text.Replace("$", "[").Replace("&", "]");
                    }
                }
                return ReplaceAction.Skip;
            }

            //private Node SplitRun(Run currentNode, int matchOffset)
            //{
            //    throw new NotImplementedException();
            //}
        }

        private static Run SplitRun(Run run, int position)
        {
            Run afterRun = (Run)run.Clone(true);
            afterRun.Text = run.Text.Substring(position);
            run.Text = run.Text.Substring(0, position);
            run.ParentNode.InsertAfter(afterRun, run);
            return afterRun;
        }

        public ParagraphCollection WordParagraphs(string fileName)
        {
            Document doc = new Document(fileName);
            if (doc.FirstSection.Body.Paragraphs.Count > 0)
            {
                return doc.FirstSection.Body.Paragraphs;//word中的所有段落
            }
            return null;
        }

        /// <summary>
        /// Word导出
        /// </summary>
        /// <param name="savePath">转换成Pdf的保存路径</param>
        /// <param name="wordFileName">转换成文件名字</param>
        public void SaveWord(string savePath, string wordFileName)
        {
            doc.Save(savePath + "\\" + wordFileName + ".docx");
        }

        /// <summary>
        /// Word导出
        /// </summary>
        /// <param name="fileName">转换成Pdf的保存路径</param> 
        public void SaveWord(string fileName)
        {
            doc.Save(fileName);
        }

        /// <summary>
        /// Word转成Pdf
        /// </summary>
        /// <param name="savePath">转换成Pdf的保存路径</param>
        /// <param name="wordFileName">转换成文件名字</param>
        public void SaveDocWord(string savePath, string wordFileName)
        {
            doc.Save(savePath + "\\" + wordFileName + ".doc");
        }

        /// <summary>
        /// 打印Word
        /// </summary>
        /// <param name="savePath">转换成Pdf的保存路径</param>
        /// <param name="wordFileName">转换成文件名字</param>
        public void PrintWord()
        {
            doc.Print();
        }

        /// <summary>
        /// Word转成Pdf
        /// </summary>

        /// <param name="savePath">转换成Pdf的保存路径</param>
        /// <param name="wordFileName">转换成html的文件名字</param>
        public void WordToPDF(string savePath, string wordFileName)
        {
            //DocumentBuilder builder = new DocumentBuilder(doc);
            //builder.Writeln("Test Signed PDF.");
            //X509Certificate2 cert = new X509Certificate2(System.Windows.Forms.Application.StartupPath + @"\Temp\" + "Ponymedicine.pfx", "Ponymedicine***");

            //Aspose.Words.Saving.PdfSaveOptions saveOption = new Aspose.Words.Saving.PdfSaveOptions();
            //saveOption.SaveFormat = Aspose.Words.SaveFormat.Pdf;
            //PdfEncryptionDetails encryptionDetails = new PdfEncryptionDetails(string.Empty, "Pony1234", PdfEncryptionAlgorithm.RC4_128);
            //encryptionDetails.Permissions = PdfPermissions.DisallowAll;
            //encryptionDetails.Permissions = PdfPermissions.Printing;
            //saveOption.EncryptionDetails = encryptionDetails;

            //saveOption.DigitalSignatureDetails = new PdfDigitalSignatureDetails(cert, "Test Signing", "Aspose Office", DateTime.Now);
            //doc.Save(savePath + wordFileName + ".pdf", saveOption);
            //PDFSign(savePath + wordFileName + ".pdf");
            doc.Save(savePath + wordFileName + ".pdf", SaveFormat.Pdf);
        }

        public void PDFSign(string savePath)
        {
            using (Aspose.Pdf.Document pdfDocument = new Aspose.Pdf.Document(savePath))
            {
                using (PdfFileSignature signature = new PdfFileSignature(pdfDocument))
                {
                    PKCS7 pkcs = new PKCS7(System.Windows.Forms.Application.StartupPath + @"\Temp\" + "puni.pfx", "Ponytest"); // Use PKCS7/PKCS7Detached objects
                    DocMDPSignature docMdpSignature = new DocMDPSignature(pkcs, DocMDPAccessPermissions.FillingInForms);
                    System.Drawing.Rectangle rect = new System.Drawing.Rectangle(100, 100, 200, 100);
                    // Set signature appearance
                    signature.SignatureAppearance = System.Windows.Forms.Application.StartupPath + @"\Temp\" + "123.bmp";
                    // Create any of the three signature types
                    signature.Certify(1, "Signature Reason", "Contact", "Location", true, rect, docMdpSignature);
                    // Save digitally signed PDF file
                    signature.Save(savePath);
                }
            }
        }

        /// <summary>
        /// Word转成Pdf
        /// </summary>
        /// <param name="path">要转换的文档的路径</param>
        /// <param name="savePath">转换成Pdf的保存路径</param>
        /// <param name="wordFileName">转换成html的文件名字</param>
        public void WordToPDFT(string path, string savePath, string wordFileName)
        {
            Aspose.Words.Document d = new Aspose.Words.Document(path);
            doc.Save(savePath + wordFileName + ".pdf", SaveFormat.Pdf);
        }

        /// <summary>
        /// Word转成jpg
        /// </summary>
        /// <param name="path">要转换的文档的路径</param>
        /// <param name="savePath">转换成Pdf的保存路径</param>
        /// <param name="wordFileName">转换成html的文件名字</param>
        public void WordToJPG(string path, string savePath, string wordFileName)
        {
            ImageSaveOptions iso = new ImageSaveOptions(SaveFormat.Jpeg);
            iso.Resolution = 128;
            iso.PrettyFormat = true;
            iso.UseAntiAliasing = true;
            for (int i = 0; i < doc.PageCount; i++)
            {
                iso.PageIndex = i;
                doc.Save("D:/test/test" + i + ".jpg", iso);
            }
        }

        #region 判断远程文件是否存在

        /// <summary>
        /// 判断远程文件是否存在
        /// </summary>
        /// <param name="fileUrl"></param>
        /// <returns></returns>
        public static bool RemoteFileExists(string fileUrl)
        {
            HttpWebRequest re = null;
            HttpWebResponse res = null;
            try
            {
                re = (HttpWebRequest)WebRequest.Create(fileUrl);
                res = (HttpWebResponse)re.GetResponse();
                if (res.ContentLength != 0)
                {
                    //MessageBox.Show("文件存在");
                    return true;
                }
            }
            catch (Exception)
            {
                //MessageBox.Show("无此文件");
                return false;
            }
            finally
            {
                if (re != null)
                {
                    re.Abort();//销毁关闭连接
                }
                if (res != null)
                {
                    res.Close();//销毁关闭响应
                }
            }
            return false;
        }

        #endregion 判断远程文件是否存在
    }

    public class FontSubscript
    {
        public int FontType { get; set; }//上标，还是下标
        public string ReplaceText { get; set; }//替换文本
        public string ReplaceValue { get; set; }//替换值
    }

    public class ReplaceAndInsertImage : IReplacingCallback
    {
        /// <summary>
        /// 需要插入的图片路径
        /// </summary>
        public string url { get; set; }

        public ReplaceAndInsertImage(string url)
        {
            this.url = url;
        }

        public ReplaceAction Replacing(ReplacingArgs e)
        {
            //获取当前节点
            var node = e.MatchNode;
            //获取当前文档
            //Document doc = node.Document as Document;
            //DocumentBuilder builder = new DocumentBuilder(doc);
            ////将光标移动到指定节点
            //builder.MoveTo(node);
            ////插入图片
            //builder.InsertImage(url);
            return ReplaceAction.Replace;
        }
    }
}
```
</details>

