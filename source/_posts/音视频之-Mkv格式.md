---
title: 音视频之-Mkv格式
date: 2021-09-22 15:39:49
tags:
- 音频/视频
categories: 
- 音频/视频
---

[mkv](https://baike.baidu.com/item/MKV)

## 简介

&emsp;&emsp;Matroska多媒体容器（Multimedia Container）是一种开放标准的自由的容器和文件格式，是一种多媒体封装格式，能够在一个文件中容纳无限数量的视频、音频、图片或字幕轨道。所以其不是一种压缩格式，而是Matroska定义的一种多媒体容器文件。其目标是作为一种统一格式保存常见的电影、电视节目等多媒体内容。在概念上Matroska和其他容器，比如AVI、MP4或ASF（Advanced Streaming Format，即高级流格式）比较类似，但其在技术规程上完全开放，在实现上包含很多开源软件。可将多种不同编码的视频及16条以上不同格式的音频和不同语言的字幕流封装到一个Matroska 媒体文件当中。最大的特点就是能容纳多种不同类型编码的视频、音频及字幕流。

&emsp;&emsp;mkv不同于DivX、XviD等视频编码格式，也不同于MP3、Ogg等音频编码格式。MKV是为这些音、视频提供外壳的“组合”和“封装”格式。换句话说就是一种容器格式，常见的 DAT(是VCD的一种编码格式)AVl、VOB、MPEG、RM 格式其实也都属于这种类型。但它们要么结构陈旧，要么不够开放，这才促成了MKV这类新型多媒体封装格式的诞生。

&emsp;&emsp;Matroska的文件扩展名，对于携带了音频、字幕的视频文件是.MKV；对于3D立体影像视频是.MK3D；对于单一的纯音频文件是.MKA；对于单一的纯字幕文件是.MKS。

<!--more-->
## MKV特点

&emsp;&emsp;Matroska最大的特点就是能容纳多种不同类型编码的视频、音频及字幕流，甚至囊括了RealMedia及QuickTime这类流媒体，可以说是对传统媒体封装格式的一次大颠覆！它现在几乎变成了一个万能的媒体容器，目前它所能封装的视频、音频、字幕类型包括：

* AVI文件，包括采用DivX、XviD、3ivX、VP6视频编码，及PCM、MP3、AC3等音频编码的AVI
* RealMedia文件，包括RealVideo和RealAudio，QuickTime的MOV及MP4视频
* Windows Media文件，包括ASF、WMV格式
* MPEG文件，包括MPEG-1/2的M1V、M2V
* Ogg/OGM 文件，包括Ogg Vorbis、OGM、FLAC文件
* Matroska Media文件，包括MKV、MKA、MKS文件
* WAV、AC3、DTS、MP2、MP3、AAC/MP4音频
* SRT、USF及SSA/ASS文本字幕
* SubVob图形字幕，后缀为IDX、SUB
* BMP图形字幕，以一组BMP图片及时间码构成的字幕 。

&emsp;&emsp;此外，Matroska文件中还可包括章节、标签（Tag）等信息，甚至还可加上附件！需要指出的Matroska所谓的封装AVI、RM、MOV等媒体，但它并不是简单将它们不加改变的合并到Matroska中，而是将它们的音视频流进行了重新组织。

&emsp;&emsp;Matroska加入AVI所没有的EDC错误检测代码，这意味着即使是没有下载完毕的MKV文件也可以顺利回放，这些对AVI来说完全是不可想象的。虽然Matroska加入了错误检测代码，但由于采用了新的更高效的组织结构，用MKV封装后的电影还是比AVI源文件要小了约1%，这就是说即使加上了多个字幕，MKV文件的体积也不可能比AVI文件大。

&emsp;&emsp;Matroska支持可变帧率（VFR，即Variable Frame Rate）的视频编码，这种VFR视频的帧率是不固定的，它可在动态画面中使用较大的帧率，而在静态画面中使用较小的帧率，这样可以有效的减少视频文件的体积，并改善动态画面的质量。它的作用比目前广泛使用的VBR（可变码率）更为明显。

&emsp;&emsp;看看目前比较流行的多媒体容器类型，例如AVI，它可以容纳多种类型的视频编码和音频编码，像VP6、DivX、XviD等视频编码和PCM、MP3、AC3等音频编码； VOB则是另一种特点更为鲜明的媒体容器，它可容纳MPEG-2视频流、多个AC3、 DTS、THX、PCM音频流、多个不同语言的图形字幕流。
