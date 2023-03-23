---
title: DotNet6调用Soap
date: 2023-02-27 10:52:53
tags:
- DotNet
- CSharp
- WCF
- WebService
- Soap
categories: 
- WebService
---

## SoapUI获取请求参数

通过SoapUI获取到PostMan请求参数：

SoapUI下载地址：[https://www.soapui.org/downloads/soapui/](https://www.soapui.org/downloads/soapui/)

1，新建Soap项目
2，选择 通过VS生成的`.wsdl`文件。
![Snipaste_2023-02-27_11-02-13.png](/img1/Snipaste_2023-02-27_11-02-13.png)

3，选择需要测试的方法，右键新建请求

![Snipaste_2023-02-27_11-08-44.png](/img1/Snipaste_2023-02-27_11-08-44.png)
![Snipaste_2023-02-27_11-09-11.png](/img1/Snipaste_2023-02-27_11-09-11.png)

<!--more-->
## 帮助类

工具类：

```cs
/// <summary>
///  Soap帮助类
/// </summary>
public class SoapHelper
{
    //<webServices>
    //  <protocols>
    //    <add name="HttpGet"/>
    //    <add name="HttpPost"/>
    //  </protocols>
    //</webServices>
    private static Hashtable _xmlNamespaces = new Hashtable();//缓存xmlNamespace，避免重复调用GetNamespace

    /// <summary>
    /// 需要WebService支持Post调用
    /// </summary>
    public static XmlDocument QueryPostWebService(String url, String methodName, Hashtable pars)
    {
        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(url + "/" + methodName);
        request.Method = "POST";
        request.ContentType = "application/x-www-form-urlencoded";
        SetWebRequest(request);
        byte[] data = EncodePars(pars);
        WriteRequestData(request, data);
        return ReadXmlResponse(request.GetResponse());
    }

    /// <summary>
    /// 需要WebService支持Get调用
    /// </summary>
    public static XmlDocument QueryGetWebService(String url, String methodName, Hashtable pars)
    {
        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(url + "/" + methodName + "?" + ParsToString(pars));
        request.Method = "GET";
        request.ContentType = "application/x-www-form-urlencoded";
        SetWebRequest(request);
        return ReadXmlResponse(request.GetResponse());
    }


    /// <summary>
    /// 通用WebService调用(Soap),参数Pars为String类型的参数名、参数值
    /// </summary>
    public static XmlDocument QuerySoapWebService(String url, String methodName, Hashtable pars, bool isSoapEnv = false)
    {
        if (_xmlNamespaces.ContainsKey(url))
        {
            return QuerySoapWebService(url, methodName, pars, _xmlNamespaces[url].ToString(), isSoapEnv);
        }
        else
        {
            return QuerySoapWebService(url, methodName, pars, GetNamespace(url), isSoapEnv);
        }
    }

    /// <summary>
    /// 通用WebService调用(Soap)
    /// </summary>
    /// <param name="url"></param>
    /// <param name="methodName"></param>
    /// <param name="pars"></param>
    /// <param name="xmlNs"></param>
    /// <param name="isSoapEnv"></param>
    /// <returns></returns>
    private static XmlDocument QuerySoapWebService(String url, String methodName, Hashtable pars, string xmlNs, bool isSoapEnv)
    {
        _xmlNamespaces[url] = xmlNs;//加入缓存，提高效率
        HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(url);
        request.Method = "POST";
        request.ContentType = "text/xml; charset=utf-8";

        if (!isSoapEnv)
        {
            request.Headers.Add("SOAPAction", "\"" + xmlNs + (xmlNs.EndsWith("/") ? "" : "/") + methodName + "\"");
        }

        SetWebRequest(request);
        byte[] data = isSoapEnv ? EncodeParsToSoapEnv(pars, xmlNs, methodName) : EncodeParsToSoap(pars, xmlNs, methodName);
        WriteRequestData(request, data);
        XmlDocument doc = new XmlDocument(), doc2 = new XmlDocument();
        doc = ReadXmlResponse(request.GetResponse());
        XmlNamespaceManager mgr = new XmlNamespaceManager(doc.NameTable);
        mgr.AddNamespace("soap", "http://schemas.xmlsoap.org/soap/envelope/");
        String RetXml = doc.SelectSingleNode("//soap:Body/*/*", mgr).InnerXml;
        doc2.LoadXml("<root>" + RetXml + "</root>");
        AddDelaration(doc2);
        return doc2;
    }

    /// <summary>
    /// 通过WebService的WSDL获取XML名称空间
    /// </summary>
    /// <param name="url"></param>
    /// <returns></returns>
    private static string GetNamespace(String url)
    {
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url + "?WSDL");
        SetWebRequest(request);
        WebResponse response = request.GetResponse();
        StreamReader sr = new StreamReader(response.GetResponseStream(), Encoding.UTF8);
        XmlDocument doc = new XmlDocument();
        doc.LoadXml(sr.ReadToEnd());
        sr.Close();
        return doc.SelectSingleNode("//@targetNamespace").Value;
    }

    /// <summary>
    /// 动态生成SOP请求报文内容
    /// </summary>
    /// <param name="pars"></param>
    /// <param name="xmlNs"></param>
    /// <param name="methodName"></param>
    /// <returns></returns>
    private static byte[] EncodeParsToSoap(Hashtable pars, String xmlNs, String methodName)
    {
        XmlDocument doc = new XmlDocument();
        doc.LoadXml("<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\"></soap:Envelope>");
        AddDelaration(doc);
        XmlElement soapBody = doc.CreateElement("soap", "Body", "http://schemas.xmlsoap.org/soap/envelope/");
        XmlElement soapMethod = doc.CreateElement(methodName);
        soapMethod.SetAttribute("xmlns", xmlNs);
        foreach (string k in pars.Keys)
        {
            XmlElement soapPar = doc.CreateElement(k);
            soapPar.InnerXml = ObjectToSoapXml(pars[k]);
            soapMethod.AppendChild(soapPar);
        }
        soapBody.AppendChild(soapMethod);
        doc.DocumentElement.AppendChild(soapBody);
        return Encoding.UTF8.GetBytes(doc.OuterXml);
    }

    /// <summary>
    /// 动态生成SOP请求报文内容
    /// </summary>
    /// <param name="pars"></param>
    /// <param name="xmlNs"></param>
    /// <param name="methodName"></param>
    /// <returns></returns>
    private static byte[] EncodeParsToSoapEnv(Hashtable pars, String xmlNs, String methodName)
    {
        XmlDocument doc = new XmlDocument();
        doc.LoadXml($"<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:web=\"{xmlNs}\"><soapenv:Header/></soapenv:Envelope>");
        AddDelaration(doc);
        XmlElement soapBody = doc.CreateElement("soapenv", "Body", "http://schemas.xmlsoap.org/soap/envelope/");
        XmlElement soapMethod = doc.CreateElement("web", methodName, xmlNs);
        //soapMethod.SetAttribute("xmlns", xmlNs);
        foreach (string k in pars.Keys)
        {
            XmlElement soapPar = doc.CreateElement(k);
            soapPar.InnerXml = ObjectToSoapXml(pars[k]);
            soapMethod.AppendChild(soapPar);
        }
        soapBody.AppendChild(soapMethod);
        doc.DocumentElement.AppendChild(soapBody);
        return Encoding.UTF8.GetBytes(doc.OuterXml);
    }

    /// <summary>
    /// 将对象转换成XML节点格式
    /// </summary>
    /// <param name="o"></param>
    /// <returns></returns>
    private static string ObjectToSoapXml(object o)
    {
        XmlSerializer mySerializer = new XmlSerializer(o.GetType());
        MemoryStream ms = new MemoryStream();
        mySerializer.Serialize(ms, o);
        XmlDocument doc = new XmlDocument();
        doc.LoadXml(Encoding.UTF8.GetString(ms.ToArray()));
        if (doc.DocumentElement != null)
        {
            return doc.DocumentElement.InnerXml;
        }
        else
        {
            return o.ToString();
        }
    }

    /// <summary>
    /// 设置WEB请求
    /// </summary>
    /// <param name="request"></param>
    private static void SetWebRequest(HttpWebRequest request)
    {
        request.Credentials = CredentialCache.DefaultCredentials;
        request.Timeout = 10000;
    }

    /// <summary>
    /// 设置请求数据
    /// </summary>
    /// <param name="request"></param>
    /// <param name="data"></param>
    private static void WriteRequestData(HttpWebRequest request, byte[] data)
    {
        request.ContentLength = data.Length;
        Stream writer = request.GetRequestStream();
        writer.Write(data, 0, data.Length);
        writer.Close();
    }

    /// <summary>
    /// 获取字符串的UTF8码字符串
    /// </summary>
    /// <param name="pars"></param>
    /// <returns></returns>
    private static byte[] EncodePars(Hashtable pars)
    {
        return Encoding.UTF8.GetBytes(ParsToString(pars));
    }

    /// <summary>
    /// 将Hashtable转换成WEB请求键值对字符串
    /// </summary>
    /// <param name="pars"></param>
    /// <returns></returns>
    private static String ParsToString(Hashtable pars)
    {
        StringBuilder sb = new StringBuilder();
        foreach (string k in pars.Keys)
        {
            if (sb.Length > 0)
            {
                sb.Append("&");
            }
            sb.Append(HttpUtility.UrlEncode(k) + "=" + HttpUtility.UrlEncode(pars[k].ToString()));
        }
        return sb.ToString();
    }

    /// <summary>
    /// 获取Webservice响应报文XML
    /// </summary>
    /// <param name="response"></param>
    /// <returns></returns>
    private static XmlDocument ReadXmlResponse(WebResponse response)
    {
        StreamReader sr = new StreamReader(response.GetResponseStream(), Encoding.UTF8);
        String retXml = sr.ReadToEnd();
        sr.Close();
        XmlDocument doc = new XmlDocument();
        doc.LoadXml(retXml);
        return doc;
    }

    /// <summary>
    /// 设置XML文档版本声明
    /// </summary>
    /// <param name="doc"></param>
    private static void AddDelaration(XmlDocument doc)
    {
        XmlDeclaration decl = doc.CreateXmlDeclaration("1.0", "utf-8", null);
        doc.InsertBefore(decl, doc.DocumentElement);
    }
}
```

版本2：

```cs
using Furion.RemoteRequest.Extensions;
using System.Collections;
using System.IO;
using System.Net;
using System.Text;
using System.Web;
using System.Xml;
using System.Xml.Serialization;

namespace Admin.NET.Application;

/// <summary>
///  Soap帮助类
/// </summary>
public class SoapHelper : ISingleton
{
    public SoapHelper()
    {
    }

    //<webServices>
    //  <protocols>
    //    <add name="HttpGet"/>
    //    <add name="HttpPost"/>
    //  </protocols>
    //</webServices>
    private Hashtable _xmlNamespaces = new();//缓存xmlNamespace，避免重复调用GetNamespace

    /// <summary>
    /// Post调用
    /// </summary>
    public async Task<XmlDocument> QueryPostWebService(String url, String methodName, Hashtable pars, string clientName = "clientName")
    {
        // 设置请求拦截
        var response = await url.SetClient(clientName).SetBody(pars, "application/x-www-form-urlencoded", Encoding.UTF8).PostAsStringAsync();
        XmlDocument doc = new();
        doc.LoadXml(response);
        return doc;
    }

    /// <summary>
    /// 需要WebService支持Get调用
    /// </summary>
    public async Task<XmlDocument> QueryGetWebService(String url, String methodName, Hashtable pars, string clientName = "clientName")
    {
        var requestUrl = $"{url}/{methodName}?{ParsToString(pars)}";
        // 设置请求拦截
        var response = await requestUrl.SetClient(clientName).SetContentType("application/x-www-form-urlencoded").SetContentEncoding(Encoding.UTF8).GetAsStringAsync();
        XmlDocument doc = new();
        doc.LoadXml(response);
        return doc;
    }

    /// <summary>
    /// 通用WebService调用(Soap),参数Pars为String类型的参数名、参数值
    /// </summary>
    public async Task<XmlDocument> QuerySoapWebService(String url, String methodName, Hashtable pars, bool isSoapEnv = false, string clientName = "clientName")
    {
        if (_xmlNamespaces.ContainsKey(url))
        {
            return await QuerySoapWebService(url, methodName, pars, _xmlNamespaces[url].ToString(), isSoapEnv, clientName);
        }
        else
        {
            var xmlNx = await GetNamespace(url, clientName);
            return await QuerySoapWebService(url, methodName, pars, xmlNx, isSoapEnv, clientName);
        }
    }

    /// <summary>
    /// 通用WebService调用(Soap)
    /// </summary>
    /// <param name="url"></param>
    /// <param name="methodName"></param>
    /// <param name="pars"></param>
    /// <param name="xmlNs"></param>
    /// <param name="isSoapEnv"></param>
    /// <param name="clientName"></param>
    /// <returns></returns>
    private async Task<XmlDocument> QuerySoapWebService(String url, String methodName, Hashtable pars, string xmlNs, bool isSoapEnv, string clientName)
    {
        _xmlNamespaces[url] = xmlNs;//加入缓存，提高效率

        var data = isSoapEnv ? EncodeParsToSoapEnv(pars, xmlNs, methodName) : EncodeParsToSoap(pars, xmlNs, methodName);

        // 设置请求拦截
        var request = url.SetClient(clientName).SetBody(data, "text/xml", Encoding.UTF8);

        if (!isSoapEnv)
        {
            request.Headers.Add("SOAPAction", "\"" + xmlNs + (xmlNs.EndsWith("/") ? "" : "/") + methodName + "\"");
        }

        var response = await request.PostAsStringAsync();
        XmlDocument doc = new(), doc2 = new();
        doc.LoadXml(response);

        XmlNamespaceManager mgr = new(doc.NameTable);
        mgr.AddNamespace("soap", "http://schemas.xmlsoap.org/soap/envelope/");
        String RetXml = doc.SelectSingleNode("//soap:Body/*/*", mgr).InnerXml;
        doc2.LoadXml("<root>" + RetXml + "</root>");
        AddDelaration(doc2);
        return doc2;
    }

    /// <summary>
    /// 通过WebService的WSDL获取XML名称空间
    /// </summary>
    /// <param name="url"></param>
    /// <param name="clientName"></param>
    /// <returns></returns>
    private async Task<string> GetNamespace(String url, string clientName)
    {
        // 设置请求拦截
        var response = await $"{url}?WSDL".SetClient(clientName).GetAsStringAsync();
        XmlDocument doc = new();
        doc.LoadXml(response);
        return doc.SelectSingleNode("//@targetNamespace").Value;
    }

    /// <summary>
    /// 动态生成SOP请求报文内容
    /// </summary>
    /// <param name="pars"></param>
    /// <param name="xmlNs"></param>
    /// <param name="methodName"></param>
    /// <returns></returns>
    private string EncodeParsToSoap(Hashtable pars, String xmlNs, String methodName)
    {
        XmlDocument doc = new();
        doc.LoadXml("<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\"></soap:Envelope>");
        AddDelaration(doc);
        XmlElement soapBody = doc.CreateElement("soap", "Body", "http://schemas.xmlsoap.org/soap/envelope/");
        XmlElement soapMethod = doc.CreateElement(methodName);
        soapMethod.SetAttribute("xmlns", xmlNs);
        foreach (string k in pars.Keys)
        {
            XmlElement soapPar = doc.CreateElement(k);
            soapPar.InnerXml = ObjectToSoapXml(pars[k]);
            soapMethod.AppendChild(soapPar);
        }
        soapBody.AppendChild(soapMethod);
        doc.DocumentElement.AppendChild(soapBody);
        return doc.OuterXml;
    }

    /// <summary>
    /// 动态生成SOP请求报文内容
    /// </summary>
    /// <param name="pars"></param>
    /// <param name="xmlNs"></param>
    /// <param name="methodName"></param>
    /// <returns></returns>
    private string EncodeParsToSoapEnv(Hashtable pars, String xmlNs, String methodName)
    {
        XmlDocument doc = new XmlDocument();
        doc.LoadXml($"<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:web=\"{xmlNs}\"><soapenv:Header/></soapenv:Envelope>");
        AddDelaration(doc);
        XmlElement soapBody = doc.CreateElement("soapenv", "Body", "http://schemas.xmlsoap.org/soap/envelope/");
        XmlElement soapMethod = doc.CreateElement("web", methodName, xmlNs);
        //soapMethod.SetAttribute("xmlns", xmlNs);
        foreach (string k in pars.Keys)
        {
            XmlElement soapPar = doc.CreateElement(k);
            soapPar.InnerXml = ObjectToSoapXml(pars[k]);
            soapMethod.AppendChild(soapPar);
        }
        soapBody.AppendChild(soapMethod);
        doc.DocumentElement.AppendChild(soapBody);
        //return Encoding.UTF8.GetBytes(doc.OuterXml);
        return doc.OuterXml;
    }

    /// <summary>
    /// 将对象转换成XML节点格式
    /// </summary>
    /// <param name="o"></param>
    /// <returns></returns>
    private string ObjectToSoapXml(object o)
    {
                var ms = new MemoryStream();
        using (XmlWriter xmlWriter = XmlWriter.Create(ms, new XmlWriterSettings() { Encoding = new UTF8Encoding(false), OmitXmlDeclaration = true }))//使用UTF8Encoding
        {
            XmlSerializer xz = new(o.GetType());
            XmlSerializerNamespaces ns = new();
            ns.Add(string.Empty, string.Empty);//去掉xmlns属性
            xz.Serialize(xmlWriter, o, ns);
            
            var xml = Encoding.UTF8.GetString(ms.ToArray());//得到xml，不含BOM 

            XmlDocument doc = new();
            doc.LoadXml(xml);

            if (doc.DocumentElement != null)
            {
                return doc.DocumentElement.InnerXml;
            }

            return o.ToString();
        }


        //XmlSerializer mySerializer = new XmlSerializer(o.GetType());
        //MemoryStream ms = new MemoryStream();
        //mySerializer.Serialize(ms, o);
        //XmlDocument doc = new XmlDocument();
        //var str = Encoding.UTF8.GetString(ms.ToArray());
        //Log.Information($"将对象转换成XML节点格式:{str}");
        //doc.LoadXml(str);
        //if (doc.DocumentElement != null)
        //{
        //    return doc.DocumentElement.InnerXml;
        //}
        //else
        //{
        //    return o.ToString();
        //}
    }

    /// <summary>
    /// 设置WEB请求
    /// </summary>
    /// <param name="request"></param>
    private void SetWebRequest(HttpWebRequest request)
    {
        request.Credentials = CredentialCache.DefaultCredentials;
        request.Timeout = 10000;
    }

    /// <summary>
    /// 设置请求数据
    /// </summary>
    /// <param name="request"></param>
    /// <param name="data"></param>
    private void WriteRequestData(HttpWebRequest request, byte[] data)
    {
        request.ContentLength = data.Length;
        Stream writer = request.GetRequestStream();
        writer.Write(data, 0, data.Length);
        writer.Close();
    }

    /// <summary>
    /// 获取字符串的UTF8码字符串
    /// </summary>
    /// <param name="pars"></param>
    /// <returns></returns>
    private byte[] EncodePars(Hashtable pars)
    {
        return Encoding.UTF8.GetBytes(ParsToString(pars));
    }

    /// <summary>
    /// 将Hashtable转换成WEB请求键值对字符串
    /// </summary>
    /// <param name="pars"></param>
    /// <returns></returns>
    private String ParsToString(Hashtable pars)
    {
        StringBuilder sb = new StringBuilder();
        foreach (string k in pars.Keys)
        {
            if (sb.Length > 0)
            {
                sb.Append("&");
            }
            sb.Append(HttpUtility.UrlEncode(k) + "=" + HttpUtility.UrlEncode(pars[k].ToString()));
        }
        return sb.ToString();
    }

    /// <summary>
    /// 获取Webservice响应报文XML
    /// </summary>
    /// <param name="response"></param>
    /// <returns></returns>
    private XmlDocument ReadXmlResponse(WebResponse response)
    {
        StreamReader sr = new StreamReader(response.GetResponseStream(), Encoding.UTF8);
        String retXml = sr.ReadToEnd();
        sr.Close();
        XmlDocument doc = new XmlDocument();
        doc.LoadXml(retXml);
        return doc;
    }

    /// <summary>
    /// 设置XML文档版本声明
    /// </summary>
    /// <param name="doc"></param>
    private void AddDelaration(XmlDocument doc)
    {
        XmlDeclaration decl = doc.CreateXmlDeclaration("1.0", "utf-8", null);
        doc.InsertBefore(decl, doc.DocumentElement);
    }
}

// 在 ConfigureServices 中配置：

// 远程请求
services.AddRemoteRequest(options =>
{
    // 配置内蒙古客户端
    options.AddHttpClient("clientName", c =>
    {
        /*其他配置*/
        c.Timeout = TimeSpan.FromSeconds(15);
    })
    .ConfigurePrimaryHttpMessageHandler(u => new WinHttpHandler()
    {
        ServerCertificateValidationCallback = (_, _, _, _) => true
    });
});
```

调用示例：

```cs
var serializerSettings = new JsonSerializerSettings
{
    // 设置为驼峰命名
    ContractResolver = new CamelCasePropertyNamesContractResolver()
};

Hashtable ht = new Hashtable();
ht.Add("certificate", certificate);
ht.Add("data", dataStr);

var rps = SoapHelper.QuerySoapWebService(url, "getData", ht, true);

//获取节点列表 
XmlNodeList topM = rps.SelectNodes("/root");
foreach (XmlElement element in topM)
{
    string text = element.InnerText;
    Console.WriteLine(text);

    var json = JsonConvert.DeserializeObject<ResponDto<List<BaseRps>>>(text,
        serializerSettings);
}
```
