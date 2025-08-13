---
title: 使用Naudio将Pcm转G711
date: 2019-05-29 11:43:21
tags:
- 音频/视频
- CSharp
- DotNet
categories: 
- Naudio
---
# 使用Naudio将Pcm转G711

我们在音频处理的时候经常会接触到PCM数据：它是模拟音频信号经模数转换（A/D变换）直接形成的二进制序列，该文件没有附加的文件头和文件结束标志。

声音本身是模拟信号，而计算机只能识别数字信号，要在计算机中处理声音，就需要将声音数字化，这个过程叫经模数转换（A/D变换）。最常见的方式是通过<font color=#ff0000 size=4 face="黑体">脉冲编码调制PCM(Pulse Code Modulation)</font> 。

下面是从github上[Naudio库](https://github.com/naudio/NAudio/blob/master/Docs/RecordWavFileWinFormsWaveIn.md)拷贝的 `WinForm` 录音代码：

```cs
var outputFolder = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), "NAudio");
Directory.CreateDirectory(outputFolder);
var outputFilePath = Path.Combine(outputFolder,"recorded.wav");

var waveIn = new WaveInEvent();

WaveFileWriter writer = null;
MuLawChatCodec _codec = new MuLawChatCodec();
bool closing = false;
var f = new Form();
var buttonRecord = new Button() { Text = "Record" };
var buttonStop = new Button() { Text = "Stop", Left = buttonRecord.Right, Enabled = false };
f.Controls.AddRange(new Control[] { buttonRecord, buttonStop });

buttonRecord.Click += (s, a) => 
{ 
    writer = new WaveFileWriter(outputFilePath, waveIn.WaveFormat); 
    waveIn.StartRecording(); 
    buttonRecord.Enabled = false; 
    buttonStop.Enabled = true; 
};

buttonStop.Click += (s, a) => waveIn.StopRecording();

waveIn.DataAvailable += (s, a) =>
{
    // Pcm
    // writer.Write(a.Buffer, 0, a.BytesRecorded);

    // Pcm to G.711
    byte[] encoded = _codec.Encode(a.Buffer, 0, a.BytesRecorded);
    writer.Write(encoded, 0, encoded.Length);

    if (writer.Position > waveIn.WaveFormat.AverageBytesPerSecond * 30)
    {
        waveIn.StopRecording();
    }
};

waveIn.RecordingStopped += (s, a) =>
{
    writer?.Dispose(); 
    writer = null; 
    buttonRecord.Enabled = true;
    buttonStop.Enabled = false;
    if (closing) 
    { 
        waveIn.Dispose();
    }
};

f.FormClosing += (s, a) => { closing=true; waveIn.StopRecording(); };
f.ShowDialog();
```

转G.711 mu-law 编码代码：

引用：[MuLawChatCodec.cs](https://github.com/naudio/NAudio/blob/master/NAudioDemo/NetworkChatDemo/MuLawChatCodec.cs)

```cs
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

    public bool IsAvailable { get { return true; } }
}
```