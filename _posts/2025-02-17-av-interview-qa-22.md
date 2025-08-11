---
title: 音视频面试题集锦第 22 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 04:08:08 +0800
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

下面是第 22 期面试题精选：

- **1、iOS 中系统 API 提供了哪些视频编码的方式？**
- **2、VideoToolbox 视频帧解码失败以后应该如何重试？**
- **3、如何使用 PSNR 对视频转码质量进行评估？**
- **4、什么是 VAO，什么是 VBO，它们的作用是什么？**



## 1、iOS 中系统 API 提供了哪些视频编码的方式？


在 iOS 中，实现视频编码的方式主要包括以下两种：

- AVFoundation 框架：AVFoundation 是苹果提供的一个用于处理音视频数据的框架，它提供了一系列用于捕获、处理和输出音视频数据的类和方法。通过 AVFoundation 框架，可以使用 AVAssetWriter 和 AVAssetWriterInput 类来实现编码视频。

- VideoToolbox 框架：VideoToolbox 是苹果提供的一个专门用于处理视频数据的框架，它提供了硬件加速的视频编码和解码功能。使用 VideoToolbox，可以利用 iOS 设备上的硬件编码器来实现高效的视频编码。


相比而言，AVFoundation 框架则提供了更加上层的接口，更简单易用，但因此对于一些特殊需求和高级功能，可能无法满足。VideoToolbox 则提供了更直接的对硬件编码器的访问，允许开发者能更细致的控制编码器的配置和参数，并且可以直接操作编码器的输入和输出数据，灵活性更好。





## 2、Videotoolbox 视频帧解码失败以后应该如何重试？


- 1、重新初始化解码器：尝试重新初始化 Videotoolbox 解码器，有时候重新初始化可以解决解码过程中的一些临时问题。
- 2、检查视频文件：确保视频文件没有损坏或者格式不正确。有时候解码失败是因为视频文件本身的问题，可以尝试使用其他工具或者重新获取视频文件。
- 3、检查当前内存：在解码过程中如果 CMSampleBuffer 不及时释放，可能会导致内存过高导致解码器报 -11800 通用错误。
- 4、尝试重新解码当前帧：将当前帧以及当前 gop 内前序帧都重新输入给解码器。



## 3、如何使用 PSNR 对视频转码质量进行评估？


- 1、计算图像差异：获得原始视频帧和转码后的未经过任何图像效果处理的视频帧使用同一解码器解码，并将它们的每一帧转换成相同的格式（比如 YUV 格式）。
- 2、计算 PSNR 值：使用以下公式计算每一帧的 PSNR 值。
- 3、计算平均 PSNR：将所有帧的 PSNR 值求平均，得到视频的平均 PSNR 值。
- 4、分析结果：根据平均 PSNR 值来评估转码后视频的质量。较高的 PSNR 值表示转码后的视频质量与原始视频相似度较高，而较低的 PSNR 值则表示质量损失较大。



举例来说两个宽高为 m×n 视频帧 I 和 K， I 为转码前视频帧，K 为转码后的视频帧，那么它们的均方误差（MSE）定义为：

![MSE 计算公式](assets/resource/av-interview-qa/mse.webp)


他们的 PSNR 计算公式如下：

![PSNR 计算公式](assets/resource/av-interview-qa/psnr.webp)



其中，MAX<sub>I</sub> 是表示图像点颜色的最大数值，如果每个采样点用 8 位表示，那么就是 255。



![不同 PSNR 的图像质量对比](assets/resource/av-interview-qa/psnr-compare.webp)



## 4、什么是 VAO，什么是 VBO，它们的作用是什么？


**1、Vertex Buffer Object (VBO)**

- VBO 主要用于存储顶点数据，如顶点坐标、法线、颜色等。
- 通过将顶点数据存储在 GPU 的显存中，可以提高渲染效率，因为 GPU 能够更快地访问这些数据，而无需反复从 CPU 内存中读取。

**2、Vertex Array Object (VAO)**

- VAO（Vertex Array Object）顶点数组对象，主要作用是用于管理 VBO 或 EBO。
- VBO 保存了一个模型的顶点属性信息，每次绘制模型之前需要绑定顶点的所有信息，当数据量很大时，重复这样的动作变得非常麻烦。VAO 可以把这些所有的配置都存储在一个对象中，每次绘制模型时，只需要绑定这个 VAO 对象就可以了，可以减少 glBindBuffer 、glEnableVertexAttribArray、 glVertexAttribPointer 这些调用操作，高效地实现在顶点数组配置之间切换。


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

