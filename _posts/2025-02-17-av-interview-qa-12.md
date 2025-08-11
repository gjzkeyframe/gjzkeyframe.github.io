---
title: 音视频面试题集锦第 12 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:28:08 +0800
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


下面是第 12 期面试题精选：


- **1、iOS 平台上如何判断 VideoToolbox 是否支持某种编码格式（H.264、HEVC 等）或者某种颜色空间呢？**
- **2、Android 平台上使用 MediaCodec 编码，如何告知编码器结束编码？**
- **3、想要把 iOS、Android 应用开发中 OpenGL ES 渲染相关模块下沉到 C++ 实现双端共用要怎么实现？**
- **4、Android 平台上使用 MediaCodec 异步编码，使用 Surface 作为编码输入源的流程和使用 ByteBuffer 作为编码输入源有什么区别？**




## 1、iOS 平台上如何判断 VideoToolbox 是否支持某种编码格式（H.264、HEVC 等）或者某种颜色空间呢？

  

1）判断 VideoToolbox 是否支持一种编码格式，首先可以查一下 `CMFormatDescription.h` 文件的 `CMVideoCodecType` 枚举中有没有对应的编码格式。对于具体的设备是否支持某种编码格式，可以用类似下面的代码检查：

````objc
BOOL supportHEVC = NO;
if (@available(iOS 11.0, *)) {
	if (&VTIsHardwareDecodeSupported) {
		supportHEVC = VTIsHardwareDecodeSupported(kCMVideoCodecType_HEVC);
	}
}
````

2）判断是否支持某一种类型的颜色空间，可以先看看 `CVPixelBuffer.h` 文件中的 `kCVPixelFormatType` 关于颜色空间的声明。

  

## 2、Android 平台上使用 MediaCodec 编码，如何告知编码器结束编码？

MediaCodec 内部是状态机的流转，有三种状态：Stopped、Executing、Released。

其中：

- Stopped 包含三种子状态：Uninitialized、Configured、Error。
- Executing 包含三种子状态：Flushed、Running、End-of-Stream。


当我们想要告知编码器结束编码的时候，其实是在 Executing 状态中，从 Running 子状态中流转到 End-of-Stream 子状态中，并且在 MediaCodec 使用结束后，调用 `release()` 方法将其释放。


当 MediaCodec 的输入队列中接受了一个带有 `_BUFFER_FLAG_END_OF_STREAM_` 标志的输入帧时，MediaCodec 会继续输出剩余的编码数据。直到收到 End-of-Stream 的编码帧后，可以认为编码器已经从之前的 Running 状态运行到 End-of-Stream 状态。

在输入带有 `_BUFFER_FLAG_END_OF_STREAM_` 标志的输入帧后，不可再继续向 MediaCodec 输入数据，除非 MediaCodec 已经被 Flush、Stop 或 Restart。

  
<!-- ![MediaCodec 状态机](https://developer.android.com/images/media/mediacodec_async_states.svg) -->

![MediaCodec 状态机](assets/resource/av-interview-qa/mediacodec_async_states.png)




## 3、想要把 iOS、Android 应用开发中 OpenGL ES 渲染相关模块下沉到 C++ 实现双端共用要怎么实现？

  
大致过程如下：

- 1、创建一个 C++ 的渲染引擎模块。该模块将负责处理 OpenGL ES 的渲染逻辑。您可以使用现有的 C++ 图形库，如 Angle、Diligent Engine 等，或者自己编写一个。
- 2、将渲染逻辑从 iOS 和 Android 应用中移动到 C++ 渲染业务模块。包括创建 OpenGL ES 上下文、编译和链接着色器程序、设置渲染状态、绘制图形等。
- 3、封装 C++ 接口层。为了在 iOS 和 Android 应用中使用 C++ 渲染模块，您需要封装 C++ 接口层，该接口层需要定义 iOS 和 Android 业务层代码与渲染模块交互的函数和方法。这样，应用程序可以通过调用这些接口来使用 C++ 渲染模块。
- 4、处理平台特定的差异。由于 iOS 和 Android 平台的差异，您可能需要处理一些平台特定的差异，例如，处理不同的输入事件、处理不同的窗口管理等。在 C++ 接口中，可以定义平台特定的函数，然后在 iOS 和 Android 应用中实现这些函数。

参考：

- [Angle](https://github.com/google/angle "Angle")
- [Diligent Engine](https://github.com/DiligentGraphics/DiligentEngine "Diligent Engine")
- [Diligent Engine 示例](https://github.com/DiligentGraphics/DiligentSamples "Diligent Engine 示例")



  

## 4、Android 平台上使用 MediaCodec 异步编码，使用 Surface 作为编码输入源的流程和使用 ByteBuffer 作为编码输入源有什么区别？

使用 ByteBuffer 异步编码的时候，Surface 和 ByteBuffer 都需要设置 MediaCodec 的 `setCallback` 方法设置相关的回调。

区别在于：

- 在使用 ByteBuffer 作为输入源的时候，我们需要在 `onInputBufferAvailable` 的时候根据 index 拿到空闲的 ByteBuffer，塞入有效的数据后，再调用 `queueInputBuffer` 传回给 codec 进行编码。
- 在使用 Surface 作为输入源的时候，Surface 会自己管理 buffer，`dequeueInputBuffer` 和 `getInputBuffers` 的调用都被认为是非法的。因此 `onInputBufferAvailable` 方法不会回调，我们需要把数据直接送到 Surface 上进行渲染。


---


## 音视频岗位招聘


字节跳动音视频团队，主要负责剪映上的音视频非线性编辑相关工作，目前业务前景较好，有三个方向的岗位招人：

- 桌面端音视频研发：https://job.toutiao.com/s/i8enPrw5
- 多端音视频引擎研发：https://job.toutiao.com/s/i8enjTHT
- C++工程基础架构研发：https://job.toutiao.com/s/i8enr7Es

Base 北京、上海、广州、深圳、杭州都可以，薪资也比较 open，欢迎大家投递简历。


有需要发布音视频岗位招聘信息的朋友欢迎加博主微信。




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

