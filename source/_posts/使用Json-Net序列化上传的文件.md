---
title: 使用Json.Net序列化上传的文件
date: 2019-05-31 13:47:09
tags:
- CSharp
- DotNet
- Json.Net
categories: 
- Json.Net
---
# 使用Json.Net序列化上传的文件

基本知识：

序列化： 将数据结构或对象转换成二进制串的过程。
反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程。

本文提供一种针对目标文件，重写文件序列化，从而实现上传。

客户端调用：

```cs
public void Send()
{
    var inFile1 =
        @"C:\Users\User1\source\repos\ConsoleApp1\Test.mp3";
    byte[] data = File.ReadAllBytes(inFile1);

    FileUpLoadDTO gdto = new FileUpLoadDTO();
	
	// 其它参数
    gdto.DeviceId = "1234321";
    gdto.DeviceMac = "1343212";

    var filedto = new FileDTO();
    filedto.FileData = data;
    filedto.FileName = GetFileName();
    filedto.FileSize = data.Length;
    gdto.FileDto = filedto;
	
	// 序列化文件
    var jsonData = JsonHelper.Serialize(new { dto = gdto });
    var url = "http://temp.uri/UpLoadFile";
    var response = HttpHelper.SendRequest(url, "POST", jsonData);
}

public string GetFileName()
{
    Random rd = new Random();
    StringBuilder serial = new StringBuilder();
    serial.Append(DateTime.Now.ToString("yyyyMMddHHmmssff"));
    serial.Append(rd.Next(0, 999999).ToString());
    return serial.ToString();
}
```

服务端实现：

```cs
public ReturnInfoDTO ReceiveFile(FileUpLoadDTO dto)
{
    ReturnInfoDTO rdto = new ReturnInfoDTO() { StatusCode = "200", IsSuccess = true, Message = "调用成功" };

    var outputFolder = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "NAudio");
    Directory.CreateDirectory(outputFolder);

    var filename = dto.FileDto.FileName;

    try
    {
        using (FileStream stream = File.Open(filename, FileMode.OpenOrCreate))
        {
            stream.Seek(0, SeekOrigin.End);
            stream.Write(dto.FileDto.FileData, 0, dto.FileDto.FileData.Length);
        }
    }
    catch (Exception e)
    {
        rdto.IsSuccess = false;
        rdto.StatusCode = "500";
        rdto.Message = "调用失败";
    }

    return rdto;
}
```

测试：

```cs
private void button9_Click(object sender, EventArgs e)
{
    var inFile1 =
        @"C:\Users\User1\Desktop\Test.mp3";
    byte[] data = File.ReadAllBytes(inFile1);

    FileUpLoadDTO gdto = new FileUpLoadDTO();

    // 其它参数
    gdto.DeviceId = "1234321";
    gdto.DeviceMac = "1343212";

    var filedto = new FileDto();
    filedto.FileData = data;
    filedto.FileName = GetFileName() + System.IO.Path.GetExtension(inFile1);
    filedto.FileSize = data.Length;
    gdto.FileDto = filedto;

    ReceiveFile(gdto);
}
```

JsonHelper帮助类：

```cs
public class JsonHelper
{
    public static T Deserialize<T>(string json)
    {
        return JsonConvert.DeserializeObject<T>(json, new JsonSerializerSettings()
        {
            ReferenceLoopHandling = ReferenceLoopHandling.Ignore
        });
    }

    public static string Serialize<T>(T data)
    {
        JsonSerializerSettings settings = new JsonSerializerSettings();
        ByteJsonConverter byteJsonConverter = new ByteJsonConverter();
        settings.ReferenceLoopHandling = ReferenceLoopHandling.Ignore;
        DateTimeConverter dateTimeConverter = new DateTimeConverter();
        settings.Converters.Add(dateTimeConverter);
        settings.Converters.Add(byteJsonConverter);
        return JsonConvert.SerializeObject((object)data, settings);
    }

    internal static string ObjectToJson<T>(T obj)
    {
        using (MemoryStream memoryStream = new MemoryStream())
        {
            new DataContractJsonSerializer(typeof(T)).WriteObject(memoryStream, obj);
            memoryStream.Position = 0L;
            using (StreamReader streamReader = new StreamReader(memoryStream, Encoding.UTF8))
                return streamReader.ReadToEnd();
        }
    }
}

internal class DateTimeConverter : JsonConverter
{
    public override bool CanRead
    {
        get { return false; }
    }

    public override bool CanWrite
    {
        get { return true; }
    }

    public override bool CanConvert(Type objectType)
    {
        return typeof(DateTime) == objectType || typeof(DateTime?) == objectType;
    }

    public override object ReadJson(
        JsonReader reader,
        Type objectType,
        object existingValue,
        JsonSerializer serializer)
    {
        throw new NotImplementedException();
    }

    public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
    {
        string json = JsonHelper.ObjectToJson<DateTime>(Convert.ToDateTime(value));
        writer.WriteRawValue(json);
    }
}

internal class ByteJsonConverter : JsonConverter
{
    public override bool CanRead
    {
        get { return false; }
    }

    public override bool CanWrite
    {
        get { return true; }
    }

    public override bool CanConvert(Type objectType)
    {
        return typeof(byte[]) == objectType;
    }

    public override object ReadJson(
        JsonReader reader,
        Type objectType,
        object existingValue,
        JsonSerializer serializer)
    {
        throw new NotImplementedException();
    }

    public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
    {
        string json = JsonHelper.ObjectToJson<byte[]>(value as byte[]);
        writer.WriteRawValue(json);
    }
}
```

参考：

[system.io.filestream](https://docs.microsoft.com/en-us/dotnet/api/system.io.filestream?view=netframework-4.8)

[Newtonsoft.Json](https://github.com/JamesNK/Newtonsoft.Json)