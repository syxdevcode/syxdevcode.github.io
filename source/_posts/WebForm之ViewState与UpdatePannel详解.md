---
title: WebForm之ViewState与UpdatePannel详解
date: 2019-02-19 14:16:35
tags:
- DotNet
- WebForm
- ASP.NET/IIS请求处理管道
categories: 
- WebForm
---
# WebForm之ViewState与UpdatePannel详解

## ViewState

### ViewState简介

1,类似于Dictionary的一种数据结构

ViewState通过String类型的数据作为索引。ViewState对应项中的值可以存储任何类型的值（参数是Object类型），任何类型的值存储到ViewState中都会被装箱为Object类型。

注：ViewState不能存储所有的数据类型，仅支持以下的这几种：
String、Integer、Boolean、Array、ArrayList、Hashtable以及一些自定义类型。

2，ViewState存储在浏览器的页面之中

ViewState的作用域是页面，消耗服务器资源相对较少。

服务器在向浏览器返回html之前，对ViewState中的内容进行了Base64的加密编码。VIEWSTATE适用于同一个页面在不关闭的情况下多次与服务器交互（PostBack）

当用户点击页面中的某个按钮提交表单时，浏览器会将这个_VIEWSTATE的隐藏域也一起提交到服务端；服务器端在解析请求时，会将浏览器提交过来的ViewState进行反序列化后填充到ViewState属性中；再根据业务处理需要，从这个属性中根据索引找到具体的Value值并对其进行操作；操作完成后，再将ViewState进行Base64编码再次返回给浏览器端；

```cs
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwUKLTg4NTU3NzYwOGRkbQwtdx3+TX000hqmVYIcP3+P3zcihvV0deiDjaSgFRQ=" />
```

### ViewState存在的问题

当 `Repeater`等控件使用时，存储较大的值，用户请求显示页面的速度会减慢。

```cs
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwUJLTQ2OTExMTk1D2QWAgIBD2QWBAIBDxAPFgYeDkRhdGFWYWx1ZUZpZWxkBQJJRB4NRGF0YVRleHRGaWVsZAUETmFtZR4LXyFEYXRhQm91vZflubPop4Lpn7PooZfmsp/pgJoxMDDljoUn5LqR5Y2X6L+q5L+h6YCa572X5bmz6LSi5a+M5rKf6YCaMTAw5Y6FKuS6keWNl+i/quS/oUCBTExNDczBTExNDczZAIDDw8WAh8FBQUxMTQ3M2RkAgUPDxYCHwUFCzE4Mzg4MDM4MTY1ZGQCBQ8PFgIeC1JlY29yZGNvdW50AvIDZGRkmmJuzxcRIKhlTny8ibiHOR92G/JYSgGwzO69sFxoLV8=" />
```

### 禁用ViewState

1,页面级禁用，在Page指令集中添加EnableViewState="false"。

```cs
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="RepeaterViewState.aspx.cs"
    Inherits="WebForm1.RepeaterViewState" EnableViewState="false" %>
```

注：禁用之后页面依然存在ViewState，但数据量已经减小。

2，控件级禁用ViewState

控件设置一个属性EnableViewState="false"。

```cs
<asp:Repeater ID="repeaterProducts" runat="server" EnableViewState="false">
</asp:Repeater>
```

3,全局级禁用ViewState

Web.config的system.web中增加一句配置

```cs
<pages enableViewState="false" />
```

4,单独控制禁用与启用

ASP.NET4.0之前，控件的视图只有在 Page 的 ViewState 启用的前提下才可以单独控制。

ASP.NET4.0供了一个叫做 ViewStateMode 的新属性，这个属性可以单独设置控件的视图状态。即使页面的 `ViewState` 没有启用，控件依然可以启用视图状态。

ViewStateMode 属性有三种取值：

* Inherit：视图状态从父控件继承；
* Enabled：即使父控件的视图状态没有启用，也启用该控件的视图状态；
* Disabled：即使父控件的视图状态启用了，也禁用此控件的视图状态。

## UpdatePanel控件

```cs
 <form id="form1" runat="server">
    <div align="center">
        <asp:ScriptManager ID="scriptManager" runat="server">
        </asp:ScriptManager>
        <asp:UpdatePanel ID="updatePanel" runat="server">
            <ContentTemplate>
                <asp:TextBox ID="txtNumber1" runat="server"></asp:TextBox>
                <asp:DropDownList ID="ddlFunc" runat="server">
                    <asp:ListItem Value="0">+</asp:ListItem>
                    <asp:ListItem Value="1">-</asp:ListItem>
                    <asp:ListItem Value="2">*</asp:ListItem>
                    <asp:ListItem Value="3">/</asp:ListItem>
                </asp:DropDownList>
                <asp:TextBox ID="txtNumber2" runat="server"></asp:TextBox>
                <asp:Button ID="btnGetResult" runat="server" Text="=" Width="50" OnClick="btnGetResult_Click" />
                <asp:Label ID="lblResult" runat="server" Text="" Font-Bold="true"></asp:Label>
            </ContentTemplate>
        </asp:UpdatePanel>
    </div>
</form>
```

UpdatePanel本质封装了以XmlHttpRequest为核心的一系列方法帮我们将CodeBehind中的同步事件变为了异步操作，并通过DOM更新指定的HTML内容，使得我们可以方便地实现AJAX效果。

参考：

[ASP.Net WebForm温故知新学习笔记：二、ViewState与UpdatePanel探秘](http://www.cnblogs.com/edisonchou/p/3901559.html)

[禁掉VIEWSTATE之后（一）](http://www.cnblogs.com/freeflying/archive/2009/12/28/1634229.html)

[Asp.Net 4.0 新特性，输出更纯净的Html代码 ClientIDMode，ViewStateMode等](http://www.cnblogs.com/yukaizhao/archive/2010/05/22/asp-net-new-feature-pure-html.html)