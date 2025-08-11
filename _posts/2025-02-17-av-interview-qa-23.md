---
title: 音视频面试题集锦第 23 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 04:18:08 +0800
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




下面是第 23 期面试题精选：

- **1、VideoToolbox 遵循哪种视频码率控制策略？如何设置？**
- **2、Annex B 如何转换为 AVCC？**
- **3、iOS 中如何判断一个视频帧是不是关键帧？**
- **4、纹理有哪些环绕方式（wrapping）？**


## 1、VideoToolbox 遵循哪种视频码率控制策略？如何设置？

码率控制策略主要分为以下几种：

- CBR（Constant Bit Rate）恒定码率：一定时间范围内比特率基本保持的恒定。
	- 有运动发生时，由于码率恒定，只能通过增大 QP 来减少码字大小，图像质量变差；当场景静止时，图像质量又变好，因此图像质量不稳定。
	- 优点是码率处于一个稳定值，缺点是质量不稳定，在复杂运动场景下的视频会很糊。
	- 适合在流式播放中应用。
- VBR（Variable Bit Rate）可变码率：码率分配根据图像内容的复杂度进行。
	- 简单场景分配较低码率，复杂场景分配较高码率。
	- 优点是视频质量稳定，缺点是码率不可控，编码速度较慢。
	- 适合的应用场景是本地存储（如视频录制），不适合网络传输（如直播推流）。
- ABR（Average Bitrate）平均目标码率：控制一段时间内的编码平均码率。
	- 是在 CBR 和 VBR 两者之间的一种权衡，即设定一段时间的平均码率，在此时间内，对简单的、静态的图像分配低于平均码率的码率，对于复杂的，大量运动的图像分配高于平均码率的码流。
	- 速度快，同时兼顾了视频质量和带宽,对于转码速度有要求的情况下也可以选择该模式。
	- 适合网络传输。

目前 VideoToolbox 没有属性可以直接设置码率控制策略给调用方，只有开放了 `kVTCompressionPropertyKey_DataRateLimits`(为最高码率上限)和 `kVTCompressionPropertyKey_AverageBitRate`(编码平均码率)。可以通过 API 属性名称和注释结合编码后的视频码率猜测 VideoToolBox 目前使用的应该是 ABR 视频编码策略。


## 2、 Annex B 如何转换为 AVCC？

Annex B 格式通常以 `0x000001` 或 `0x00000001` 用于标识 NAL 单元的开始。SPS 和 PPS 按流的方式写在头部。

AVCC 格式使用 NALU 长度（固定字节，通常为 4 字节）分隔 NAL；在头部包含 extradata 或 sequence header 的结构体。

以下是 AnnexB 转换为 AVCC 的思路：

- 1、解析 Annex B 格式：读取字节流，识别每个 NAL 单元的起始码，确定每个 NAL 单元的开始和结束位置。
- 2、去除起始码：去除每个 NAL 单元的起始码。
- 3、计算长度：对于每个 NAL 单元，计算其长度（以字节为单位）。
- 4、写入长度前缀：将每个 NAL 单元的长度作为字节序列写入到 AVCC 格式的流中，可能 1 个字节，2 字节或者 4 字节（较为常见），NAL 单元长度会存储在 AVCC 的 extradata 中。
- 5、根据 Annex B 的 SPS 和 PPS 生成对应的 extradata。
- 6、写入 NAL 单元数据：在长度字段后面写入去除起始码后的 NAL 单元数据。

## 3、iOS 中如何判断一个视频帧是不是关键帧？

在 VideoToolbox 中，可以通过检查给定的 `CMSampleBuffer` 是否是视频帧，并且是否是关键帧。通过检查 `kCMSampleAttachmentKey_NotSync` 键的值，如果它为 `false` ，则说明这是一个关键帧。以下是示例代码


```objc
#import <VideoToolbox/VideoToolbox.h>

BOOL isKeyFrame(CMSampleBufferRef sampleBuffer) {
    CMFormatDescriptionRef formatDescription = CMSampleBufferGetFormatDescription(sampleBuffer);
    if (!formatDescription) {
        return NO;
    }
    
    CMMediaType mediaType = CMFormatDescriptionGetMediaType(formatDescription);
    if (mediaType != kCMMediaType_Video) {
        return NO;
    }
    
    CFArrayRef sampleAttachmentsArray = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true);
    CFDictionaryRef sampleAttachments = (CFDictionaryRef)CFArrayGetValueAtIndex(sampleAttachmentsArray, 0);
    BOOL isKeyFrame = !CFDictionaryContainsKey(sampleAttachments, kCMSampleAttachmentKey_NotSync);
    return isKeyFrame;
}
```


## 4、纹理有哪些环绕方式（wrapping）？

- `重复（GL_Repeat）`：纹理在每个纹理坐标轴上重复出现，当纹理坐标超出 `[0,1]` 范围时，纹理会在该轴上重复出现。这种方式适用于创建无缝平铺效果。这是对纹理的默认行为。
- `镜像重复（GL_MIRRORED_REPEAT）`：与重复（GL_Repeat）方式相似，但当纹理坐标超出 `[0,1]` 范围时，会将其镜像翻转后再重复出现。这可以有效减少纹理重复造成的视觉疲劳。
- `夹取到边缘（GL_CLAMP_TO_EDGE）`：与夹取方式类似，但在超出范围时，会使用边缘纹素的颜色，产生一种边缘被拉伸的效果。
- `夹取到边框（GL_CLAMP_TO_BORDER）`：超出范围时，使用指定的边框颜色。这种方式通常用于在超出纹理范围时填充边框颜色，避免黑边。


![纹理环绕方式](assets/resource/av-interview-qa/texture-wrapping.webp)



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
>你还可以加入我们的微信群：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

