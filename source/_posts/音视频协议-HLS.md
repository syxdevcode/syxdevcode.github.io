---
title: 音视频协议-HLS
date: 2023-04-03 18:10:59
tags:
  - 音视频
  - 音频
  - 视频
  - HLS
categories:
  - 音视频协议
---

## HLS简介

HLS 全称是 HTTP Live Streaming，是一个由 Apple 公司提出的基于 HTTP 的媒体流传输协议。是苹果公司QuickTime X和iPhone软件系统的一部分。 它的工作原理是把整个流分成一个个小的基于HTTP的文件来下载，每次只下载一些。当媒体流正在播放时，客户端可以选择从许多不同的备用源中以不同的速率下载同样的资源，允许流媒体会话适应不同的数据速率。目前HLS协议被广泛的应用于视频点播和直播领域。

在 HTML5 页面上使用 HLS 非常简单：

直接：

```html
<video src="example.m3u8" controls></video>
```

或者：

```html
<video controls>
    <source src="example.m3u8"></source>
</video>
```

<!--more-->
在开始一个流媒体会话时，客户端会下载一个包含元数据的 `extended M3U (m3u8) playlist`文件，用于寻找可用的媒体流。HLS 只请求基本的HTTP报文，与实时传输协议（RTP)不同，HLS可以穿过任何允许HTTP数据通过的防火墙或者代理服务器。​它也很容易使用内容分发网络来传输媒体流。

RTMP指Adobe的RTMP(`Realtime Message Protocol`)，广泛应用于低延时直播，也是编码器和服务器对接的实际标准协议，在PC（Flash）上有最佳观看体验和最佳稳定性。

HLS指Apple的HLS(`Http Live Streaming`)，本身就是Live（直播）的，不过Vod（点播）也能支持。HLS是Apple平台的标准流媒体协议，和RTMP在PC上一样支持得天衣无缝。

## HLS协议详解

### HLS协议规定

* 视频的封装格式是TS。
* 视频的编码格式为H264,音频编码格式为MP3、AAC或者AC-3。
* 除了TS视频文件本身，还定义了用来控制播放的m3u8文件（文本文件）。

HLS的m3u8，是一个ts的列表，也就是告诉浏览器可以播放这些ts文件。

### 整体架构图

HLS 整体架构图总共有三个部分：Server，CDN，Client。

![v2-561df18ad78a909fa18570bc4a6ec2ad_720w.webp](/img1/v2-561df18ad78a909fa18570bc4a6ec2ad_720w.webp)

HLS 通过 URI(RFC3986) 指向的一个 `Playlist` 来表示一个媒体流。
一个 `Playlist` 可以是一个 Media `Playlist` 或者 `Master Playlist`，使用 UTF-8 编码的文本文件，包含一些 URI 跟描述性的 tags。
一个 `Media Playlist` 包含一个 `Media Segments` 列表,当顺序播放时，能播放整个完整的流。
要想播放这个 `Playlist`，客户端需要首先下载他，然后播放里面的每一个 `Media Segment`。
更加复杂的情况是，`Playlist` 是一个 `Master Playlist`，包含一个 `Variant Stream` 集合，通常每个 `Variant Stream` 里面是同一个流的多个不同版本(如：分辨率，码率不同)。

### HLS Media Segments

* 每一个 `Media Segment` 通过一个 URI 指定, 可能包含一个 byte range。
* 每一个 `Media Segment` 的 duration 通过 `EXTINF` tag 指定。
* 每一个 `Media Segment` 有一个唯一的整数 `Media Segment Number`。
* 有些媒体格式需要一个 `format-specific sequence` 来初始化一个 parser，在 `Media Segment` 被 parse 之前。这个字段叫做 `Media Initialization Section`，通过 EXT-X-MAP tag 来指定。

## 支持的 Media Segment 格式

### MPEG-2 Transport Streams

* 最常见的 TS 文件。
* RFC: ISO_13818。
* `Media Initialization Section`: `PAT(Program Association Table)` 跟 `PMT(Program Map Table)`。
* 每个 `TS segment` 必须值含一个 `MPEG-2 Program`。
* 每一个 `TS segment` 包含一个 PAT 和 PMT, 最好在 segment 的开始处, 或者通过一个 `EXT-X-MAP` tag 来指定。

### Fragmented MPEG-4

* 即常提到的 fMP4.
* RFC: ISOBMFF.
* `Media Initialization Section`: ftyp box(包含一个高于 ios6 的 brand), ftypbox 必须紧跟在 moovbox 之后. moov box 必须包含一个 trak box(对于每个 `fMP4 segment` 里面的 traf box, 包含匹配的 track_ID). 每个 trakbox 应该包含一个 sample table, 但是他的 sample count 必须为 0. mvhdbox 跟 tkhd 的 duration 必须为 0. mvex box 必须跟在上一个 trak box 后面.
* 不像普通的 MP4 文件包含一个 moov box(包含 sample tables) 和一个 mdatbox(包含对应的 samples), 一个 fMP4 包含一个 moof box (包含 sample table 的子集), 和一个 mdat box(包含对应的 samples).
* 在每一个 `fMP4 segment` 里面, 每一个 traf box 必须包含一个 tfdt box, `fMP4 segment` 必须使用 movie-fragment relative addressing. `fMP4 segments` 绝对不能使用外部的 data references.
* 每一个 `fMP4 segment` 必须有一个 `EXT-X-MAP` tag.

### Packed Audio

* 一个 `Packed Audio Segment` 包含编码的 audio samples 和 ID3 tags. 简单的打包到一起, 包含最小的 framing, 并且没有 per-sample timestamp.
* 支持的 `Packed Audio`: AAC with ADTS framing [ISO_13818_7], MP3 [ISO_13818_3], AC-3 [AC_3], Enhanced AC-3 [AC_3].
* 一个 `Packed Audio Segment` 没有 `Media Initialization Section`.
* 每一个 `Packed Audio Segment` 必须在他的第一个 sample 指定 timestamp 通过一个 `ID3 PRIV` tag.
* `ID3 PRIV owner identifier` 必须是 com.apple.streaming.transportStreamTimestamp.
* `ID3 payload` 必须是一个 33-bit MPEG-2 Program Elementary Stream timestamp 的大端 eight-octet number, 高 31 为设置为 0.

### WebVTT

* 一个 `WebVTT Segment` 是一个 WebVTT 文件的一个 section, `WebVTT Segment` 包含 subtitles.
* Media Initialization Section: WebVTT header.
* 每一个 `WebVTT Segment` 必须有以一个 WebVTT header 开始, 或者有一个 EXT-X-MAP tag 来指定.
* 每一个 `WebVTT header` 应该有一个 X-TIMESTAMP-MAP 来保证音视频同步.

### HLS Playlists

* `Playlist` 文件的格式是起源于 M3U, 并且继承两个 tag: `EXTM3U` 和 `EXTINF`
* 下面的 tags 通过 `BNF-style` 语法来指定.
* 一个 `Playlist` 文件必须通过 URI(`.m3u8` 或 `m3u`) 或者 `HTTP Content-Type` 来识别(`application/vnd.apple.mpegurl` 或 `audio/mpegurl`).
* 换行符可以用 `\n` 或者 `\r\n`.
* 以 `#` 开头的是 tag 或者注释, 以 `#EXT` 开头的是 tag, 其余的为注释, 在解析时应该忽略.
* `Playlist` 里面的 URI 可以用绝对地址或者相对地址, 如果使用相对地址, 那么是相对于 `Playlist` 文件的地址.

### Attribute Lists

* 有的 tags 的值是 `Attribute Lists`.
* 一个 `Attribute List` 是一个用逗号分隔的 `attribute/value` 对列表.
* 格式为: `AttributeName=AttributeValue`.

### Basic Tags

`Basic Tags` 可以用在 `Media Playlist` 和 `Master Playlist` 里面.

* `EXTM3U`: 必须在文件的第一行, 标识是一个 `Extended M3U Playlist` 文件.
* `EXT-X-VERSION`: 表示 `Playlist` 兼容的版本.

### Media Segment Tags

每一个 `Media Segment` 通过一系列的 `Media Segment` tags 跟一个 URI 来指定. 有的 `Media Segment` tags 只应用与下一个 segment, 有的则是应用所有下面的 segments. 一个 `Media Segment` tag 只能出现在 `Media Playlist` 里面.

* `EXTINF`: 用于指定 `Media Segment` 的 duration
* `EXT-X-BYTERANGE`: 用于指定 URI 的 sub-range
* `EXT-X-DISCONTINUITY`: 表示不连续.
* `EXT-X-KEY`: 表示 `Media Segment` 已加密, 该值用于解密.
* `EXT-X-MAP`: 用于指定 Media Initialization Section.
* `EXT-X-PROGRAM-DATE-TIME`: 和 `Media Segment` 的第一个 sample 一起来确定时间戳.
* `EXT-X-DATERANGE`: 将一个时间范围和一组属性键值对结合到一起.

### Media Playlist Tags

`Media Playlist` tags 描述 `Media Playlist` 的全局参数. 同样地, `Media Playlist` tags 只能出现在 `Media Playlist` 里面.

* `EXT-X-TARGETDURATION`: 用于指定最大的 `Media Segment` duration.
* `EXT-X-MEDIA-SEQUENCE`: 用于指定第一个 `Media Segment` 的 Media Sequence Number.
* `EXT-X-DISCONTINUITY-SEQUENCE`: 用于不同 Variant Stream 之间同步.
* `EXT-X-ENDLIST`: 表示结束.
* `EXT-X-PLAYLIST-TYPE`: 可选, 指定整个 Playlist 的类型.
* `EXT-X-I-FRAMES-ONLY`: 表示每个 `Media Segment` 描述一个单一的 I-frame.

示例：

```txt
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:6
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-KEY:METHOD=AES-128,URI="/20220219/tWWHAcVm/1200kb/hls/key.key"
#EXTINF:3.103,
/20220219/tWWHAcVm/1200kb/hls/kmZoIJ8h.ts
```

### Master Playlist Tags

`Master Playlist` tags 定义 `Variant Streams`, Renditions 和 其他显示的全局参数. `Master Playlist` tags 只能出现在 `Master Playlist` 中.

* `EXT-X-MEDIA`: 用于关联同一个内容的多个 `Media Playlist` 的多种 renditions.
* `EXT-X-STREAM-INF`: 用于指定一个 `Variant Stream`.
* `EXT-X-I-FRAME-STREAM-INF`: 用于指定一个 `Media Playlist` 包含媒体的 I-frames.
* `EXT-X-SESSION-DATA`: 存放一些 session 数据.
* `EXT-X-SESSION-KEY`: 用于解密.

示例：

```txt
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=1200000,RESOLUTION=1280x720
/20220219/tWWHAcVm/1200kb/hls/index.m3u8
```

### Media or Master Playlist Tags

这里的 tags 可以出现在 `Media Playlist` 或者 `Master Playlist` 中. 但是如果同时出现在同一个 `Master Playlist` 和 `Media Playlist` 中时, 必须为相同值.

* `EXT-X-INDEPENDENT-SEGMENTS`: 表示每个 `Media Segment` 可以独立解码.
* `EXT-X-START`: 标识一个优选的点来播放这个 Playlist.

## 服务器端与客户端逻辑

以下流程仅供参考, 其实不同的播放器客户端以及服务器端的拉取规则都有很多细节差异.

### 服务器端逻辑

1，将媒体源切片成 `Media Segment`, 应该优先从可以高效解码的时间点来进行切片(如: I-frame).
2，为每一个 `Media Segment` 生成 URI.
3，Server 需要支持 “gzip” 方式压缩文本内容.
4，创建一个 `Media Playlist` 索引文件, `EXT-X-VERSION` 不要高于他需要的版本, 来提供更好的兼容性.
5，Server 不能随便修改 `Media Playlist`, 除了 Append 文本到文件末尾, 按顺序移除 `Media Segment` URIs, 增长 `EXT-X-MEDIA-SEQUENCE` 和 `EXT-X-DISCONTINUITY-SEQUENCE`, 添加 `EXT-X-ENDLIST` 到文件尾.
6，在最后添加 `EXT-X-ENDLIST` tag, 来减少 Client reload Playlist 的次数.
7，注意点播与直播服务器不同的地方是, 直播的 m3u8 文件会不断更新, 而点播的 m3u8 文件是不会变的, 只需要客户端在开始时请求一次即可.

### 客户端逻辑

1，客户端通过 URI 获取 Playlist. 如果是 Master Playlist, 客户端可以选择一个 Variant Stream 来播放.
2，客户端检查 `EXT-X-VERSION` 版本是否满足.
3，客户端应该忽略不可识别的 tags, 忽略不可识别的属性键值对.
4，加载 `Media Playlist` file.
5，播放 `Media Playlist` file.
6，重加载 `Media Playlist` file.
7，决定下一次要加载的 `Media Segment`.

参考：

[HLS 直播协议m3u8详解](https://juejin.cn/post/6976075491656761358)

[音视频开发学习：HLS 协议详解](https://zhuanlan.zhihu.com/p/460011665)
