---
title: AI 教程：从图像获取提示词，提示词反推
description: 有时候我们看到不错的 AI 生成作品，想要获得对应的提示词来自己生成类似的图片，这时候有什么办法吗？
author: Keyframe
date: 2025-11-06 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-014.png
  alt: Stable Diffusion 提示词反推
---


>向你推荐 **<a href="https://apps.apple.com/app/id6752116909" target="_blank">FaceXSwap</a>**：On-Device Offline AI Face Swap for Free
>
>在 iPhone 上直接对照片、GIF 动图、视频中的人脸实现快速换脸。无需上传任何数据，确保完全隐私与安全。即时、安全、无限次数、功能强大，现在即可免费试用。
>
>![在 AppStore 搜索 'facexswap'](assets/resource/aigc-product/facexswap-2.png)
>_在 AppStore 搜索 'facexswap'_
>
>- FaceXSwap 官网：<a href="https://facexswap.com" target="_blank">https://facexswap.com</a>
>- FaceXSwap iOS App 下载：<a href="https://apps.apple.com/app/id6752116909" target="_blank">https://apps.apple.com/app/id6752116909</a>
{: .prompt-tip }

---


有时候我们看到不错的 AI 生成作品，想要获得对应的提示词来自己生成类似的图片，这时候有什么办法吗？这章内容，我们就来介绍几种可以尝试的方案。


## 1、使用 PNG Info 提取提示词

在我们使用 Stable Diffusion WebUI 来生成作品时，默认情况下，生成任务用到的提示词、负向提示词以及相关参数信息都会被写入到生成作品 PNG 图像信息中去。对于这种情况，我们只需要将这些信息读取处理就可以获得作品的提示词了，甚至还包括更全面的其他参数信息。



Stable Diffusion WebUI 提供了从图片中读取这些信息的功能页面：`PNG Info`。该功能是 Stable Diffusion WebUI 的常用功能之一，可以通过此功能自动填写所有配置信息来还原出原图像。



使用 `PNG Info` 的流程如图：


![使用 PNG Info](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-014.png)


- 1）在图片输入区导入图片；
- 2）等待一段时间后，如果图片中包含对应的信息，在图片信息区就会显示读取出来的信息；
- 3）在功能区，可以选择将相关参数拷贝设置到 `txt2img`、`img2img`、`inpaint`、`extras` 等功能页面。


这里可以提取到的参数信息包括：

- 提示词
- 负向提示词（Negative prompts）
- 采样步数（Steps）
- 采样器（Sampler）
- 提示词相关性（CFG scale）
- 种子值（Seed）
- 分辨率（Size）
- 模型（Model）
- Clip skip
- Stable Diffusion WebUI 版本（Version）

不过，导入的图片并非 Stable Diffusion WebUI 生成的图片，或图片中的信息被清除过，那么在 `PNG Info` 中将无法读取到对应的参数信息。


















## 2、使用模型反推获取提示词


使用 `PNG Info` 提取提示词本质上只是信息的读取，下面我们要介绍的方法则是使用 AI 模型去理解图像内容并推理出对应的提示词。


我们打开 Stable Diffusion WebUI 页面的图生图栏目，可以看到提示词反推功能。如图：


![反推提示词](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-001.png)



**使用模型反推获取提示词**包括两种方式：

- CLIP 反推（Interrogate CLIP）
- DeepBooru 反推（Interrogate DeepBooru）

**CLIP 反推**是一种深度学习模型，它能够学习图像与相应文本描述之间的关联，从而可以用于图像检索、图像描述生成等任务。侧重于理解图像的内容，包括里面对象的关系。

**DeepBooru 反推**是一个用于图像标注和分析的开源项目，它通过训练神经网络模型来自动为图片添加标签，从而方便进行图像管理和搜索。侧重于对图像内容的识别，生成标签。






### 2.1、使用 Interrogate CLIP 反推获取提示词


使用 Interrogate CLIP 反推获取提示词的流程如下图所示：

![Interrogate CLIP 反推提示词](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-005.png)


- 1）在图片输入区选择并导入图片；
- 2）点击 `Interrogate CLIP` 按钮启动 CLIP 反推提示词；
- 3）在提示词输入框获得反推得到的提示词。


我们在上面示例中反推得到的提示词如下：

```
a painting of a woman sitting at a table with a vase of flowers in her lap and a vase of flowers in her lap, post-impressionism, impressionist painting, Blanche Hoschedé Monet, a painting
```

翻译一下就是：

```
一幅女人坐在桌边的画，她的腿上放着一瓶花，后印象派，印象派绘画，布兰奇·霍舍德·莫奈，一幅画
```

可以看出 CLIP 反推得到的提示词侧重于图像的内容，包括里面对象的关系。


接下来，我们在文生图（txt2img）中把上面得到的提示词作为输入来重新生成一幅图像，看看效果：

![用 CLIP 反推的提示词生成图像](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-007.png)


生成的结果与原画有一些差异，不过构图上还是对上了原图的内容以及对象的关系。




### 2.2、使用 Interrogate DeepBooru 反推获取提示词


使用 Interrogate DeepBooru 反推获取提示词的流程如下图所示：


![Interrogate DeepBooru 反推提示词](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-008.png)


- 1）在图片输入区选择并导入图片；
- 2）点击 `Interrogate DeepBooru` 按钮启动 DeepBooru 反推提示词；
- 3）在提示词输入框获得反推得到的提示词。


我们在上面示例中反推得到的提示词如下：


```
1girl, acrylic paint \(medium\), book, bookshelf, chair, fine art parody, indoors, painting \(medium\), photo \(medium\), profile, rain, signature, sitting, solo, traditional media, unconventional media
```

翻译一下就是：

```
1 个女孩，丙烯颜料，书，书架，椅子，美术模仿，室内，绘画，照片，轮廓，雨，签名，坐着，独奏，传统媒介，非传统媒介
```

可以看出 DeepBooru 反推后的提示词更侧重于用一个一个的标签来描述内容特征，但是没有描述对象之间的关系。


接下来，我们在文生图（txt2img）中把上面得到的提示词作为输入来重新生成一幅图像，看看效果：

![用 DeepBooru 反推的提示词生成图像](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-009.png)


生成的结果与原画有一些差异，画出了原画中的一些元素，但是对象之间的关系与原画有很大出入。




### 2.3、叠加使用 CLIP 和 DeepBooru 反推的提示词


CLIP 和 DeepBooru 反推的提示词各有特点，基于 CLIP 反推得到提示词更侧重于图像的内容及对象构图关系，DeepBooru 反推得到的提示词更侧重于生成标签来描述对象特征，那我们可以结合两者特性一起，看看效果。

我们就简单将二者反推到的提示词拼接起来：


```
a painting of a woman sitting at a table with a vase of flowers in her lap and a vase of flowers in her lap, post-impressionism, impressionist painting, Blanche Hoschedé Monet, a painting, acrylic paint \(medium\), book, bookshelf, chair, fine art parody, indoors, painting \(medium\), photo \(medium\), profile, rain, signature, sitting, solo, traditional media, unconventional media
```

然后，我们使用文生图（txt2img）生产新图，效果如下：


![叠加使用 CLIP 和 DeepBooru 反推的提示词](assets/resource/aigc-tutorial/sd-interrogate/sd-interrogateclip-010.png)


生成的图像效果还不错。


在使用时，我们可以根据自己的需求组合使用 CLIP 和 DeepBooru 来反推提示词，比如根据图 A 获取描述人物特征的提示词，根据图 B 获取描述图像内容与构图关系的提示词，然后叠加这些提示词来生成新图。此外，反推功能的结果并不一定精确，可以作为参考功能来使用。










