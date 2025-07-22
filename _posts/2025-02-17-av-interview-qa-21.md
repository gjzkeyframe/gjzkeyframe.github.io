---
title: 音视频面试题集锦第 21 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:58:58 +0800
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



下面是第 21 期面试题精选：


- **1、纹理抗锯齿有哪些算法？各有哪些利弊？**
- **2、使用 OpenGL PBO 为什么能提高效率？**
- **3、iOS 如何使用分段转码，如何设置分片大小？**
- **4、VideoToolbox 中是不是不存在平面格式（planar）对应的 YUV420、YUV422 和 YUV444 的 OSType 常量？**



## 1、纹理抗锯齿有哪些算法？各有哪些利弊?

纹理抗锯齿主要是指在计算机图形学中，减少或消除图像中由于纹理映射导致的锯齿效应的技术。常见的有以下几种：

- FXAA（快速近似抗锯齿）：
	- FXAA 是一种后处理技术，主要通过在像素着色器中应用边缘检测算法，对边缘附近的像素进行模糊处理，以减少锯齿。
	- 它对显卡的要求不高，不依赖于额外的采样，因此性能消耗相对较低。
	- FXAA 可以提供较快的处理速度，但可能会导致一些细节丢失，图像看起来可能会有些模糊。
- SSAA（超级采样抗锯齿）：
	- SSAA 是一种全场景抗锯齿技术，它通过在更高的分辨率下渲染整个场景，然后将其缩放到最终输出的分辨率，以获得更平滑的边缘。
	- 这种方法可以在不损失细节的情况下提供非常高质量的图像，但性能消耗很高，因为它需要渲染更多的像素。
	- SSAA 通常用于离线渲染，而不是实时渲染，因为它对硬件资源的要求非常高。
- MSAA（多重采样抗锯齿）：
	- MSAA 是一种在渲染过程中应用的抗锯齿技术，它只对每个像素的多个样本进行计算，而不是对整个像素进行计算。这可以减少几何锯齿，但对纹理锯齿的效果有限。
	- MSAA 主要针对多边形边缘进行抗锯齿处理。相比 SSAA、MSAA 的性能消耗要低得多，因为它不需要渲染额外的像素，但可能在画质上略有妥协。





## 2、 使用 OpenGL PBO 为什么能提高效率？

为什么能提高效率：

- 减少 CPU 等待：PBO 支持异步传输，这意味着 CPU 在发起传输请求后不必等待 GPU 完成传输，可以继续执行其他任务。
- 减少数据拷贝：使用 PBO 可以减少从 CPU 内存到 GPU 内存的数据拷贝次数。例如，当更新纹理时，可以先将数据复制到 PBO，然后由 GPU 直接从 PBO 读取，而不是每次都从 CPU 内存中复制。
- 提高带宽利用：PBO 允许更有效地利用内存带宽，因为它减少了 CPU 和 GPU 之间的数据传输量。
- 优化显存利用：使用 PBO 可以避免在每次更新纹理时销毁和重新创建纹理内存，从而优化显存的利用率。
- 双缓冲或多缓冲技术：通过使用两个或多个 PBO，可以在一个 PBO 进行 GPU 操作的同时，使用 CPU 填充另一个 PBO，从而实现更高效的流水线操作。

![双 PBO](assets/resource/av-interview-qa/pbo.png "双 PBO")

例如上图所示，利用 2 个 PBO 从帧缓冲区读回图像数据，使用 `glReadPixels` 通知 GPU 将图像数据从帧缓冲区读回到 PBO1 中，同时 CPU 可以直接处理 PBO2 中的图像数据。

通过交换 PBO 的方式进行拷贝和传送，可以实现这两步操作同时进行。

- 内存映射：PBO 的内存映射机制允许 CPU 直接访问 GPU 的缓冲区，这样可以更快速地传输数据，因为它避免了常规内存访问的开销。
- 适用场景：对于需要频繁更新或读取大量像素数据的应用程序，如图像处理、计算机视觉或大规模渲染任务，PBO 可以显著提高性能。



## 3、iOS 如何使用分段转码，如何设置分片大小？


初始化 `AVAssetWriter` 时可以设置 `outputFileTypeProfile = AVFileTypeProfileMPEG4AppleHLS` 即可打开 MP4 分段转码。

通过 `preferredOutputSegmentInterval` 设置转码后的每片大小。如果想手动切片，需要设置 `preferredOutputSegmentInterval = kCMTimeIndefinite`，并且在每次想要切片的位置调用 `flushSegment` 接口强制分片，注意调用接口的时机必须是 Sync 帧前。即每片的开始帧都是 Sync 帧。

分片的结果会通过设置的 `AVAssetWriterDelegate` 内部方法返回。

```objc
assetWriter_ = [[AVAssetWriter alloc] initWithContentType:[UTType typeWithIdentifier:AVFileTypeMPEG4]]; // 不能设置输出文件因为已经分段
assetWriter_.outputFileTypeProfile = AVFileTypeProfileMPEG4AppleHLS; 
assetWriter_.preferredOutputSegmentInterval = CMTimeMake(5, 1); // 5s 一段
assetWriter_.initialSegmentStartTime = kCMTimeZero; // 开始时间，必设
assetWriter_.delegate = self; 
```


## 4、VideoToolbox 中是不是不存在平面格式（planar）对应的 YUV420、YUV422 和 YUV444 的 OSType 常量？

iOS 的 API 中能找到的平面格式有：`kCVPixelFormatType_420YpCbCr8Planar`、`kCVPixelFormatType_420YpCbCr8PlanarFullRange`，这两种平面格式分别对应 y420 和 f420。对这两种格式测试下来，目前系统播放器支持，系统相机不支持。yuv422 和 yuv444 的平面格式目前没找到。


此外，iOS 的 API 中能找到的半平面格式有：`kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange`、`kCVPixelFormatType_422YpCbCr8BiPlanarVideoRange`、`kCVPixelFormatType_444YpCbCr8BiPlanarVideoRange` 这些。





---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_






