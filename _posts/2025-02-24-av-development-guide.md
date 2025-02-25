---
title: 音视频开发文章合集：69 篇文章带你系统性的学习音视频开发
description: 系列音视频开发相关的文章大合集。
author: Keyframe
date: 2025-02-24 16:28:08 +0800
categories: [职场进阶]
tags: [音视频基础知识, 音视频实用工具, 音视频源码示例, 音视频实战经验, 音视频]
pin: true
math: false
mermaid: false
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
>_微信扫码关注我们_
>
>您还可以加入知识星球 `关键帧的音视频开发圈` 来一起交流工作中的**技术难题、职场经验**：
>
>![知识星球](assets/img/keyframe-zsxq.png)
>_微信扫码加入星球_
{: .prompt-tip }




距离我们发出第一篇音视频技术文章已经过去一年了，回顾这一年，我们发了几十篇文章，覆盖了音视频**基础知识**、**实用工具**、**源码示例**、**实战经验**等主题，这些文章基本上构成了**入门音视频开发并做一些功能实现和指标优化工作所需要的知识框架**，这里我们来回顾下这些文章，做一下内容简介，给需要的朋友提供一些指引。也请大家多多**收藏**、**转发**、**点赞**。


最近我们还组建了**技术交流群**并试运营了一段时间，现在这个群放开了，如果大家是音视频开发或者对音视频开发感兴趣，可以在**本文末尾找到微信群的二维码来加入**，与更多的朋友来交流和讨论。


下面我们进入主题，按照下面的结构回顾一下这一年我们发出的 69 篇文章，希望对大家有用。

- 音视频基础
	- 声音和图像基础
	- 音视频编码
	- 音视频格式
	- 音视频协议
	- 渲染
- 音视频工具
- 音视频工程示例
	- 音视频 Demo
	- 渲染 Demo
	- 平台能力
- 音视频工业实战
	- 音视频生产
	- 音视频消费



## 1、音视频基础


### 1.1、声音和图像基础


这个章节的几篇文章从`将我们耳朵听见的声音、眼睛看见的画面，数字化为我们用手机、电脑所处理的音频数据和图像数据，其中经历了什么？`这个问题出发，分别探讨了声音和图像相关的基础原理知识。这其中包含了如何对司空见惯的声音和图像进行物理定义、特征探索、规律发现、数学描述，并用信息处理手段对它们进行数字化的过程。这些知识可能在音视频开发中并不会经常用到，但是对于我们更深入的理解音视频是有很大帮助的。


---

![《声音的表示》概要](assets/resource/av-guide/声音的表示.png)
_《声音的表示》概要_

- 1）[《声音的表示（1）：声音的定义和特征》](https://mp.weixin.qq.com/s/b4XkNaZWnSLx8KxqV0Ul1A)

>本文介绍了声音的定义：一种波动现象，以及声音几个特征：响度、音调、音色，还初步介绍了研究声音时的辅助工具：波形图和频谱图。

- 2）[《声音的表示（2）：声音的数学描述》](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)

>本文介绍了如何对声音的响度、音调、音色特征进行数学描述。在这个过程中，引入了众多的物理量和概念：比如与响度相关的声能、声强、声压、声强级、声压级、响度级等；与音调相关的频率、科学音调记号法、十二平均律等；与音色相关的基频、基音、谐波、泛音等。这些物理量和概念是对声音进行数学描述的工具和桥梁，而基于这些物理量和概念建立起来的数学模型是我们对声音数字化的基础。

- 3）[《声音的表示（3）：声音的数字化》](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)

>本文介绍了对声音进行数字化的过程：采样 → 量化 → 编码，以及数字音频的要素：采样率、量化位深、声道数。其中还讲到了有趣的小知识：44100Hz 这个奇葩采样率的来历。经过数字化过程后，就可以得到我们熟悉的 PCM 数字音频数据了。


---

![《图像的表示》概要](assets/resource/av-guide/图像的表示.png)
_《图像的表示》概要_


- 4）[《图像的表示（1）：图像的定义和成像原理》](https://mp.weixin.qq.com/s/fOiV32SZb7UcN6KLUCI61g)

>本文介绍了图像定义：人对视觉感知的物质再现。介绍了我们人眼感知图像及其颜色的人眼视觉感知三原色理论，还介绍了颜色的特征：色调（hue）、亮度（brightness）、饱和度（saturation）。




- 5）[《图像的表示（2）：图像的数学描述》](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)

>颜色是对图像视觉感知最核心的要素，所以对图像进行数学描述，最重要的是建立『颜色模型（颜色空间）』。所以这篇文章主要介绍了图像数字描述过程中对颜色进行建模的发展历程：基于人眼视觉感知三原色理论，CIE 通过大量实验数据建立了 RGB 颜色模型，标准化了 RGB 表示 → 为了解决 RGB 模型中与负光混合所带来的种种问题，CIE 从数学上定义了三种标准基色 XYZ，形成了 CIE XYZ 颜色模型 → 在模拟电视时代，RGB 工业显示器要求一幅彩色图像由分开的 R、G、B 信号组成，而电视显示器则需要混合信号输入，为了实现对这两种标准的兼容，NTSC 基于 XYZ 模型制定了 YIQ 颜色模型，实现了彩色电视和黑白电视的信号兼容 → 为了解决 NTSC YIQ 的组合模拟视频信号中分配给色度信息的带宽较低，而影响了图像颜色质量的问题，PAL 引入了 YUV 颜色模型，支持用不同的采样格式来调整传输的色度信息量 → 进入数字电视时代，ITU-R 为数字视频转换制定了 YCbCr 颜色模型，成为我们现在最常使用的颜色模型。在早年 CRT 显示器流行的年代，我们遇到了显示伽马问题，从而引入了伽马校正过程并延用至今。可见这一路都是遇到问题解决问题的过程。这些知识或许并不会在日常开发中用到，但知其然，又知其所以然，不亦乐乎。

- 6）[《图像的表示（3）：图像的数字化》](https://mp.weixin.qq.com/s/RhMLfOoDdf6TvRoGVQaNHA)


>本文介绍了最基本的图像数据数字化的过程：采样和量化。其中，对坐标值的数字化称为采样，对颜色值的数字化称为量化。本文也介绍了数字化处理后的图像的基本属性：图像分辨率和像素深度，并介绍我们在音视频开发中最常接触到的数字图像数据是 RGB、YCbCr 数据。

---



### 1.2、音视频编码

这个章节的几篇文章主要介绍了音频和视频主流的编码格式，比如音频的 PCM、AAC，视频的 H.264、H.265、H.266。我们介绍了这些编码格式的原理、规范和技术优化，帮助大家更深入的理解音视频编码。


---

![《音频编码》概要](assets/resource/av-guide/音频编码.png)
_《音频编码》概要_


- 7）[《音频编码：PCM 和 AAC》](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)

>对音频或视频进行编码最重要目的就是为了进行数据压缩，以此来降低数据传输和存储的成本。这篇文章从时域冗余、频域冗余、听觉冗余等方面介绍了音频压缩的原理。同时也介绍了 PCM 编码流程，并重点探讨了目前广泛流行的 AAC 编码的工具集、编码流程、编码规格以及对应的数据格式。



---

![《视频编码》概要](assets/resource/av-guide/视频编码.png)
_《视频编码》概要_


- 8）[《视频编码（1）：H.264（AVC）》](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)

>跟音频编码一样，视频编码最重要的目的也是为了进行数据压缩，以此来降低数据传输和存储成本。视频编码主要是建立在空间冗余、时间冗余、编码冗余、视觉冗余的基础上进行的。本文主要介绍了 H.264（AVC）编码的基本概念、分层结构、编码工具及码流结构。相关的基础知识对于帮助我们了解 H.264 编码，以及在其基础上继续发展演进的 H.265、H.266 编码都有很大的作用。


- 9）[《视频编码（2）：H.265（HEVC）》](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)

>本文主要介绍了 H.265（HEVC）视频编码技术的编码工具和特色编码技术，这些内容有助于我们了解 H.265 是如何在 H.264 的基础上通过技术发展和演进实现比前者更加的数据压缩效率。

- 10）[《视频编码（3）：H.266（VVC）》](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)

>本文主要介绍了 H.266（VVC）视频编码技术相对于 H.264、H.265 在编码工具、系统和传输接口上的技术发展和优化，通过这些优化 H.266 给我们带来了更好的数据压缩效率和更多新能力的支持。

---



### 1.3、音视频格式

这个章节的几篇文章主要介绍了短视频和直播业务中经常使用的音视频封装格式，了解这些格式对于我们优化音视频业务体验以及完善相关功能有重要的作用。

---

![《MP4 格式》概要](assets/resource/av-guide/MP4格式.png)
_《MP4 格式》概要_

- 11）[《MP4 格式：短视频常用格式》](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)

>本文介绍了当下互联网短视频最常使用的封装格式 MP4 的基础格式。MP4 格式是以 Box 的形式进行组织，本文着重介绍了 File Type Box(ftyp)、Media Data Box(mdat)，以及最重要的 Movie Box(moov) 及其重要子 Box，并给出案例分析了 Box 的结构对优化视频播放体验的作用。

---

![《FLV 格式》概要](assets/resource/av-guide/FLV格式.png)
_《FLV 格式》概要_

- 12）[《FLV 格式：直播常用格式》](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)

>本文介绍了 FLV 流媒体格式，FLV 是一种结构相对简单的格式。在直播领域，由于 RTMP 推流、HTTP-FLV 播放的整套方案低延时的特性，以及服务端普遍提供 HTTP Web 服务，能更广泛的兼容 HTTP-FLV，使得 FLV 仍然是大多数直播产品的首选流媒体格式。

---


![《M3U8 格式》概要](assets/resource/av-guide/M3U8格式.png)
_《M3U8 格式》概要_


- 13）[《M3U8 格式：直播回放常用格式》](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)

>本文介绍了 M3U8 媒体格式，M3U8 是苹果公司推出的 HLS(HTTP Live Streaming) 协议的基础。在实际应用场景中，由于 HLS/M3U8/TS 这套方案在控制直播延时上不太理想，所以一般实时直播场景不会选择使用 M3U8 媒体格式。但是，对于直播回放这种场景，由于使用 M3U8/TS 这套方案能够在直播过程中就持续生成和存储切片，所以直播回放基本上都会选择 M3U8 媒体格式。

---

![《TS 格式》概要](assets/resource/av-guide/TS格式.png)
_《TS 格式》概要_


- 14）[《TS 格式：直播回放切片常用格式》](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)

>本文介绍了 TS 封装格式，TS 格式主要是用于传输流，它可以实时传输节目内容，这就要求从传输流的任一片段开始都是可以独立解码的，在直播中可以用到。也正是因为 TS 任一切片开始都可以独立解码，所以它非常适合按切片的方式存储直播内容。


---


### 1.4、音视频协议


这个章节的几篇文章主要介绍了常见的几种音视频协议，正确的选择和优化音视频协议，对于提升音视频业务体验非常重要，协议的选择甚至决定了某些音视频体验指标的下限和上限。

---

![《RTMP 协议》概要](assets/resource/av-guide/RTMP协议.png)
_《RTMP 协议》概要_



- 15）[《RTMP 协议：直播推流常用协议》](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)

>由于协议设计对低延时、音视频同步等能力的良好支持，RTMP 是实时直播场景，尤其是在推流上行链路中，最常用的传输协议之一。本文主要介绍了 RTMP 协议的数据传输流程和协议设计思想，并详细介绍了消息和块的具体细节规范。

---

![《KCP 协议》概要](assets/resource/av-guide/KCP协议.png)
_《KCP 协议》概要_

- 16）[《KCP 协议：自研常用参考协议》](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)

>KCP 是一个开源的快速可靠协议，能以比 TCP 浪费 10%-20% 带宽的代价，换取平均延迟降低 30%-40%，最大延迟降低 3 倍的传输速度。本文介绍了 KCP 协议的特性、基本使用方式及最佳使用实践。

---

![《HLS 协议》概要](assets/resource/av-guide/HLS协议.png)
_《HLS 协议》概要_

- 17）[《HLS 协议：直播回放常用协议》](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)

>HLS 作为苹果公司提出的协议，在 iOS 客户端上得到了很好的支持，比如 AVPlayer 和 Safari 都支持对 HLS 流媒体的播放；再加上 M3U8/TS 封装格式可以在直播中持续处理和存储流媒体数据，所以直播回放通常都会选择 HLS 协议来实现。HLS 协议的实现是和 M3U8 文件的定义密切相关的，这部分的知识在《M3U8 格式》中已经做了详细介绍。本文则简单介绍一下 HLS 协议的整体框架。





---




### 1.5、渲染

这个章节的几篇文章主要介绍了 OpenGL 的一些基础概念，对这些基础概念的正确理解将为后续更深入的学习 OpenGL 做好铺垫。

---

![《OpenGL 基础概念》概要](assets/resource/av-guide/OpenGL基础概念.png)
_《OpenGL 基础概念》概要_

- 18）[《OpenGL 基础概念（1）：渲染架构、状态机、渲染管线》](https://mp.weixin.qq.com/s/XNiDto9ABfTCCpptDktHng)

>本文介绍了 OpenGL、OpenGL ES、Metal、Vulkan 等场景的图形渲染方案及它们的历史渊源，并着重介绍了 OpenGL 在应用程序中的位置和角色，以及它的渲染架构、状态机、渲染管线的设计。

- 19）[《OpenGL 基础概念（2）：EGL》](https://mp.weixin.qq.com/s/ob9UID8xDazxcnYm_-dY8A)

>EGL 是 OpenGL ES 与设备的桥梁，以实现让 OpenGL ES 能够在当前设备上进行绘制。本文介绍了 EGL 的基础概念，还介绍了 Android 和 iOS 平台各自对 EGL 的实现方案。

- 20）[《OpenGL 基础概念（3）：VBO、EBO、VAO》](https://mp.weixin.qq.com/s/KYj4H1BhGd9YmhLHjIinbw)

>VBO、EBO、VAO 是几种优化 OpenGL 程序性能常用的对象，本文介绍了这几种对象的基础知识和使用方式。

- 21）[《OpenGL 基础概念（4）：FBO》](https://mp.weixin.qq.com/s/6AUoejRK0GOVsLzWV-w5kw)

>FBO 是我们使用 OpenGL 常用到的重要对象，它支持我们去做离屏渲染。本文介绍了 FBO 的基础知识和使用方式。

- 22）[《OpenGL 基础概念（5）：颜色混合》](https://mp.weixin.qq.com/s/mpj8_i71wbswRPtJp2r4cw)

>如果不能很好的理解 OpenGL 的颜色混合原理，很容易在开发中渲染不对我们需要的颜色。本文介绍了 OpenGL 的颜色混合基础知识。


---


### 1.6、其他


- 23）[《音视频基础概念合集：148 个音视频基础问题集锦》](https://mp.weixin.qq.com/s/X1idBFp7T5zhnqPmYSbQDw)

>本文以问题的形式介绍了众多的音视频基础概念的知识点，其范围涵盖声音和图像基础、音视频编码、音视频封装格式、音视频协议等方面。


## 2、音视频工具

这个章节的几篇文章介绍了常用的音视频工具，这些工具对于我们进行音视频数据分析、网络数据抓包、竞品分析等工作会有很大的帮助。借助这些工具往往可以让我们在工作中事半功倍。

---

![《FFmpeg 工具》概要](assets/resource/av-guide/FFmpeg工具.png)
_《FFmpeg 工具》概要_


- 24）[《FFmpeg 工具》](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)

>本文主要介绍了基于 FFmpeg 开源项目提供的 ffmpeg、ffplay、ffprobe 等命令行工具。这些工具可以帮助我们实现音视频转封装、转码、流媒体处理、音视频播放及音视频数据分析等工作。

---


![《可视化音视频分析工具》概要](assets/resource/av-guide/可视化音视频分析工具.png)
_《可视化音视频分析工具》概要_

- 25）[《可视化音视频分析工具》](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)

>本文介绍了多种可视化的音视频分析工具，这些工具可以帮助我们分析音画原始数据、音视频编码数据、音视频封装格式数据等。

---


![《数据抓包工具》概要](assets/resource/av-guide/数据抓包工具.png)
_《数据抓包工具》概要_

- 26）[《数据抓包工具》](https://mp.weixin.qq.com/s/EAJGSprnSXEb_dzMx71fZw)

>本文介绍了 Charles、Wireshark 等数据抓包工具，这些工具对于我们分析音视频协议数据非常有用。


---

![《iOS 逆向工具》概要](assets/resource/av-guide/iOS逆向工具.png)
_《iOS 逆向工具》概要_

- 27）[《iOS 逆向工具》](https://mp.weixin.qq.com/s/QhVyUzyZN_h1qjEZhO9jYQ)

>App 逆向工程是做竞品分析的常用方法，本文介绍了实现 iOS 逆向的几种工具，借助它们可以帮助我们实现竞品分析。

---



## 3、音视频工程示例




### 3.1、音视频 Demo

这个章节我们拆解了音频和视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并借助 iOS 和 Android 的系统音视频 API 能力来实现 Demo 向大家介绍如何在 iOS/Android 平台上手音视频开发。


---

![《iOS AVDemo》概要](assets/resource/av-guide/iOSAVDemo.png)
_《iOS AVDemo》概要_

- 28）[《iOS AVDemo（1）：音频采集》](https://mp.weixin.qq.com/s/FDR_5cMfAJQgZhSvjgeWYA)
- 29）[《iOS AVDemo（2）：音频编码》](https://mp.weixin.qq.com/s/q4n1dYTjcJVJolX-Wrdr9Q)
- 30）[《iOS AVDemo（3）：音频封装》](https://mp.weixin.qq.com/s/R86qnQAi2njr6k7tFvTF-w)
- 31）[《iOS AVDemo（4）：音频解封装》](https://mp.weixin.qq.com/s/fCZfIXriTXUPcI4d4te_ew)
- 32）[《iOS AVDemo（5）：音频解码》](https://mp.weixin.qq.com/s/7Db81B9i16cLuq0jS42bmg)
- 33）[《iOS AVDemo（6）：音频渲染》](https://mp.weixin.qq.com/s/xrt277Ia1OFP_XtwK1qlQg)
- 34）[《iOS AVDemo（7）：视频采集》](https://mp.weixin.qq.com/s/CJAhkk9BmhMOXgD2pl_rjg)
- 35）[《iOS AVDemo（8）：视频编码》](https://mp.weixin.qq.com/s/M2l-9_W8heu_NjSYKQLCRA)
- 36）[《iOS AVDemo（9）：视频封装》](https://mp.weixin.qq.com/s/W17eLiUeCszNM8Kg-rlmBg)
- 37）[《iOS AVDemo（10）：视频解封装》](https://mp.weixin.qq.com/s/4Ua9PZllWRLYF79hwsH0DQ)
- 38）[《iOS AVDemo（11）：视频转封装》](https://mp.weixin.qq.com/s/VVItfhebc6L-JQFCGBtapQ)
- 39）[《iOS AVDemo（12）：视频编码》](https://mp.weixin.qq.com/s/BIazU0Wd5_p4bx4nKJoH-g)
- 40）[《iOS AVDemo（13）：视频渲染》](https://mp.weixin.qq.com/s/4K8xPX_A8NA01ecmA6UCtw)

---

![《Android AVDemo》概要](assets/resource/av-guide/AndroidAVDemo.png)
_《Android AVDemo》概要_

- 41）[《Android AVDemo（1）：音频采集》](https://mp.weixin.qq.com/s/o0JqYnjY0aR3oGCEXk7vcQ)
- 42）[《Android AVDemo（2）：音频编码》](https://mp.weixin.qq.com/s/yI6XMPbLfNvaJNZEmQkNqQ)
- 43）[《Android AVDemo（3）：音频封装》](https://mp.weixin.qq.com/s/8PLWVp3soM7A5jJIywDmqQ)
- 44）[《Android AVDemo（4）：音频解封装》](https://mp.weixin.qq.com/s/2tv6J-11FMjq3YCQoJC8eQ)
- 45）[《Android AVDemo（5）：音频解码》](https://mp.weixin.qq.com/s/YLy2uGP_r5CQ8ToyXBtm8w)
- 46）[《Android AVDemo（6）：音频渲染》](https://mp.weixin.qq.com/s/wpuzD9Joxeln9Ji3w7YUtg)
- 47）[《Android AVDemo（7）：视频采集》](https://mp.weixin.qq.com/s/rS077Uu4qJnDJMDD7vto5Q)
- 48）[《Android AVDemo（8）：视频编码》](https://mp.weixin.qq.com/s/URVrvHrDaj-RsJnJiNT6oA)
- 49）[《Android AVDemo（9）：视频封装》](https://mp.weixin.qq.com/s/o136KnMrUcZTKA6rT6-eyA)
- 50）[《Android AVDemo（10）：视频解封装》](https://mp.weixin.qq.com/s/YoRFkRd0UURyCxMFEzpqGw)
- 51）[《Android AVDemo（11）：视频转封装》](https://mp.weixin.qq.com/s/VWk2TgXNSBBr4JLLnZUkOw)
- 52）[《Android AVDemo（12）：视频解码》](https://mp.weixin.qq.com/s/PK5IOzInOcxAmM3w9x9_Rw)
- 53）[《Android AVDemo（13）：视频渲染》](https://mp.weixin.qq.com/s/pycfFwBCnIQNUjtJpIlq9Q)

---


### 3.2、渲染 Demo

这个章节展示了一些渲染相关的 Demo，来向大家介绍如何在 iOS/Android 平台上手一些渲染相关的开发。

---


![《RenderDemo》概要](assets/resource/av-guide/RenderDemo.png)
_《RenderDemo》概要_

- 54）[《RenderDemo（1）：用 OpenGL 画一个三角形》](https://mp.weixin.qq.com/s/b-nFCBMf-oaayyG8a86mgw)
- 55）[《RenderDemo（2）：用 OpenGL 渲染视频》](https://mp.weixin.qq.com/s/eBB6jkvsufXbIWCLwsueOQ)



---


### 3.3、平台能力

这个章节的几篇文章主要是介绍了 iOS 平台的音视频处理框架及 API 能力。

---

![《iOS 音频处理框架及重点 API 合集》概要](assets/resource/av-guide/iOS音频处理框架及重点API.png)
_《iOS 音频处理框架及重点 API 合集》概要_


- 56）[《iOS 音频处理框架及重点 API 合集》](https://mp.weixin.qq.com/s/w_5pZoeV0GdcFppIpuvVcw)



---

![《iOS 视频处理框架及重点 API 合集》概要](assets/resource/av-guide/iOS视频处理框架及重点API.png)
_《iOS 视频处理框架及重点 API 合集》概要_

- 57）[《iOS 视频处理框架及重点 API 合集》](https://mp.weixin.qq.com/s/QqkwaVz4lX_yy4sGusoThg)

---


![《WWDC 2022 音视频相关的更新》概要](assets/resource/av-guide/WWDC2022音视频相关的更新.png)
_《WWDC 2022 音视频相关的更新》概要_

- 58）[《一文看完 WWDC 2022 音视频相关的更新要点》](https://mp.weixin.qq.com/s/K5DtSvK3RGLZZ5V761WllA)
- 59）[《WWDC 2022 音视频相关 Session 概览（HLS 相关）》](https://mp.weixin.qq.com/s/W-ABs1FaXk6Yy08Tp3eA7Q)
- 60）[《WWDC 2022 音视频相关 Session 概览（EDR 相关）》](https://mp.weixin.qq.com/s/LrGZRoZ95kjtv5VKxQvINg)


---

## 4、音视频工业实战



### 4.1、音视频生产


这个章节的几篇文章主要介绍了音视频生产端各个模块的关键指标定义及优化思路。


---

![《采集预览优化》概要](assets/resource/av-guide/采集预览优化.png)
_《采集预览优化》概要_


- 61）[《音视频生产关键指标：采集预览优化》](https://mp.weixin.qq.com/s/bPbRLmQn4ztKYVobS_xO0A)

>采集预览阶段表示打开相机，但是还没开始进行直播推流或者视频录制的阶段，但这时候一般也开始进行滤镜、美颜、特效前处理了。本文介绍了采集预览阶段关注的相机打开成功率、相机打开速度、采集预览流畅度、采集画面质量、采集内存等指标的定义和优化。


---

![《视频录制优化》概要](assets/resource/av-guide/视频录制优化.png)
_《视频录制优化》概要_

- 62）[《音视频生产关键指标：视频录制优化》](https://mp.weixin.qq.com/s/Y2gFat4iZXk9B-GE72jNcg)

>视频录制阶段除了开始采集音视频数据，做滤镜、美颜、特效等前处理，还会做音视频编码、封装、文件存储。本文介绍了视频录制阶段关注的录制成功率、录制流畅度等相关的指标定义和优化。

---


![《视频编辑优化》概要](assets/resource/av-guide/视频编辑优化.png)
_《视频编辑优化》概要_

- 63）[《音视频生产关键指标：视频编辑优化》](https://mp.weixin.qq.com/s/uBr0Um40ZztFsfGCWPxpZA)

>在视频编辑场景中，涉及到的模块很多，比如：抽帧模块、预览播放模块、视频编辑模块、特效合成模块、视频转码模块等等。这些模块各自都有对应的性能指标，这些指标影响着编辑场景的用户体验。本文介绍了抽帧模块和预览播放模块相关的指标定义和优化思路。

---


![《视频质量优化》概要](assets/resource/av-guide/视频质量优化.png)
_《视频质量优化》概要_

- 64）[《音视频生产关键指标：视频质量优化》](https://mp.weixin.qq.com/s/Qj46I9llahphBRAtAJ0DNw)

>随着音视频内容日趋成为主要的内容消费载体，用户们对视频清晰度、画质的要求也在不断提高，我们在这里把视频清晰度、画质都统称为视频质量，本文介绍了视频质量评估的标准及相关的优化思路。

---

![《视频发布优化》概要](assets/resource/av-guide/视频发布优化.png)
_《视频发布优化》概要_

- 65）[《音视频生产关键指标：视频发布优化》](https://mp.weixin.qq.com/s/Kgpy3r9PVGO78G-TzFXWaQ)

>视频发布流程是指视频录制和编辑完成后，对视频进行转码、上传的过程。本文介绍了视频发布成功率、发布耗时等视频发布流程中的指标定义及优化思路。

---


### 4.2、音视频消费

这个章节的几篇文章主要介绍了音视频消费端播放器的关键指标定义及优化思路。

---

![《播放器成功率优化》概要](assets/resource/av-guide/播放器成功率优化.png)
_《播放器成功率优化》概要_


- 66）[《音视频消费关键指标：播放器成功率优化》](https://mp.weixin.qq.com/s/FVN7lsPkPXz3F8R_BS74TQ)

>视频播放器是视频消费链路最核心的组件，本文主要介绍了视频播放器成功率相关的指标定义和优化思路。

---

![《播放器秒开优化》概要](assets/resource/av-guide/播放器秒开优化.png)
_《播放器秒开优化》概要_

- 67）[《音视频消费关键指标：播放器秒开优化》](https://mp.weixin.qq.com/s/8n8cgsv2o-ziFxK9fJO2Yw)

>视频播放时的画面打开速度是播放体验中一个非常重要的指标，如果视频画面打开速度太慢，用户失去耐心可能就直接划走不看了。如果视频速度打开够快，甚至可以带来业务上的收益。本文主要介绍了视频播放器秒开相关的指标定义和优化思路。

---




![《播放器卡顿优化》概要](assets/resource/av-guide/播放器卡顿优化.png)
_《播放器卡顿优化》概要_

- 68）[《音视频消费关键指标：播放器卡顿优化》](https://mp.weixin.qq.com/s/jx0krbxkSGdPBkiI1s282g)

>播放卡顿是播放体验中另一个非常重要的指标，本文介绍了播放器卡顿相关的指标定义和优化思路。

---




![《直播延时优化》概要](assets/resource/av-guide/直播延时优化.png)
_《直播延时优化》概要_

- 69）[《音视频消费关键指标：直播延时优化》](https://mp.weixin.qq.com/s/WOZFaCk4fP_xIbNKPhcPfA)

>直播播放延时，指的是从主播推流一帧画面到用户观看到这帧画面之间的时间差。本文介绍了直播播放延时的指标定义及优化思路。

---




