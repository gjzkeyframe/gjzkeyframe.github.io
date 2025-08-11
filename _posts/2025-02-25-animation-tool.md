---
title: 音视频场景常用的动画系统：PAG 与 Lottie
description: 介绍音视频场景常用的动画系统：PAG 与 Lottie。
author: Keyframe
date: 2025-02-25 09:08:08 +0800
categories: [音视频实战经验]
tags: [音视频实战经验, 音视频, 视频编辑, 特效]
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




在音视频编辑、特效等场景我们通常需要用到一些动画效果，结合一些动画系统解决方案，可以大大降低我们开发的工作量，这里面常用到开源动画系统的就有：Lottie 和 PAG。这篇文章里我们就来介绍一下这两者。



## 1、 背景


Lottie 和 PAG 都是旨在提高跨平台动画实现效率的技术，但它们的开发背景和特点有所不同。

Lottie 是由 Airbnb 开发的开源库，最初是为了解决在不同平台上重现 Adobe After Effects 设计的动画的问题。Lottie 允许设计师将 AE 动画导出为 JSON 文件，然后通过 Lottie 的跨平台库在各种设备上渲染这些动画。Lottie 的开发背景主要是为了提高设计和开发的协作效率，同时保持动画的质量和性能。它特别适合实现 UI 动画、状态过渡效果、错误提示等，且易于集成和使用，具有跨平台兼容性、高效率协作、卓越性能等优势。

PAG（Portable Animated Graphics）则由腾讯自主研发，是一套完整的动画工作流解决方案，支持从 AE 导出到多平台渲染的全流程。PAG 的开发背景是为了提供一个高效的文件格式和渲染架构，以实现高性能的动画播放，同时支持更广泛的 AE 特性和跨平台一致性。PAG 特别关注性能优化和动画文件的压缩率，支持图层级别的编辑和视频模板功能，适用于 UI 动画、贴纸动效、照片或视频模板等场景。

总的来说，Lottie 的开发背景更侧重于提供一个简单易用、跨平台的动画解决方案，而 PAG 则更注重性能优化和对复杂动画特性的支持，两者各有侧重点，适用于不同的使用场景和需求。



## 2、 工作流

### 2.1、Lottie 使用流程

Lottie 的使用流程主要包括以下几个步骤：

- 1、设计师在 Adobe After Effects (AE) 中制作动画。
- 2、使用 Bodymovin 插件将 AE 动画导出为 JSON 数据格式。
- 3、开发者使用 Lottie 库加载 JSON 文件并在相应平台上渲染动画 。

![Lottie 使用流程](assets/resource/av-experience/lottie_product.png)
_Lottie 使用流程_



我们还可以用 [Lottie Web 显示](https://m.bejson.com/ui/lottie/ "Lottie Web 显示") 这个工具来展示 Lottie 动画。


![Lottie Web 显示工具](assets/resource/av-experience/lottie_show.png)
_Lottie Web 显示工具_




Lottie 动画对应的 Json 文件示例：

![Lottie Json 文件](assets/resource/av-experience/lottie_json.png)
_Lottie Json 文件_




### 2.2、PAG 使用流程


PAG 的使用流程则包括：

- 1、设计师在 AE 中设计动画并通过 PAGExporter 导出成 PAG 文件。
- 2、使用 PAGViewer 预览 PAG 文件动画效果。
- 3、在各端使用 PAG SDK 加载和渲染 PAG 文件 。

![PAG 使用流程](assets/resource/av-experience/pag_product.png)
_PAG 使用流程_


PAG 生成的是二进制文件，这里就不展示其具体内容了。

我们可以使用 [PAGViewer](https://pag.art/docs/zh-CN/pag-download.html "PAGViewer") 显示其动画效果。

![PAGViewer](assets/resource/av-experience/pag_show.png)
_PAGViewer_



PAGViewer 同时还可以进行编辑与性能监控：

![PAG 性能监控](assets/resource/av-experience/pag_edit_show.png)
_PAG 性能监控_



## 3、 文件大小

PAG 和 Lottie 在文件大小方面有显著差异，主要源于它们所采用的文件格式和压缩技术。

PAG 使用的是自研的二进制文件格式，具有高压缩能力，这使得它能够将各种资源合并到单一文件中，同时保持比其他解决方案更高的压缩比率和更快的解码速度。PAG 采用了动态比特位压缩技术，通过跳过大量默认值的存储，使用比特位来紧凑存储，可以在相同动画内容的情况下，实现比同类型方案平均减少约 50% 的文件大小。在实际对比中，PAG 导出的二进制文件通常只有 Lottie JSON 文件的 56% 大小，如果不进行 zip 压缩，差距还会更大。

相比之下，Lottie 使用 JSON 格式存储动画数据，这种文本格式的可读性高，但文件体积相对较大，且解码速度较慢。Lottie 社区提供了一些工具，如 Lottiemizer，来帮助优化 Lottie 动画文件的大小，通过使用压缩算法减小文件大小，同时保持动画质量。

总结来说，PAG 在文件大小和压缩比率上具有明显的优势，这主要得益于其采用的二进制文件格式和先进的压缩技术。而 Lottie 虽然在文件格式上不如 PAG 紧凑，但其社区提供了工具来优化文件大小，以改善性能和用户体验。


![文件大小对比](assets/resource/av-experience/pag_lottie_filesize.png)
_文件大小对比_



## 4、 性能

PAG 和 Lottie 在性能方面各有优势，它们在渲染机制、文件格式、解码速度以及内存使用等方面存在差异。

首先，PAG 使用自研的二进制文件格式和编解码器，这使得它在解码速度上具有显著优势，解码速度可以比 Lottie 快 20 倍，首帧渲染耗时也比Lottie 快 6 倍，总耗时上比 Lottie 快约 9 倍。

在渲染性能方面，PAG 的优势在于它使用 C++ 实现的跨平台渲染架构，所有平台共享同一套代码，天然地保障了跨端渲染的一致性。而 Lottie 则依赖于各端使用 native 语言的研发策略，不同端上的渲染效果可能因平台差异性而不一致。

在内存使用方面，PAG 的内存占用可能会比 Lottie 稍高一些，尤其是在渲染复杂动画时。然而，PAG 提供了一些优化策略，比如使用矢量导出而不是 BMP 预合成导出，以及优化 BM P预合成文件，以减少内存占用和提高性能。

总的来说，PAG 在性能和解码速度上具有明显优势，特别是在需要跨平台一致渲染效果的场景中。而 Lottie 则在易用性和设计灵活性方面表现更佳，适合需要快速迭代和设计复杂动画的项目。开发者应根据具体需求和场景选择合适的动画渲染框架。

![性能对比](assets/resource/av-experience/pag_lottie_speed.png)
_性能对比_



## 5、 AE 特性支持

PAG 支持几乎所有 AE 特性的导出，包括第三方插件效果和视频素材，通过矢量和 BMP 预合成混合导出的方式，实现了在支持所有 AE 特性的同时保持运行时的可编辑性。PAG 的这种全面支持为设计师提供了更大的创作空间，尤其适用于需要复杂动效的视频编辑场景。

相比之下，Lottie 虽然支持 AE 中的许多基本动画特性，如形状、颜色、变换等，但它主要针对的是矢量图形的动画，对一些复杂的 AE 特性支持有限。Lottie 的社区作品大多是矢量图形类动效，它在 Web 端支持的功能是最多的，但在客户端的兼容性方面可能存在一些限制。

总的来说，PAG 在 AE 特性支持和性能方面具有明显优势，尤其适合需要复杂动效和跨平台一致性渲染的项目；而 Lottie 则以其易用性、灵活性和丰富的社区资源在简单到中等复杂度的矢量动画场景中占有一席之地。开发者和设计师应根据项目的具体需求和场景选择合适的工具。


## 6、 工程 Demo 测试

- [PAG Demo](https://pag.art/docs/pad-demo-download.html "PAG Demo")
- [Lottie iOS 工程](https://github.com/airbnb/lottie-ios "Lottie iOS 工程")
- [Lottie Android 工程](https://github.com/airbnb/lottie-android "Lottie Android 工程")



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

