---
title: 音视频面试题集锦第 7 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 02:08:08 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦, 音视频]
pin: false
math: true
mermaid: true
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

我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。

下面是 2023.09 月音视频面试题集锦的一些精选：

- **1、简要介绍一下对 H.264 的了解？**
- **2、H.264 编码框架分层目的是什么？**
- **3、H.264 如何根据 NALU 判断当前视频帧的类型？**
- **4、介绍一下 I、P、B 帧编码、解码、显示顺序？**
- **5、H.264 与 H.265 有什么区别？**

## 1、简要介绍一下对 H.264 的了解？

**1）基础描述**

H.264 是由国际标准组织机构（ISO）下属的运动图象专家组（MPEG）和国际电传视讯联盟远程通信标准化组织（ITU-T）开发的系列编码标准之一。

![](assets/resource/av-interview-qa/video_encode_develop.png)
	
**2）码流结构**

H.264 原始码流（裸流）是由⼀个接⼀个 NALU 组成，它的功能分为两层：VCL（视频编码层）和 NAL（⽹络抽象层）。

- 视频编码层 VCL（Video Coding Layer）：是对视频编码核心算法过程、子宏块、宏块、片等概念的定义。这层主要是为了尽可能的独立于网络来高效的对视频内容进行编码。
- 网路抽象层 NAL（Network Abstract Layer）：负责将 VCL 产生的比特字符串适配到各种各样的网络和多元环境中，覆盖了所有片级以上的语法级别。

![](assets/resource/av-interview-qa/nalu.png)

**3）两种封装**

H.264 的两种封装：AnnexB 模式和 AVCC 模式。

- AnnexB 模式：
	- 传统模式
	- 有 startcode，startcode 码是：`00 00 01` 或 `00 00 00 01` (3 字节或 4 字节) 
	- SPS 和 PPS 在码流中分别作为一个 NALU
- AVCC 模式：
	- 没有 startcode，SPS 和 PPS 以及其它信息被封装在 container 中，每⼀个 frame 前⾯ 4 个字节是这个 frame 的⻓度
	- ⼀般在 mp4、mkv 格式中常用 AVCC 模式

很多解码器只⽀持 AnnexB 这种模式，因此需要将 AVCC 模式做转换，在 ffmpeg 中⽤ `h264_mp4toannexb_filter` 可以做转换，实现如下：

```c
const AVBitStreamFilter *bsfilter = av_bsf_get_by_name("h264_mp4toannexb"); 

AVBSFContext *bsf_ctx = NULL; 
// 初始化过滤器上下⽂ 
av_bsf_alloc(bsfilter, &bsf_ctx); 
// 添加解码器属性
avcodec_parameters_copy(bsf_ctx->par_in, ifmt_ctx->streams[videoindex]->cod ecpar);
av_bsf_init(bsf_ctx);
```

## 2、H.264 编码框架分层目的是什么？

对 H.264 编码框架进行分层的主要目标是为了有高的视频压缩比和良好的网络亲和性。

VCL 层负责视频的信号处理，包含压缩，量化等处理，NAL 层则负责解决编码后数据的网络传输。

这样可以将 VCL 和 NAL 的处理放到不同平台来处理，可以减少因为网络环境不同对 VCL 的比特流进行重构和重编码。

这样将编码和网络传输进行隔离，使功能单一、便于维护。


## 3、H.264 如何根据 NALU 判断当前视频帧的类型？

NALU 结构一般为：`[NALU Header][NALU Payload]`，可以根据 `[NALU Header]` 这 1 个字节来获取帧类型，它的结构如下图：

![](assets/resource/av-interview-qa/nula_type1.png)

- F：1bit，禁⽌位，H.264 规范中规定了这⼀位必须为 0，值为 1 表示语法出错（编码出错），不可用。
- R：2bit，被参考的级别，重要性指示位，取值越⼤，表示当前 NALU 越重要，需要优先受到保护，如果当前 NALU 是属于参考帧的⽚、序列参数集或图像参数集这些重要的单位时，本句法元素必需⼤于 0。
- T：5bit，负荷数据类型，表示 NALU 单元的类型，1～12 由 H.264 使⽤，24～31 由 H.264 以外的应⽤使⽤。

![](assets/resource/av-interview-qa/nula_type2.png)

常用的 NAL 头的取值类型：

| 0x | 0b | 类型 | 重要程度 |
| :---: | :---: | :---: | :---: |
| 0x67 | 0 11 00111 |  SPS   |  非常重要 |
| 0x68 | 0 11 01000 |  PPS   |  非常重要 |
| 0x65 | 0 11 00101 |  IDR 帧 |  非常重要 | 
| 0x61 | 0 11 00001 |  I帧   |  重要 |
| 0x41 | 0 10 00001 |  P帧   |  重要 |
| 0x01 | 0 00 00001 |  B帧   |  不重要 | 
| 0x06 | 0 00 00110 | SEI   |  不重要 |



## 4、介绍一下 I、P、B 帧编码、解码、显示顺序？

我们以下图为例来介绍一下 I、P、B 帧的编码过程：

![](assets/resource/av-interview-qa/frame_order.png)

编码器编码一个 I 帧，然后向后跳过几个帧，用这个 I 帧作为基准帧对一个未来 P 帧进行编码，然后跳回到这个 I 帧之后的下一个帧。I 帧和 P 帧之间的帧可以被编码为 B 帧。之后，编码器会再次跳过几个帧，使用第一个 P 帧作为基准帧，编码另外一个 P 帧，然后再次跳回，用 B 帧填充显示序列中的空隙。这个过程不断持续，然后每间隔一定的帧数后插入一个新的 I 帧。

由于帧之间存在依赖关系，所以各帧的解码顺序和编码顺序是一致的，先被编码的帧在解码时就会先被解码。为了实现这一点，编码的时候需要根据每帧的编码顺序会为其记录上一个 DTS（Decoding Time Stamp）用于解码时按此顺序进行解码。

如上面介绍的编码过程，P 帧由前一个 I 帧或 P 帧来预测，而 B 帧由前后的两个 P 帧或一个 I 帧和一个 P 帧来预测，因而当存在 B 帧时，帧的编解码顺序和帧的显示顺序会有所不同，这时候就需要为每帧记录上一个 PTS（Presentation Time Stamp）用于解码后按顺序显示。



## 5、H.264 与 H.265 有什么区别？

**1）主要区别**

- H.265 也称为高效视频编码 (HEVC)，是 H.264 的升级和更高级的版本；
- H.265 的编码架构大致上 和 H.264 的架构相似，主要也包含：帧内预测（intra prediction）、帧间预测（inter prediction）、转换（transform）、量化（quantization）、去区块滤波器（deblocking filter）、熵编码（entropy coding）等模块。但在 H.265 编码架构中，整体被分为了三个基本单位，分别是编码单位（coding unit, CU）、预测单位（predict unit, PU）和转换单位（transform unit, TU）；
- 比起 H.264，H.265 提供了更多不同的工具来降低码率，以编码单位来说，H.264 中每个宏块（macroblock/MB）大小最大为 16x16 像素，而 H.265 的编码单位最大为 64x64；
- H.265 的帧内预测模式支持 35 种方向（而 H.264 只支持 8 种），并且提供了更好的运动补偿处理和矢量预测方法。

![](assets/resource/av-interview-qa/h264_vs_h265_1.png)

参考：[Difference Between H.264 and H.265](https://www.gumlet.com/learn/h264-vs-h265/ "Difference Between H.264 and H.265")

**2）比较点**

- 压缩比：压缩比是区分 H.264 和 H.265 编解码器的主要因素。与 H.264 相比，H.265 提供了两倍的编码效率。 这意味着 H.265 在提供相同编码质量的同时节省了大约 50% 的比特率。更具体而言，H.265 的平均比特减少在 4K UHD 时为 65%，在 1080p 时为 60%，在 720p 时为 58%，在 480p 时为 50%。
- 视频质量：H.264 和 H.265 编解码器在相同比特率下的视频质量存在很大差异。在 H.264 中，块的边界可能会失真。这是因为每个宏块是固定的，每个宏块的数据是相互独立的。另一方面，使用 H.265 时，图像更清晰、更详细，并且具有更少的阻塞和伪影。这是因为它根据区域信息确定 CTU 的大小。因此，H.265 在压缩时优于 H.264，具有更好的图像质量；
- 文件（码流）大小：编解码器对数字视频的压缩程度与需要传输或流式传输的最终文件大小直接相关。带宽越小，文件大小越低。通常，H.264 编解码器生成的视频比 H.265 生成的视频大 1-3 倍。 因此，在文件大小和保存大文件的有限存储空间方面，H.265 胜过 H.264；
- 兼容性：在兼容性方面，H.264 胜过 H.265，与 H.264 相比，H.265 的普及程度相当落后。如果 100 个设备和平台支持 H.264 编解码器，您会发现只有 30 个相应的设备和平台支持 H.265。您不能否认 H.265 是未来的编解码器，并且缓慢但肯定地，更多的平台和设备将适应 H.265；
- 性能：关于整体性能比较，H.265 无疑胜过 H.264，但这并非没有它的背景。H.264 具有适用于几乎所有常见设备的日常用例。然而，H.265 编码需要高计算能力。因此，H.265 可以比 H.264 更有效地压缩视频，同时保持相同水平的图像质量。H.264 性能达不到 4K 流媒体的标准，但 H.265 做到了这一点。

![](assets/resource/av-interview-qa/h264_vs_h265_2.png)

<!-- 参考：[Difference Between H.264 and H.265](https://www.gumlet.com/learn/h264-vs-h265/ "Difference Between H.264 and H.265")
 -->



---

**更多关于 H.264 的细节参考：[《可能是最详尽的 H.264 编码相关概念介绍》](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)**



---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_