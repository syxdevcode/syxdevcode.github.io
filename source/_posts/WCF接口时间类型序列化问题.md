---
title: WCF接口时间类型序列化问题
date: 2019-04-25 13:30:37
tags:
- DotNet
- WCF
- Json.Net
- 时间/时间戳转换
categories: 
- WCF时间类型序列化
---
# WCF接口时间类型序列化问题

使用 `Json.Net` 序列化或者反序列化 `DateTime` 类型时，通过设置 `JsonSerializerSettings` 对象，即可支持 WCF 时间类型，示例如下：

```cs
public class WCFModel
{
    public DateTime CreatedDate { get; set; }
}

private static void WCFDateTimeConvert()
{
    WCFModel p = new WCFModel { CreatedDate = TimeZone.CurrentTimeZone.ToLocalTime(DateTime.UtcNow) };

    var settings = new JsonSerializerSettings
    {
        DateFormatHandling = DateFormatHandling.MicrosoftDateFormat
    };

    string json = JsonConvert.SerializeObject(p, settings);

    var dObj = JsonConvert.DeserializeObject<WCFModel>(json, settings);
}
```

具体实现源码路径 `Newtonsoft.Json\Utilities` 第635行：

Json.Net 源码地址：[https://github.com/JamesNK/Newtonsoft.Json/blob/master/Src/Newtonsoft.Json/Utilities/DateTimeUtils.cs](https://github.com/JamesNK/Newtonsoft.Json/blob/master/Src/Newtonsoft.Json/Utilities/DateTimeUtils.cs)

```cs
internal static int WriteDateTimeString(char[] chars, int start, DateTime value, TimeSpan? offset, DateTimeKind kind, DateFormatHandling format)
{
    int pos = start;

    if (format == DateFormatHandling.MicrosoftDateFormat)
    {
        TimeSpan o = offset ?? value.GetUtcOffset();

        long javaScriptTicks = ConvertDateTimeToJavaScriptTicks(value, o);

        @"\/Date(".CopyTo(0, chars, pos, 7);
        pos += 7;

        string ticksText = javaScriptTicks.ToString(CultureInfo.InvariantCulture);
        ticksText.CopyTo(0, chars, pos, ticksText.Length);
        pos += ticksText.Length;

        switch (kind)
        {
            case DateTimeKind.Unspecified:
                if (value != DateTime.MaxValue && value != DateTime.MinValue)
                {
                    pos = WriteDateTimeOffset(chars, pos, o, format);
                }
                break;
            case DateTimeKind.Local:
                pos = WriteDateTimeOffset(chars, pos, o, format);
                break;
        }

        @")\/".CopyTo(0, chars, pos, 3);
        pos += 3;
    }
    else
    {
        pos = WriteDefaultIsoDate(chars, pos, value);

        switch (kind)
        {
            case DateTimeKind.Local:
                pos = WriteDateTimeOffset(chars, pos, offset ?? value.GetUtcOffset(), format);
                break;
            case DateTimeKind.Utc:
                chars[pos++] = 'Z';
                break;
        }
    }

    return pos;
}
```