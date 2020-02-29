---
title: C-Sharp字符串与字节互转
date: 2019-11-16 16:37:24
tags:
- DotNet
- CSharp
- CSharp基础
categories: 
- CSharp基础
---
## 十六进制

十六进制的表示:
C语言、Shell、Python语言及其他相近的语言使用字首“0x”，例如“0x5A3”。开头的“0”令解析器更易辨认数，而“x”则代表十六进制。在“0x”中的“x”可以大写或小写。

16进制就有16个数，0~15，用二进制表示15的方法就是1111，从而可以推断出，16进制用2进制可以表现成0000~1111，顾名思义，也就是每四个为一位。最高位不够可用零代替。

一个字节包含8个二进制位，一个十六进制可表示4个二进制位，所以，一个字节可以由2个十六进制表示。即，一个byte 对应两位十六进制位。

## 十六进制转换

### 字符串转为16进制字符

```cs
public static string StringToHexString(string s, Encoding encode)
{
    byte[] b = encode.GetBytes(s);//按照指定编码将string编程字节数组
    string result = string.Empty;
    for (int i = 0; i < b.Length; i++)//逐字节变为16进制字符
    {
        result += Convert.ToString(b[i], 16);
    }
    return result;
}

// 使用
System.Console.WriteLine(StringToHexString("严", System.Text.Encoding.UTF8));
System.Console.WriteLine(BitConverter.ToString(Encoding.UTF8.GetBytes("严")));
```

![QQ截图20191116174552.png](/img/QQ截图20191116174552.png)

### 16进制字符串转为字符串

```cs
public static string HexStringToString(string hs, Encoding encode)
{
    string strTemp = "";
    byte[] b = new byte[hs.Length / 2];
    for (int i = 0; i < hs.Length / 2; i++)
    {
        strTemp = hs.Substring(i * 2, 2);
        b[i] = Convert.ToByte(strTemp, 16);
    }
    //按照指定编码将字节数组变为字符串
    return encode.GetString(b);
}

// 使用
string hexstring = StringToHexString("严", System.Text.Encoding.UTF8);
string content = HexStringToString(hexstring, System.Text.Encoding.UTF8);
```

### byte[]转为16进制字符串

```cs
public static string ByteToHexStr(byte[] bytes)
{
    string returnStr = "";
    if (bytes != null)
    {
        for (int i = 0; i < bytes.Length; i++)
        {
            returnStr += bytes[i].ToString("X2");
        }
    }
    return returnStr;
}

// 使用

var byteStr = StringToHexString("严", Encoding.UTF8);
Console.WriteLine(byteStr);// 输出 e4b8a5

var bytes = Encoding.UTF8.GetBytes("严");
var str = Encoding.UTF8.GetString(bytes);// 输出 严

var str1 = ByteToHexStr(bytes); //  输出 E4B8A5

var bytes1 = StrToToHexByte(str1);
var bytes2 = StrToToHexByte(byteStr);
```

### 16进制的字符串转为byte[]

```cs
public static byte[] StrToToHexByte(string hexString)
{
    hexString = hexString.Replace(" ", "");
    if ((hexString.Length % 2) != 0)
        hexString += " ";
    byte[] returnBytes = new byte[hexString.Length / 2];
    for (int i = 0; i < returnBytes.Length; i++)
        returnBytes[i] = Convert.ToByte(hexString.Substring(i * 2, 2), 16);
    return returnBytes;
}
```

参考：

[C#数字、16进制字符串和字节之间互转](https://blog.csdn.net/WuLex/article/details/75452472)