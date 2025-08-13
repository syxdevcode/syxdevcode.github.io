---
title: 使用NAudio将MP3编码转G711
date: 2019-05-30 17:19:45
tags:
- 音频/视频
- CSharp
- DotNet
categories: 
- Naudio
---
# 使用NAudio将MP3编码转G711

将MP3编码转成G711编码，以支持终端播放需求。

```cs
var inFile1 =
    @"C:\Users\test\source\repos\ConsoleApp1\7794FDBE-34E9-4218-B5EC-2272ED750201.mp3";
byte[] data = File.ReadAllBytes(inFile1);

var outputFolder = Path.Combine(System.Environment.CurrentDirectory, "NAudio");
Directory.CreateDirectory(outputFolder);

var filename = GetFileName();
string g711FileName = filename + "-g711.wav";

var inFile = Path.Combine(outputFolder, filename + ".wav");
var outfile = Path.Combine(outputFolder, g711FileName);

using (MemoryStream ms = new MemoryStream(data))
using (var reader = new Mp3FileReader(ms))
{
    WaveFileWriter.CreateWaveFile(inFile, reader);
}

NAudioHelper.EnCodeG711(inFile, outfile);
```

```cs
/// <summary>
/// Naudio 转码
/// </summary>
public class NAudioHelper
{
    private static MuLawChatCodec _codec = new MuLawChatCodec();
    public static void EnCodeG711(string inFile, string outfile)
    {
        var format = new WaveFormat(8000, 16, 1);
        using (var reader = new WaveFileReader(inFile))
        using (WaveFileWriter waveFileWriter = new WaveFileWriter(outfile, format))
        {
            byte[] buffer = new byte[reader.WaveFormat.AverageBytesPerSecond * 4];
            while (true)
            {
                int count = reader.Read(buffer, 0, buffer.Length);

                byte[] encoded = _codec.Encode(buffer, 0, count);

                if (count != 0)
                    waveFileWriter.Write(encoded, 0, encoded.Length);
                else
                    break;
            }
        }
    }
    public static string GetFileName()
    {
        Random rd = new Random();
        StringBuilder serial = new StringBuilder();
        serial.Append(DateTime.Now.ToString("yyyyMMddHHmmssff"));
        serial.Append(rd.Next(0, 999999).ToString());
        return serial.ToString();
    }
}

internal class MuLawChatCodec
{
    public string Name => "G.711 mu-law";

    public int BitsPerSecond => RecordFormat.SampleRate * 8;

    public WaveFormat RecordFormat => new WaveFormat(8000, 16, 1);

    public byte[] Encode(byte[] data, int offset, int length)
    {
        var encoded = new byte[length / 2];
        int outIndex = 0;
        for (int n = 0; n < length; n += 2)
        {
            encoded[outIndex++] = MuLawEncoder.LinearToMuLawSample(BitConverter.ToInt16(data, offset + n));
        }

        return encoded;
    }

    public byte[] Decode(byte[] data, int offset, int length)
    {
        var decoded = new byte[length * 2];
        int outIndex = 0;
        for (int n = 0; n < length; n++)
        {
            short decodedSample = MuLawDecoder.MuLawToLinearSample(data[n + offset]);
            decoded[outIndex++] = (byte)(decodedSample & 0xFF);
            decoded[outIndex++] = (byte)(decodedSample >> 8);
        }

        return decoded;
    }

    public void Dispose()
    {
        // nothing to do
    }

    public bool IsAvailable
    {
        get { return true; }
    }
}
```