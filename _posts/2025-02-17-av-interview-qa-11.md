---
title: 音视频面试题集锦第 11 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:18:08 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦,  面试, 音视频]
pin: false
math: true
mermaid: true
---

>想要学习和提升音视频技术的朋友，快来加入我们的<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">【音视频技术社群】</a>，加入后你就能：
>
>- 1）下载 30+ 个开箱即用的「音视频及渲染 Demo 源代码」
>- 2）下载包含 500+ 知识条目的完整版「音视频知识图谱」
>- 3）下载包含 200+ 题目的完整版「音视频面试题集锦」
>- 4）技术和职业发展咨询 100% 得到回答
>- 5）获得简历优化建议和大厂内推
>  
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/keyframe-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }


我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。

下面是 2023.11 月音视频面试题集锦的几条干货精选：

- **1、OpenGL 的双缓冲机制是什么？eglCreateWindowSurface、eglCreatePbuffferSurface 和双缓冲机制有什么关联吗？**
- **2、请问 Android 上如何识别一个视频是哪种格式的 HDR 视频：HDR10+/DolbyVision/HLG/HDR10？**
- **3、HEVC OpenGOP 的新增的帧类型有哪些, 在开发中需要注意什么？**
- **4、介绍一下 Android 14 引入了 Ultra HDR Image 格式？**

## 1、OpenGL 的双缓冲机制是什么？eglCreateWindowSurface、eglCreatePbuffferSurface 和双缓冲机制有什么关联吗？

双缓冲机制主要目的是为了解决计算机图形学中的屏幕闪烁和画面流畅性问题。该机制通过在内存中创建两个缓冲区：一个用于绘制图像的后缓冲区，一个用于显示图像的前缓冲区，来避免因为输入输出速度不匹配造成的界面闪烁、卡顿等现象。这个问题是很老的问题了，目前的系统基本都已经支持双缓冲了。

双缓冲机制与的 `eglCreateWindowSurface`、`eglCreatePbuffferSurface` 这两个方法没有直接的关系。这两个方法是为了实现当前屏幕渲染和离屏渲染的功能，`eglCreateWindowSurface` 是创建屏幕上的渲染区域来实现屏幕渲染，`eglCreatePbuffferSurface` 是创建屏幕外的渲染区域来实现离屏渲染。也就是说你创建 `eglCreateWindowSurface` 就自动支持双缓冲机制了。



## 2、请问 Android 上如何识别一个视频是哪种格式的 HDR 视频：Dolby/HLG/HDR10/HDR10+？

- Dolby：MediaExtractor 可以解析出 `minetype`，如果是 `video/dolby-vision` 则格式为 Dolby 视频；
- HLG：解析出 `METADATA_KEY_COLOR_TRANSFER`，如果颜色转换函数是 `HLG` 则格式为 HLG 视频；
- HDR10/HDR10+：可以通过硬件解码出来的 `mediaFormat` 变化回调 `onOutputFormatChanged` 来判断，代码如下：

```java      
public void onOutputFormatChanged(@NonNull MediaCodec mediaCodec, @NonNull MediaFormat mediaFormat) {
	if (format.containsKey(MediaFormat.KEY_HDR10_PLUS_INFO)){
	  Log.d(TAG, "hdr10");
	} else if (format.containsKey(MediaFormat.KEY_HDR_STATIC_INFO)){
	  Log.d(TAG, "hdr10+");
	}
}
```

## 3、HEVC OpenGOP 的新增的帧类型有哪些，在开发中需要注意什么？


HEVC 包含大量不同的帧类型。这些类型都会在 NALU 头信息中标记便于我们识别帧类型。其中 OpenGOP 包含下列帧类型：

- `IRAP（Intra Random Access Pictures）`：随机访问帧。解码器可以从该帧开始解码。IRAP 包含三种帧类型：瞬时解码器刷新帧（IDR）、干净随机访问帧（CRA）、断开链路访问帧（BLA）。视频的解码过程始终要从 IRAP 帧开始。
- `前导帧（Leading pictures）`：按输出顺序位于随机访问点图片之前，但在编码视频序列中在随机访问点图片之后进行编码。
	- `RADL（Random Access Decodable Leading pictures）`：按照编码顺序独立于随机访问点之前的图片的引导帧被称为随机访问可解码前导帧。
	- `RASL（Random Access Skipped Leading pictures）`：按照编码顺序使用随机访问点之前的图片进行预测的前导帧可能会被损坏。这些被称为随机访问跳过前导帧。
- `尾随帧（Trailing pictures）`：在输出和解码顺序上均在 IRAP 和前导图片之后。
- `TSA（Temporal Sublayer Access）`和 `STSA（Stepwise Temporal Sublayer Access）`：标记解码器需要切换视频分辨率的帧，也是编码分层逻辑里面的分层转换点。

![HEVC Open GOP](assets/resource/av-interview-qa/hevc_open_gop.png)

开发 OpenGOP 视频需要注意的点，读者可以结合上面的示例来理解：

- 因为 IRAP 帧类型不止 IDR 类型，因此开始播放的点可以不用强制必须从 IDR 开始；
- Leading pictures 的显示时间在前，解码时间再后，因此需要在解码之后进行排序后再进行出帧；
- RASL 可能会依赖上一个 GOP 的内容，因此第一个 IRAP 之后的 RASL 帧应该丢弃，否则会解码失败；
- 解码器在识别 TSA 和 STSA 帧时需要重启一个对应分辨率解码器；
- 编码开启 OpenGOP 需要考虑消费端是否兼容的场景，可以在 metadata 里面标记让消费侧可以选择是否消费 OpenGOP 的视频。


## 4、介绍一下 Android 14 引入了 Ultra HDR Image 格式？

Ultra HDR 图片格式的原理是结合了标准 8-bit JPEG 基础图像与一个较低分辨率的、带有增益映射的 JPEG 图像以及用于 HDR 重建的元数据。首先，它通过加入一个标准 8-bit 的 JPEG 压缩图像，这个图像提供了基础的色彩和细节。然后，它关联了一个较低分辨率的 JPEG 图像，这个图像带有增益映射，可以提供额外的细节和动态范围。最后，它还包含了用于 HDR 重建的元数据，这些元数据可以用来创建 HDR 图像。

Ultra HDR 图片格式的核心优势在于其自适应的渲染方式。它可以根据硬件设备、显示能力的条件来选择最终的渲染方式。即使硬件设备或应用程序无法识别文件中的 HDR，Ultra HDR 照片格式仍然可以作为普通的 SDR JPEG 文件进行解析和显示，具备完全的向下兼容性。

![Ultra HDR Image](assets/resource/av-interview-qa/ultrahdr.png)

Ultra HDR 图片解码过程如下：

- 1、格式识别：符合此格式的 JPEG 文件可通过主图片的 XMP 数据包中是否存在 `hdrgm:Version="1.0"` 来识别；
- 2、找到增益映射图像：绿色部分主图像在 XMP 中包含了 `Container:Directory` 元素，定义文件容器中后续媒体文件的顺序和属性。容器中每个文件在 `Container:Directory` 中都有一个相应的媒体项，媒体项描述文件容器中的位置及每个串联文件的基本属性。紫色部分为 MPF 数据，储存在主图像中 App2 字段，主要包含了文件容器中 Primary 图和 GainMap 图的偏移及文件长度。
- 3、处理无效元数据：如果必填字段不存在，或存在任何包含无效值的字段，则元数据会被视为无效。值可能无效，原因是该值无法解析为指定类型或超出预期范围。如果遇到无效元数据，应忽略增益映射并应显示 SDR 图像。
- 4、使用增益映射创建经调整的 HDR 呈现：红色部分为 `HDR Gain Map Metadata`。这部分数据说明了如何使用 GainMap 图将主图像渲染到高动态范围。

![Ultra HDR 图片编码](assets/resource/av-interview-qa/hdr_image_encode.png)

Ultra HDR 编码主要有以下 5 个步骤:

- 1、相机 Hal 采集到 HDR 数据（P010）通过 tonemap 函数生成 SDR 数据（YUV420）；
- 2、通过 HDR 数据和 SDR 数据生成未压缩的 GainMap（亮度差）数据；
- 3、GainMap 数据压缩成单通道 JPEG 文件（灰度图）；
- 4、SDR 数据压缩成 Primary JPEG 图；
- 5、生成 metadata 信息，并将 GainMap 灰度图添加到 Primary JPEG 后面。


---


**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_






---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群和更多同行朋友来交流和讨论：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

