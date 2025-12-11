---
title: AI 教程：生成创意光影字
description: 将文字用光影效果隐藏在景物中，很酷也很有创意。这篇内容我们就来介绍一下如何制作 AI 光影字。
author: Keyframe
date: 2025-11-18 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-create-art-text/sd-create-art-text-003.png
  alt: Stable Diffusion 生成创意光影字
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



你也许看过下面这样的 AI 光影字作品：

![示例 1](assets/resource/aigc-tutorial/sd-create-art-text/sd-create-art-text-003.png)

![示例 2](assets/resource/aigc-tutorial/sd-create-art-text/sd-create-art-text-004.png)


将文字用光影效果隐藏在景物中，很酷也很有创意。这篇内容我们就来介绍一下如何制作 AI 光影字。




## 1、制作文字底图

制作 AI 光影字的第一步是先制作好文字底图，这个在后面会用于 ControlNet 的引导图。有很多工具可以用来做这个，比如：Photoshop、PPT、美图秀秀等等，这里就不多说了。这里需要注意，文字底图最好用**黑底白字**，这样比较适合作为 ControlNet 的引导图。

下面是一张示例底图：

![文字底图](assets/resource/aigc-tutorial/sd-create-art-text/sd-create-art-text-001.png)





## 2、配置模型

文字底图做好了，接下来要做的是配置模型。

这里需要配置的模型是一个 ControlNet 类型的模型 `lightingBasedPicture_v10`，可以在 [https://huggingface.co/casque/lightingBasedPicture_v10/blob/main/lightingBasedPicture_v10.safetensors](https://huggingface.co/casque/lightingBasedPicture_v10/blob/main/lightingBasedPicture_v10.safetensors) 下载该模型，并将该模型放在 `stable-diffusion-webui/extensions/sd-webui-controlnet/models` 目录下。





## 3、文生图配合 ControlNet 生成光影字


接下来，我们就可以使用 Stable Diffusion WebUI 来创作光影字了。

流程如下图所示：

![生成过程](assets/resource/aigc-tutorial/sd-create-art-text/sd-create-art-text-002.png)

- 1）打开 ControlNet 面板，导入我们上面制作的文字底图作为引导图；
- 2）选中 `Enable`，开启 ControlNet；
- 3）选择 ControlNet 预处理器和模型，这里的 `Control Type` 选择 `All`，`预处理器（Preprocessor）`不用，`控制模型（Model）`选择我们上面配置的 `lightingBasedPicture_v10`；
- 4）调整 `Control Weight` 和 `Ending Control Step` 参数，我们这里都是设置为了 `0.5`，这个数值大家可以自己调试看效果；
- 5）输入提示词，我们这里输入的很简单：`masterpiece, best quality, highres, mountains`；
- 6）设置其他文生图参数，这里主要是生成图的分辨率最好和 ControlNet 引导图的分辨率比例保持一致；
- 7）点击 `Generate` 按钮开始生成；
- 8）等待生成结果。




下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-art-text/sd-create-art-text-003.png)


这种玩法还是很有创意的。


