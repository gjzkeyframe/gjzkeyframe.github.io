---
title: 可视化音视频分析工具
description: 介绍常用的音画原始数据分析工具、编码数据分析工具、封装格式分析工具。
author: Keyframe
date: 2025-02-22 10:08:08 +0800
categories: [音视频工具]
tags: [音视频工具, 音视频, Adobe Audition, YUVToolkit, YUVView, StreamEye, MP4Box.js, MediaParser, MediaInfo, FLVParser, VLC]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }

（本文基本逻辑：音画原始数据分析工具介绍 → 编码数据分析工具介绍 → 封装格式分析工具介绍）


工欲善其事，必先利其器。在音视频开发中，为了**方便、快捷、直观的分析音视频数据**，最好能有一些可视化的分析工具来帮助我们，这篇文章就来介绍一下常见的可视化音视频分析工具。


## 1、音画原始数据分析工具


### 1.1、Adobe Audition

[Adobe Audition](https://www.adobe.com/products/audition.html "Adobe Audition") 是由 Adobe 公司开发的一个专业音频编辑和处理工具，支持多音轨、多种音频特效、多种音频格式。用它来分析 PCM、AAC 等格式的音频数据自然不在话下。


由于 PCM 数据是裸的音频数据，不包含数字音频要素属性信息，所以在打开 PCM 数据文件时，如下图所示，需要指定对应的采样率、声道数、位深、字节序等信息才能正确打开。

![Adobe Audition](assets/resource/av-tool/audition-1.png)
_Adobe Audition_

下图是打开示例 PCM 音频数据后的界面，可以看到对应的双声道波形图：

![Adobe Audition](assets/resource/av-tool/audition-2.png)
_Adobe Audition_


Adobe Audition 有非常丰富的功能，网上有很多专业介绍的信息，我们在这里不做过多介绍。



### 1.2、YUVToolkit


[YUVToolkit](https://github.com/svn2github/yuvtoolkit "YUVToolkit") 是一个开源跨平台的用于播放和分析原生 YUV 数据的工具。它有这些功能：


- 支持大部分 YUV 格式和 RGB 格式。比如：I420、I422、I444、YV12、YV16、YV24、UYVY、YUY2、NV12、grayscale；RGB24、RGBX32、XRGB32。
- 支持从文件名解析图像分辨率、帧率、颜色模型。比如：文件名为 `test-640x480-30FPS-I420.yuv`。
- 使用 Direct3D 和 OpenGL 渲染，最高可支持 720P、60FPS、4 个视频同时渲染。
- 支持对比图像并逐帧计算 MSE 和 PSNR，并可视化的展示失真情况。
- 支持使用 Javascript。比如：可以用脚本一次性打开多个文件。
- 可以用插件扩展来支持更多的视频格式、质量评估方式、渲染引擎。


下图是播放两份 YUV 数据，并对比计算 MSE 和 PSNR：

![YUVToolkit](assets/resource/av-tool/yuvtoolkit-1.png)
_YUVToolkit_



### 1.3、YUVView


[YUVView](https://github.com/IENT/YUView "YUVView") 是一个基于 QT 开发的开源跨平台的 YUV 数据播放和分析工具。它有如下功能：



- 支持大部分的 YUV 采样格式。比如：4:4:4、4:2:2、4:2:0、4:4:0、4:1:0、4:1:1、4:0:0。
- 支持位深 8-16 bit。
- 支持 ITU-R.BT709、ITU-R.BT601、ITU-R.BT2020 颜色空间转换。
- 色度插值使用最近邻插值或双线性插值。
- 可自由配置色度位置和 UV plane 顺序。
- 支持紧缩式的 YUV 存储格式。
- 支持大部分 RGB 格式。
- 支持 H.265（HEVC）文件。
- 支持对视频文件生成分析数据并浮层展示。
- 支持对比分析不同文件的差异。



下图展示了 YUVView 的功能界面：

![YUVView](assets/resource/av-tool/yuvview-1.png)
_YUVView_


下图是在一个 HEVC 码流上显示 Luma Intra Direction：

![YUVView: Overlay Statistics](assets/resource/av-tool/yuvview-2.png)
_YUVView: Overlay Statistics_

下图展示了如何对比编码数据和原始数据之间的差异：

![YUVView: Inspecting Differences](assets/resource/av-tool/yuvview-3-min.gif)
_YUVView: Inspecting Differences_

更多的信息参见：[YUVView Introduction](http://ient.github.io/YUView/ "YUVView Introduction")





## 2、编码数据分析工具


### 2.1、StreamEye

[StreamEye](https://www.elecard.com/zh/products/video-analysis/streameye "StreamEye") 是一款商业的媒体分析软件。以下是它的部分功能：

- 提供了码流视图界面、HEX 视图界面、像素视图界面、信息视图界面等可视化界面。
- 支持参考文件、图像差异对比、主从控制模式。
- 可以查看和分析视频码流信息、图像帧信息、块信息、标志位信息、DPB 信息等众多数据。
- 支持 H.264、H.265、VP9、AV1、VVC 等编码格式。


下图是使用条形图导航，解码图像缓冲区（DPB）的可视化，以及块的详细信息和表示：


![StreamEye：解码图像缓冲区及块信息](assets/resource/av-tool/streameye-1.jpg)
_StreamEye：解码图像缓冲区及块信息_



下图是使用缩略图、分区和运动矢量进行导航，演示编码语法结构：

![StreamEye：演示语法结构](assets/resource/av-tool/streameye-2.jpg)
_StreamEye：演示语法结构_


下图是 SAO 滤波叠加，缓冲区分析以及像素级别的可视化：

![StreamEye：像素级别可视化](assets/resource/av-tool/streameye-3.jpg)
_StreamEye：像素级别可视化_




下图是 ALF 滤波叠加，图像概述演示：

![StreamEye：图像概述演示](assets/resource/av-tool/streameye-4.jpg)
_StreamEye：图像概述演示_





## 3、封装格式分析工具


### 3.1、MP4Box.js

[MP4Box.js](https://gpac.github.io/mp4box.js/test/filereader.html "MP4Box.js") 是一个在线的 MP4 格式分析工具。它支持导入网络视频和本地视频，并可以直观的展示 MP4 Box 的结构以及数据。

下图是用树形结构展示 MP4 Box：

![MP4Box.js](assets/resource/av-tool/mp4box-1.png)
_MP4Box.js_



<!-- sync -->

### 3.2、MediaParser

[MediaParser](https://github.com/ksvc/MediaParser "MediaParser") 是一个开源的 MP4 格式分析器，功能比较简单，支持按树形结构展示 MP4 Box 及节点数据信息，也可以按 Sample 给出数据位置信息。


下图是 MediaParser 的功能界面：

![MediaParser](assets/resource/av-tool/media-parser-1.png)
_MediaParser_





### 3.3、MediaInfo



[MediaInfo](https://mediaarea.net/en/MediaInfo "MediaInfo") 是一个 MP4 基础信息展示工具。

下图是 MediaInfo 的功能界面：

![MediaInfo](assets/resource/av-tool/media-info-1.png)
_MediaInfo_







<!-- 
### 3.3、Atom Inspector


Atom Inspector 是苹果提供的一个 MP4 格式分析工具。不过由于不再继续维护，在最新版本的 Mac OS 上已经不可用了。

![Atom Inspector](assets/resource/av-tool/atom-inspector-1.png)
_Atom Inspector_ 
-->



### 3.4、FLVParser

[FLVParser](https://github.com/imagora/FlvParser "FLVParser") 是一个可以解析在线 FLV 流，输出该 FLV 流的 Tag 及详细信息的工具。


下图是 FLVParser 的功能界面：


![FLVParser](assets/resource/av-tool/flv-analyzer-1.png)
_FLVParser_




### 3.5、VLC

[VLC](https://www.videolan.org/vlc/ "VLC") 是一个开源跨平台的多媒体播放器，可以播放大多数多媒体文件，并查看媒体信息。

下图是 VLC 的功能界面：

![VLC](assets/resource/av-tool/vlc-1.jpg)
_VLC_


### 3.6、Native HLS Playback


[Native HLS Playback](https://chrome.google.com/webstore/detail/native-hls-playback/emnphkkblegpebimobpbekeedfgemhof "Native HLS Playback") 是一个 Chrome 浏览器的插件，用于支持在 Chrome 上直接播放 HLS/M3U8/TS 流。这样配合 Chrome 的 Inspect/Network 功能就能查看 HLS 流的具体信息。

下图是 Native HLS Playback 的功能界面：

![Native HLS Playback](assets/resource/av-tool/native-hls-playback-1.jpg)
_Native HLS Playback_


### 3.7、Play HLS M3u8


[Play HLS M3u8](https://chrome.google.com/webstore/detail/play-hls-m3u8/ckblfoghkjhaclegefojbgllenffajdc "Play HLS M3u8") 也是一个 Chrome 浏览器的插件，用于支持在 Chrome 上直接播放 HLS/M3U8/TS 流。这样配合 Chrome 的 Inspect/Network 功能就能查看 HLS 流的具体信息。

下图是 Play HLS M3u8 的功能界面：

![Play HLS M3u8](assets/resource/av-tool/play-hls-m3u8-1.jpg)
_Play HLS M3u8_





















