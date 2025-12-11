---
title: AI 教程：图像局部重绘
description: Stable Diffusion 除了完整的绘制一幅图像，还能对图像的部分区域做重绘。
author: Keyframe
date: 2025-11-04 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-005.png
  alt: Stable Diffusion 图像局部重绘
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


Stable Diffusion 除了完整的绘制一幅图像，还能对图像的部分区域做重绘。当重绘部分是图像内部区域时就是`内补绘制（Inpaint）`。

这篇文章我们就来介绍一下 Stable Diffusion WebUI 中`内补绘制（Inpaint）`相关的功能。




## 1、内补绘制（Inpaint）

我们在前面一节介绍`图生图（img2img）`时提到过**内补绘制（Inpaint）**，Stable Diffusion WebUI 的内补绘制（Inpaint）功能面板是嵌在图生图（img2img）大栏目下并新增了一些参数设置组件，如下图所示：


![内补绘制（Inpaint）功能面板](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-001.png)




内补绘制（Inpaint）作为嵌在图生图（img2img）页面的子功能，在图生图（img2img）的基础上增加了一些特有能力：


- 内补绘制（Inpaint）相对图生图（img2img）提供了在输入图像上进行涂绘的功能组件。这个功能和`草稿生图（Sketch）`很像，但是不同之处在于内补绘制（Inpaint）的涂绘功能组件不能设置画笔的颜色，这是因为没有必要。内补绘制（Inpaint）的涂绘是画出蒙版（Mask）区域。
- 内补绘制（Inpaint）通过提示词、输入图像、图像蒙版区域共同引导 Stable Diffusion 进行图像重绘，但是这里只会重绘蒙版区域，蒙版以外的区域不会做修改。
- 内补绘制（Inpaint）相对图生图（img2img）增加了针对蒙版区域进行设置的系列参数。


### 1.1、内补绘制流程


我们下面通过一个示例来介绍一下这些特有能力。在这个示例中，我们想要在输入的小猫图像的猫头上添加一只皇冠头饰。我们使用`内补绘制（Inpaint）`功能的操作如下图所示：

![内补绘制（Inpaint）示例](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-004.jpg)


- 1）在`引导图像输入区`导入引导图。我们这里输入的引导图是一张梵高油画风格的猫。
- 2）使用`涂绘功能组件`画出蒙版区域。我们这里想要在猫头上增加皇冠，所以就在猫头上画出了一片蒙版区域。这里的涂绘画笔只有黑色，因为是画蒙版用的，不需要其他颜色。
- 3）在`提示词输入区`输入提示词。我们这里输入的提示词 `cat with crown, painting`。
- 4）在`参数设置区`设置图生图相关参数以及针对内补绘制新增的蒙版区域相关参数。我们这里都使用了默认参数。
- 5）点击`生成（Generate）`按钮启动图生图任务。
- 6）在`生成图预览和功能区`就可以等待生成的结果。


下面是我们生成的结果：

![内补绘制（Inpaint）示例生成结果](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-005.png)

从结果图片可以看到，内补绘制（Inpaint）只对蒙版区域进行了重绘，其他区域都是和输入的原图保持一致。这是和我们前面介绍的草稿生图（Sketch）不一样的地方。



### 1.2、内补绘制参数


内补绘制（Inpaint）在图生图（img2img）基础上增加的参数有这些：

- **1）蒙版模糊度（Mask blur）**
- **2）蒙版模式（Mask mode）**
- **3）蒙版遮罩内容（Masked content）**
- **4）重绘大小（Inpaint area）**


**1）蒙版模糊宽度（Mask blur）**


这个参数表示蒙版区域与原图界限的进行模糊平滑处理的宽度。这个参数的取值范围是 0-64。数值越小，边缘越锐利，会使得蒙版重绘后和原图界限太明显；数值越大，边缘模糊区域宽度越大，也可能会显得不自然。我们需要根据图片的情况选择一个合适的值让图片看起来更真实，通常使用默认值 `4` 即可。


**2）蒙版模式（Mask mode）**

这个参数用来指定重绘的目标区域，包括 2 种选项：

- `重绘蒙版区域（Inpaint masked）`：表示只重绘蒙版遮罩的区域。
- `重绘非蒙版区域（Inpaint note masked）`：与上面相反，表示只重绘非蒙版区域。


我们用一个示例来说明它们的区别。我们在下面的原图上涂绘了蒙版：

![内补绘制（Inpaint）原图](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-006.png)

![内补绘制（Inpaint）蒙版](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-007.png)

设置输出宽高为原图宽高（512x768），输入提示词 `Oil painting portrait of a boy holding an orange` 后，我们分别选择`重绘蒙版区域（Inpaint masked）`和`重绘非蒙版区域（Inpaint note masked）`两种模式生成的图像如下图：

![重绘蒙版区域（Inpaint masked）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-008.png)

![重绘非蒙版区域（Inpaint note masked）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-009.png)



**3）蒙版填充内容（Masked content）**

这个参数是指定对重绘区域进行绘制时应该使用什么内容，包括下面 4 种选项：

- `填充（fill）`：参考原图的一个非常模糊版本来开始绘制。
- `原始（original）`：参考蒙版区域对应的原图内容来开始绘制。这通常是我们最想要的选项，也是默认选项。
- `潜在空间噪声（latent noise）`：使用基于 Seed 值来产生的初始噪声在潜在空间开始绘制。选择这个选项就可能画出跟原图完全不相关的内容。
- `无潜在空间（latent nothing）`：基于蒙版区域附近的颜色来得到一个纯色填充到蒙版区域来开始绘制。


<!--  

https://onceuponanalgorithm.org/guide-stable-diffusion-inpaint-masked-content-options-explained/

Fill: The InPaint result will be generated off of an extremely blurred version of the input image.
Original: The result will be generated based on the original content of the designated sections of the image to be altered. This is what you will want most of the time.
Latent Noise: This option is good to select if you want the inpainted output to be very different from the original image, since the designated area will be inpainted based off of noise produced from the seed number. Basically this is starting from a blank slate.
Latent Nothing: In this option, Stable Diffusion will fill in the designated area with a single solid color that is a blend of the colors from the surrounding pixels. This option is good to select if you want the InPaint to be extremely different from the original image but still maintain a vestige of its color palette.

-->



和上面使用同样的原图、蒙版、输出宽高和提示词，下面分别是我们使用`填充（fill）`、`原始（original）`、`潜在空间噪声（latent noise）`、`无潜在空间（latent nothing）`选项生成的结果：

![填充（fill）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-010.png)

![原始（original）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-008.png)

![潜在空间噪声（latent noise）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-011.png)

![无潜在空间（latent nothing）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-012.png)

一般来说，我们选择**原始（original）**即可。






**4）重绘大小（Inpaint area）**


重绘大小（Inpaint area）是指对重绘区域进行重绘的尺寸处理方式，包括下面 2 种选项：

- `全图（Whole picture）`：在整个输入图像的基础上来生成新图，然后将新图中对应重绘区域的部分混合到原图中去作为生成结果。这个是默认选项。
- `仅蒙版（Only masked）`：将重绘区域放大到你指定的尺寸后进行绘图，画完后再将其缩小到原图相应的位置并与原图融合后作为生成结果。
	- `仅蒙版时边距（Only masked padding, pixels）`：当选择了`仅蒙版（Only masked）`时，你还需要设置这个参数。该参数数值越高，最终生成的结果越接近原图。

和上面使用同样的原图、蒙版、输出宽高和提示词，下面是我们分别选择`全图（Whole picture）`、`仅蒙版（Only masked）`选项，其他参数使用默认值生成的结果：

![全图（Whole picture）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-008.png)

![仅蒙版（Only masked）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-013.png)











## 2、基于草稿内补绘制（Inpaint sketch）

**基于草稿内补绘制（Inpaint sketch）**是在内补绘制（Inpaint）的基础上增强了蒙版在颜色设置方面的能力，如下图：


![基于草稿内补绘制（Inpaint sketch）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-020.png)


- 1）蒙版可以自定义颜色，并且这个颜色会影响蒙版区域重绘的颜色。
- 2）新增了一个`蒙版透明度（Mask transparency）`的参数，支持设置蒙版的透明度。


下面是我们使用同一张输入图和同样的提示词 `cat with hat, painting`，但分别使用红色和蓝色蒙版生成的结果：

![红色蒙版](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-021.png)

![蓝色蒙版](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-022.png)

可以看到蒙版颜色对最终生成结果的影响。






## 3、基于上传蒙版内补绘制（Inpaint upload）


`基于上传蒙版内补绘制（Inpaint upload）`相对于内补绘制（Inpaint）的区别是蒙版是通过一个独立的图片文件指定，蒙版文件中白色区域表示重绘区域，黑色区域表示不重绘区域。如下图所示：

![基于上传蒙版内补绘制（Inpaint upload）](assets/resource/aigc-tutorial/sd-inpaint/sd-inpaint-024.png)

其他功能与内补绘制（Inpaint）基本一致。






















<!--  

Stable Diffusion Ultimate Guide pt. 4: Inpainting
https://medium.com/@inzaniak/stable-diffusion-ultimate-guide-pt-4-inpainting-772ea69472c9


Guide: Stable Diffusion InPaint Masked Content Options Explained
https://onceuponanalgorithm.org/guide-stable-diffusion-inpaint-masked-content-options-explained/


Tutorial: How to Use InPainting in the Stable Diffusion Web UI
https://onceuponanalgorithm.org/using-inpaint-in-stable-diffusion-tutorial/
-->










