---
title: CSharp-form-data发送类
date: 2021-07-02 17:37:17
tags:
- CSharp
- DotNet
- CSharp基础
categories: 
- CSharp基础
---

```c#
public class HttpClientHelper
{
    private static string zhongtaiUrl = ConfigurationManager.AppSettings["ZhongTaiUrl"];

    /// <summary>
    /// 用multipart/form-data发送
    /// </summary>
    /// <param name="postParaList"></param>
    /// <returns></returns>
    public static string PostMessage(List<PostDataClass> postParaList)
    {
        try
        {
            string responseContent = "";
            var memStream = new MemoryStream();
            var webRequest = (HttpWebRequest)WebRequest.Create(zhongtaiUrl);
            // 边界符
            var boundary = "---------------" + DateTime.Now.Ticks.ToString("X");
            // 边界符
            var beginBoundary = Encoding.ASCII.GetBytes("--" + boundary + "\r\n");
            // 最后的结束符
            var endBoundary = Encoding.ASCII.GetBytes("--" + boundary + "--\r\n");
            memStream.Write(beginBoundary, 0, beginBoundary.Length);
            // 设置属性
            webRequest.Method = "POST";
            webRequest.Timeout = 10000;
            webRequest.ContentType = "multipart/form-data; boundary=" + boundary;
            for (int i = 0; i < postParaList.Count; i++)
            {
                PostDataClass temp = postParaList[i];
                // 写入字符串的Key
                var stringKeyHeader = "Content-Disposition: form-data; name=\"{0}\"" +
                               "\r\n\r\n{1}\r\n";
                var header = string.Format(stringKeyHeader, temp.Prop, temp.Value);
                var headerbytes = Encoding.UTF8.GetBytes(header);
                memStream.Write(headerbytes, 0, headerbytes.Length);

                if (i != postParaList.Count - 1)
                    memStream.Write(beginBoundary, 0, beginBoundary.Length);
                else
                    // 写入最后的结束边界符
                    memStream.Write(endBoundary, 0, endBoundary.Length);
            }

            byte[] b = memStream.ToArray();
            string param = System.Text.Encoding.UTF8.GetString(b, 0, b.Length);
            LogHelper.Info(string.Format("HttpClientHelper ==> PostMessage参数：{0}", param));

            webRequest.ContentLength = memStream.Length;
            var requestStream = webRequest.GetRequestStream();
            memStream.Position = 0;
            var tempBuffer = new byte[memStream.Length];
            memStream.Read(tempBuffer, 0, tempBuffer.Length);
            memStream.Close();
            requestStream.Write(tempBuffer, 0, tempBuffer.Length);
            requestStream.Close();
            using (HttpWebResponse res = (HttpWebResponse)webRequest.GetResponse())
            {
                using (Stream resStream = res.GetResponseStream())
                {
                    byte[] buffer = new byte[1024];
                    int read;
                    while ((read = resStream.Read(buffer, 0, buffer.Length)) > 0)
                    {
                        responseContent += Encoding.UTF8.GetString(buffer, 0, read);
                    }
                }
                res.Close();
            }

            LogHelper.Info(string.Format("HttpClientHelper ==> PostMessage返回值：{0}", responseContent));
            return responseContent;
        }
        catch (Exception e)
        {
            LogHelper.Error(string.Format("HttpClientHelper ==> PostMessage方法======>Message：{0}；StackTrace：{1}；Source：{2}；", e.Message, e.StackTrace, e.Source));
            throw;
        }
    }
}

public class PostDataClass
{
    public string Prop { get; set; }
    public string Value { get; set; }
}
```