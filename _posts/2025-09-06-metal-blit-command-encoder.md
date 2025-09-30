---
title: 探索 Metal 音视频技术（6）：Blit 命令编码器
description: 本文将介绍 Metal 计算命令编码器。
author: Keyframe
date: 2025-09-06 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 渲染, Metal]
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

---






这个系列文章我们来探索 Metal 音视频技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，本篇介绍 Blit 命令编码器。


`MTLBlitCommandEncoder` 提供了在资源（缓冲区和纹理）之间复制数据的方法。数据复制操作对于图像处理和纹理特效（例如模糊或反射）可能是必需的，也可用于访问离屏渲染的图像数据。

要执行数据复制操作，首先通过调用 `MTLCommandBuffer` 的 `blitCommandEncoder` 方法创建一个 `MTLBlitCommandEncoder` 对象。然后调用下文所述的 `MTLBlitCommandEncoder` 方法，将命令编码到命令缓冲区中。

## 1、在 GPU 内存中的资源对象之间复制数据


以下 `MTLBlitCommandEncoder` 方法用于在资源对象之间复制图像数据：在两个缓冲区之间、在两个纹理之间，以及在缓冲区和纹理之间。

### 1.1、在两个缓冲区之间复制数据

方法 `copyFromBuffer:sourceOffset:toBuffer:destinationOffset:size:`  将数据从一个缓冲区复制到另一个缓冲区。如果源缓冲区和目标缓冲区是同一个缓冲区，且被复制的区域发生重叠，则结果未定义。

### 1.2、从缓冲区复制数据到纹理

方法 `copyFromBuffer:sourceOffset:sourceBytesPerRow:sourceBytesPerImage:sourceSize:toTexture:destinationSlice:destinationLevel:destinationOrigin:` 将图像数据从源缓冲区复制到目标纹理 `toTexture`。

### 1.3、在两个纹理之间复制数据

方法 `copyFromTexture:sourceSlice:sourceLevel:sourceOrigin:sourceSize:toTexture:destinationSlice:destinationLevel:destinationOrigin:` 将图像数据的某个区域从一个纹理复制到另一个纹理，从源纹理的单个立方体贴图切片和 Mipmap 级别复制到目标纹理 `toTexture`。

### 1.4、从纹理复制数据到缓冲区

方法 `copyFromTexture:sourceSlice:sourceLevel:sourceOrigin:sourceSize:toBuffer:destinationOffset:destinationBytesPerRow:destinationBytesPerImage:` 将图像数据的某个区域从源纹理的单个立方体贴图切片和 Mipmap 级别复制到目标缓冲区 `toBuffer`。

## 2、生成 Mipmap


`MTLBlitCommandEncoder` 的 `generateMipmapsForTexture:` 方法可以为给定纹理自动生成 Mipmap，从基础级别纹理图像开始。`generateMipmapsForTexture:` 会为所有 Mipmap 级别创建缩放图像，直到最大级别。

关于 Mipmap 的数量以及每个 Mipmap 的大小如何确定，详见“切片（Slices）”部分。

## 3、填充缓冲区内容


`MTLBlitCommandEncoder` 的 `fillBuffer:range:value:` 方法将 8 位常量 `value` 存储到给定缓冲区的指定 `range` 范围内的每个字节中。

## 4、结束 Blit 命令编码器的编码


要结束 blit 命令编码器的命令编码，调用 `endEncoding`。结束上一个命令编码器后，你可以创建任意类型的新命令编码器，将更多命令编码到命令缓冲区中。






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

  
