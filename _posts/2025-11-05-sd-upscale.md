---
title: AI 教程：图像高清修复
description: Stable Diffusion WebUI 为我们提供了其他方式来生成高分辨率的高清图，我们里就来介绍一下这些方案。
author: Keyframe
date: 2025-11-05 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-upscale/sd-upscale-030.png
  alt: Stable Diffusion 图像高清修复
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


在很多场景下，我们需要用到一些高分辨率的高清图，但是我们一般不建议直接用 Stable Diffusion 来直接生成 `4K` 等高分辨率的图片，因为 Stable Diffusion 模型用到的训练图像的尺寸是 `512` 或 `768` 像素，如果直接生成高分辨率的图片通常会遇到一些问题，比如图像某些内容不合理叠加等等。

不过 Stable Diffusion WebUI 为我们提供了其他方式来生成高分辨率的高清图，我们里就来介绍一下这些方案。




## 1、高清修复几种方案


Stable Diffusion WebUI 在几个不同的地方都提供了对图像进行高清修复的功能：

- 1）文生图（txt2img）中的高清修复（Hires. fix）
- 2）图生图（img2img）中的 SD 放大（SD upscale）
- 3）额外功能（Extras）中的放大（Scale to/Scale by）


不过这里有几点需要注意：

- 方案 1（文生图）的高清修复（Hires. fix）主要是对文本生产图像的结果做分辨率提升。如果你是对已有图像做分辨率提升，那可以使用方案 2（图生图）和 方案 3（额外功能）。
- 上面方案 1（文生图）和 方案 2（图生图）中对图像分辨率提升上限为 `2048x2048`。若需生成 `4K`，可以先在文生图与图生图中生成 `2048x2048`，再使用方案 3（Extra）放大为 `4K`。
- 方案 1（文生图）和方案 2（图生图）中对图像分辨率提升时都会对图像进行重绘，所以要注意 `Denoising strength` 参数不要设置过高，最好不要超过 `0.5`。方案 3（Extra）则没有这个问题，专用于分辨率提升。



### 1.1、文生图（txt2img）中的高清修复（Hires. fix）


文生图（txt2img）中的高清修复（Hires. fix）在`文生图（txt2img）`功能栏页面选中`高清修复（Hires. fix）`来开启。使用方式如图：


![文生图高清修复](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-003.png)


- 1）在 `Prompt` 和 `Negative prompt` 提示词和负向提示词输入区输入提示词和负向提示词；
- 2）选中 `Hires .fix` 开启高清修复功能；
- 3）在 `Upscaler` 下拉框中选择分辨率放大算法；
- 4）通过 `Denoising strength` 参数设置降噪强度，该参数值越小越接近原图，建议不要超过 `0.5`；
- 5）通过 `Upscale by` 参数设置分辨率放大倍数；
- 6）点击 `Generate` 开始生成任务。




这里涉及的一些参数作用，我们在前面《文生图》一节中已经介绍过，这里就不再赘述。






### 1.2、图生图（img2img）中的 SD 放大（SD upscale）


**图生图（img2img）中的 SD 放大（SD upscale）**在`图生图（img2img）`功能栏页面选择 `Script` 下拉框中的 `SD upscale` 来使用。使用方式如图：


![图生图 SD 放大](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-031.png)


- 1）在 `img2img` 功能区导入想要放大的图片；
- 2）通过 `Denoising strength` 参数设置降噪强度，该参数值越小越接近原图，建议不要超过 `0.5`；
- 3）在 `Script` 下拉框中选择的 `SD upscale`；
- 4）通过 `Scale Factor` 参数设置分辨率放大倍数，这里需要注意最终的分辨率不要超过 `2048x2048`；
- 5）在 `Upscaler` 区选择放大算法；
- 6）点击 `Generate` 开始生成任务。


下面是原图以及我们使用上述过程放大后的图：

![原图 512x512](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-030.png)

![放大图 1024x1024](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-032.png)











### 1.3、额外功能（Extras）中的放大（Scale by/Scale to）


**额外功能（Extras）中的放大（Scale by/Scale to）**功能可以在 `Extras` 功能栏页面来通过设置上采样器（Upscaler）来使用。如图：


![额外功能中的放大](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-033.png)


- 1）在 `Single Image` 功能区中导入想要放大的图片；
- 2）通过 `Scale by` 或 `Scale to` 设置想要放大的分辨率倍数或者目标分辨率；
- 3）在 `Upscaler1` 下拉框选择上采样器；
- 4）点击 `Generate` 开始生成任务。

下面是原图以及我们使用上述过程放大后的图：

![原图 512x512](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-030.png)

![放大图 1024x1024](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-034.png)


上面是在额外功能（Extras）中提升图片分辨率最简单的方式，不过这里还有一些其他参数可以在放大分辨率的时候使用：

- `Upscaler2`：选择第二个上采样器，`Upscaler1` 和 `Upscaler2` 两个放大算法将混合使用。
- `Upscaler2 visibility`：设置第二个上采样器的可见度。比如当我们设置改参数值为 `0.3` 时，表示第二个上采样器可见度占比 `0.3`，而第一个占比 `1 - 0.3 = 0.7`。
- `GFPGAN visibility`：`GFPGAN` 模型用于在放大图片时对人脸进行修复。这个参数是设置它的可见度。
- `CodeFormer visibility`：`CodeFormer` 模型也可以用于在放大图片时对人脸进行修复。这个参数是设置它的可见度。
- `CodeFormer weight`：设置 `CodeFormer` 的权重。该参数值取值范围是 0-1，数值越小，效果越强。





## 2、高清修复算法


对图像提升分辨率，最重要的其实是选择合适的图像放大算法。从上面 Stable Diffusion WebUI 的几个图像高清修复的方案中，我们可以发现它们提供的分辨率放大算法选项是大致相同的：

![高清修复的可选算法](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-006.png)








- `Lanczos` 是一种插值算法，它使用一个称为 Lanczos 核的卷积核来进行卷积运算，这个卷积核由一个范围内的 Lanczos 函数计算得到。当放大图像时，它在通过权重计算在原图像的每个像素周围插入新的像素；当缩小图像时，它会从原图像中的每个像素周围的像素中选择一个值来替换这个像素。该算法速度不错，但效果一般。
- `Nearest` 是一种简单的插值算法，它通过缩放系数计算目标图像在原图中的坐标位置，去找到原图中距离该位置最近的像素值作为目标图像当前像素的数值。Nearest 计算速度快，但是可能会产生锯齿，效果一般不好。
- `ESRGAN_4x` 是 `ESRGAN（Enhanced Super-Resolution Generative Adversarial Network）`算法的一种改进版本。ESRGAN 是一种基于生成对抗网络（GAN）的图像超分辨率算法。其主要思想是通过学习低分辨率（LR）图像与其高分辨率（HR）对应物之间的映射，来实现从 LR 图像到 HR 图像的映射过程，从而实现图像的超分辨率。相较于传统的基于插值的超分辨率算法，ESRGAN 可以生成更加清晰、细节更加丰富的高分辨率图像。此外，ESRGAN 生成的图像效果相对锐利。`ESRGAN_4x` 则可以将低分辨率的图像通过神经网络模型增强到 4 倍的分辨率。
- `LDSR` 是 `Latent Diffusion Super Resolution` 的缩写，该算法与 Stable Diffusion 生成图像的原理有些类似，它使用一个经过训练的潜在扩散模型来提升图像分辨率。这个算法效果不错，但是对显存占用很大、速度很慢。
- `R-ESRGAN 4x+` 是 `Real-Time Enhanced Super-Resolution Generative Adversarial Network 4x+` 的缩写，是一种图像超分辨率重建算法。R-ESRGAN 4x+ 基于生成式对抗网络（GAN），是 ESRGAN 的改进版本之一。它通过引入残差连接和递归结构，改进了 ESRGAN 的生成器网络，并使用 GAN 进行训练。R-ESRGAN 4x+ 在提高图像分辨率的同时，也可以增强图像的细节和纹理，并且生成的图像质量比传统方法更高。它在许多图像增强任务中都取得了很好的效果，比如图像超分辨率、图像去模糊和图像去噪等。
- `R-ESRGAN 4x+ Anime6B` 是 `R-ESRGAN 4x+` 的一个衍生版本，它基于 `R-ESRGAN 4x+` 算法并使用了 Anime6B 数据集进行训练。Anime6B 数据集是一个专门用于动漫图像处理的数据集，其中包含了大量不同风格、不同质量的动漫图像，使得算法可以适应不同类型的动漫图像。R-ESRGAN 4x+ Anime6B 算法在动漫图像增强领域具有较高的准确性和效果，并且可以应用于不同类型的动漫图像处理，如动画制作、漫画制作等。
- `ScuNET GAN` 也叫 `Swin-Conv-UNet GAN`，是一个去除图像中噪声同时保留原始细节的神经网络模型，对于去除图像中的噪点有比较好的效果。
- `ScuNET PSNR` 类似 `ScuNET GAN`，适用于保持更多的图像细节、纹理、颜色等信息的处理场景。 
- `SwinIR 4x` 是一种基于 Swin Transformer 的图像超分辨率重建算法，可将低分辨率图像放大 4 倍，生成高分辨率图像。Swin Transformer 是一种新型的 Transformer 模型，相对于传统的 Transformer 模型，在处理图像等二维数据时，具有更好的并行性和更高的计算效率。SwinIR 4x 通过引入 Swin Transformer 和局部自适应模块（LAM）来提高图像重建的质量和速度。其中，局部自适应模块用于提高图像的局部细节，从而增强图像的真实感和清晰度。SwinIR 4x 被广泛应用于计算机视觉领域，特别是图像重建、图像增强和图像超分辨率等方面。







这些算法各自有一些适用场景：

- `Nearest` 和 `Lanczos`：一般效果不太好，不太常用。
- `ESRGAN_4x`：对于照片，效果不错，不过可能出现细节较锐利的效果，但有些人喜欢这样的风格；对于绘画，效果有些粗糙，不过可能适合有纹理的油漆风格；对于二次元漫画，效果比较差。
- `LDSR`：对于照片，效果很不错，但是速度太慢；对于绘画，可能会有一些随机噪点；对于二次元漫画，也可能出现一点噪点。
- `R-ESRGAN 4x+`：对于照片、绘画、二次元漫画效果都还不错，均衡型选择。
- `R-ESRGAN 4x+ Anime6B`：对于照片、绘画的处理都带上了些二次元漫画的风格；对于二次元漫画，效果很好。
- `ScuNET GAN`：对于照片，可以去除噪点，但是可能会糊；对于绘画，可能会糊；对于二次元漫画，效果还可以。
- `ScuNET PSNR`：对于照片，可以去除噪点，但是可能会糊；对于绘画，可能会糊；对于二次元漫画，效果比较差。
- `SwinIR_4`：对于照片，效果一般；对于绘画，效果还不错；对于二次元漫画，效果比较差。




综上，我们通常记住这条建议就好：**一般情况，我们使用 `R-ESRGAN 4x+` 即可；对于二次元漫画，我们使用 `R-ESRGAN 4x+ Anime6B`。**




下面我们就用下面一幅 `512x512` 的图像分别使用这些算法放大到 `2048x2048` 的效果：

1）示例

![原图 512x512](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-040.png)

![Lanczos 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-041.png)

![Nearest 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-042.png)

![ESRGAN_4x 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-043.png)

![LDSR 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-044.png)

![R-ESRGAN 4x+ 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-045.png)

![R-ESRGAN 4x+ Anime6B 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-046.png)

![ScuNET GAN 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-047.png)

![ScuNET PSNR 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-048.png)

![SwinIR 4x 2048x2048](assets/resource/aigc-tutorial/sd-upscale/sd-upscale-049.png)





