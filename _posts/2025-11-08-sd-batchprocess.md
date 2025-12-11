---
title: AI 教程：批量生成图像
description: Stable Diffusion WebUI 提供了多种批量生成图像的功能，可以帮助我们更加高效地完成大量的绘图任务。
author: Keyframe
date: 2025-11-08 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-013.png
  alt: Stable Diffusion 批量生成图像
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



我们经常会遇到一些需要生成或处理多张图像的需求，这时候如果只能一张一张来处理未免效率太低，对于这种情况，Stable Diffusion WebUI 提供了多种批量生成图像的功能，可以帮助我们更加高效地完成大量的绘图任务。




## 1、文生图中的批量处理功能

在文生图（txt2img）中我们可以用到的批量处理功能有下面几种：

- 1）使用 `Batch count` 批处理
- 2）使用 `Batch size` 批处理
- 3）使用 `X/Y/Z plot` 批处理
- 4）使用 `Prompt matrix` 批处理
- 5）使用 `Prompts from file or textbox` 批处理






### 1.1、使用 Batch count 批处理


`Batch count` 表示当前生成图像的任务要循环执行的次数。在多次生成任务中，每次使用的提示词和参数都相同，但是 Seed 值会依次递增来保证每次任务生成的结果不重复。



在下面的示例中，我们使用提示词：

```
a young girl with brown eyes, wearing a white outfit, sitting outside cafe, side light, full body shot, by Vincent van Gogh
```

负向提示词：

```
worst quality, low quality, grayscale, monochrome, missing arms, extra legs, fused fingers, too many fingers, unclear eyes
```

并设置 `Batch count` 为 `8` 来批量生成图像，结果如下图：


![使用 Batch count 批处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-001.png)





### 1.2、使用 Batch size 批处理


`Batch size` 表示的是当次生成任务要生成几张图片。

上面介绍的 Batch count 是通过将任务运行多次的方式来实现，是以时间换空间。Batch size 则是通过在一次任务里生成多张图来实现，这样会需要更多显卡内存和计算资源，不过耗时会更短，是以空间换时间。

在下面的示例中，我们使用提示词：

```
a young girl with brown eyes, wearing a white outfit, sitting outside cafe, side light, full body shot, by Vincent van Gogh
```

负向提示词：

```
worst quality, low quality, grayscale, monochrome, missing arms, extra legs, fused fingers, too many fingers, unclear eyes
```

并设置 `Batch size` 为 `8` 来批量生成图像，结果如下图：


![使用 Batch size 批处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-002.png)










### 1.3、使用 `X/Y/Z plot` 批处理


`X/Y/Z plot` 功能在 `Script` 下拉栏中可以选择使用，这个功能可以最多选择给 3 个维度（X、Y、Z）的参数各自设置一组值，并遍历生成对应的图像。这个功能对于测试参数效果非常有用。


我们下面通过几个示例来介绍这个功能。





**示例 1：批量遍历采样步数（Steps）**

例如，对于采样步数 `Sampling Steps` 这个参数，默认值是 20，指越大出图效果越好但出图速度越慢，那么设置多少为最优呢？这时候我们可以在其它参数一致的情况下测试不同 `Sampling Steps` 参数值的效果来做出抉择。使用 `X/Y/Z plot` 功能就可以帮助我们批量完成不同参数值的测试，流程如下图所示：


![使用 X/Y/Z plot 批处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-003.png)

- 1）在提示词输入区输入提示词和负向提示词；
- 2）在 `Script` 下拉框选择 `X/Y/Z plot` 功能，此后会在下面出现设置 `X/Y/Z plot` 细分参数的操作区；
- 3）在 `X type` 下拉框选择采样步数 `Steps`；在 `X values` 输入框输入 `5,10,20,30,40`，表示对采样步数遍历使用这些数值分别生成图像；
- 4）点击 `Generate` 按钮启动生成任务；
- 5）等待生成结果。

按上面的示例得到的生成结果如下图所示，可以发现采样步数 `Steps` 超过 30 以后就变化不大了，所以可以得到结论：采样步数设置为 30 以下即可。

![采样步数（Steps）遍历](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-004.png)



在上面第 3 步中设置 `X values` 时，如果 `X values` 的取值类型是数值类时，这里有一些便捷的语法可以使用：

- 语法 `5,10,20,30,40`：如我们在上面示例中使用，以逗号分开的取值列表，批量任务会依次遍历这些数值作为参数值来生成图像。
- 语法 `1-5`：表示的取值集合等同于 `1,2,3,4,5`。
- 语法 `1-5(+2)`：表示在 1-5 的范围内以步进 2 来取值，取值集合等同于 `1,3,5`。
- 语法 `1-10[5]`：表示在 1-10 的范围内按尽量相等的间隔取 5 个值，取值集合等同于 `1,3,5,7,10`

当然，如果 `X values` 的取值类型是字符串或者其他枚举值，就直接排列即可。




**示例 2：批量遍历采样器（Sampler）**

比如，当我们使用 `X/Y/Z plot` 批量遍历不同`采样器（Sampler）`的效果时，我们就可以如下图来测试：

![采样器（Sampler）遍历](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-005.png)

和上面遍历采样步数的示例不同的是：我们在 `X type` 下拉框选择采样器 `Sampler`；这时 `X values` 变成了下拉框来给我们选择枚举值，我们选择了 `Euler a`、`DPM++ 2S a`、`DPM fast`，表示分别遍历这 3 个采样器来生成图像。

最后的生成结果如下图所示：

![采样器（Sampler）遍历结果](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-006.png)





**示例 3：批量遍历提示词**

除了遍历不同的参数外，我们还可以批量遍历不同的提示词来测试对应的生成效果。这时候就需要使用到 `Promt S/R` 功能了。

`Promt S/R` 表示对提示词中的一些词进行查找和替换，比如下面的提示词：

```
a young girl with brown eyes, wearing a white outfit, sitting outside cafe, side light, full body shot, (by Vincent van Gogh:0.6)
```

我们想对 `(by Vincent van Gogh:0.6)` 分别替换为 `(by Vincent van Gogh:1.0)`、`(by Vincent van Gogh:1.2)` 看看生成效果。

这时候，如下图所示：

![提示词遍历](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-012.png)

我们在 `X type` 下拉框选择采样器 `Promt S/R`；然后在 `X values` 输入框中输入 `(by Vincent van Gogh:0.6),(by Vincent van Gogh:1.0),(by Vincent van Gogh:1.2)`，这样 `X/Y/Z plot` 就会在提示词中查找 `(by Vincent van Gogh:0.6)`，并分别替换为 `(by Vincent van Gogh:0.6)`、`(by Vincent van Gogh:1.0)`、`(by Vincent van Gogh:1.2)` 来生成结果。


最后我们得到的结果如下图，这样可以看到我们对提示词设置不同权重时对结果的影响：

![提示词遍历结果](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-013.png)




**示例 4：批量二维遍历**

上面的几个示例中我们都是对单一维度的变量做了批量遍历，下面我们增加一个维度来看看效果。

这里我们将 X 轴设置为`采样器（Sampler）`，Y 轴设置为 `Prompt S/R`。

![采样器（Sampler）、Prompt S/R 二维遍历](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-014.png)

结果如下：


![采样器（Sampler）、Prompt S/R 二维遍历结果](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-015.png)



**示例 5：批量三维遍历**

`X/Y/Z plot` 最多支持 3 个维度，现在我们来示例遍历三个维度变量的情况。

我们将 X 轴设置为`采样器（Sampler）`，Y 轴设置为 `Prompt S/R`，Z 轴设置为`采样步数（Steps）`。

![采样器（Sampler）、Prompt S/R、采样步数（Steps）三维遍历](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-016.png)

结果如下：

![采样器（Sampler）、Prompt S/R、采样步数（Steps）三维遍历结果](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-017.png)






**其他参数**

`X/Y/Z plot` 功能除了上面介绍的设置几个维度的变量及值外，还有几个参数可以设置：


![X/Y/Z plot 的参数](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-011.png)


- `Draw legend`：绘制轴和类型。
- `Keep -1 for seeds`：保持随机种子为 -1，从而每张图都随机。
- `Include Sub Images`：预览子图像，生成结果不仅仅包含对比图，还包括每个子图像。
- `Include Sub Grids`：预览子宫格图，带 Z 轴情况下会拆分子图。
- `Grid margins (px)`：宫格图边框，设置对比图的边框间隔，数值越大间隔越大。














### 1.4、使用 Prompt matrix 批处理


`Prompt matrix` 可以叫做**提示词矩阵**，该功能也是在 `Script` 下拉栏中选择使用，它可以组合提示词中不同参数，可以快速验证不同参数组合的所有效果。


`Prompt matrix` 的语法是在各个提示词之间使用 `|` 进行分割，竖线前的提示词为固定内容，竖线后的提示词会被组合遍历。


下面是我们使用 `Prompt matrix` 语法修改过的提示词，其中通过 `|` 分割了最后 2 个词 `full body shot` 和 `by Vincent van Gogh`：

```
a young girl with brown eyes, wearing a white outfit, sitting outside cafe, side light | full body shot | by Vincent van Gogh
```

我们来用这个提示词示例一下 `Prompt matrix` 的使用流程：


![Prompt matrix 使用流程](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-007.png)


- 1）在提示词输入区输入使用了 `Prompt matrix` 语法的提示词；
- 2）在 `Script` 下拉框选择 `Prompt matrix`，开启提示词矩阵功能，此后会在下面出现设置 `Prompt matrix` 细分参数的操作区；
- 3）点击 `Generate` 按钮启动生成任务；
- 4）等待生成结果。


最终的生成结果如下图所示，它展现提示词中 `full body shot` 组合 `by Vincent van Gogh` 的 4 种效果，通过这个功能可以便于我们测试和写出适合的提示词。

![Prompt matrix 生成结果](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-008.png)


开启 `Prompt matrix` 后，可以设置相关的细分参数，如下图：

![Prompt matrix 参数](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-010.png)


- `Put variable parts at start of prompt`：表示把可变部分放在提示词文本的开头。基于提示词越靠前权重越高的规则，如果需要让可变提示词提高权重可以勾选该参数。
- `Use different seed for each picture`：为每张图片使用不同的种子。默认使用相同的种子，勾选该参数则会在遍历生成图像时使用不同的种子。。
- `Select prompt`：选择 `Prompt matrix` 的应用目标。指定 `Prompt matrix` 的应用目标是正面提示词还是负向提示词。
- `Select joining char`：选择分割符。选择可变提示词是通过逗号还是空格进行连接。
- `Grid margins (px)`：宫格图边框，设置对比图的边框间隔，数值越大间隔越大。











### 1.5、使用 Prompts from file or textbox 批处理

`Prompts from file or textbox` 功能也是在 `Script` 下拉栏中选择使用，这个功能是使用文本指令的方式来执行生成图像的任务，它支持将生成图像任务需要的所有参数都通过文本输入给 Stable Diffusion，并且支持一次性输入多条文本指令，从而能够批量执行多条生成任务。

使用 `Prompts from file or textbox` 批量执行图像生成任务的流程如图：

![使用 Prompts from file or textbox 批处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-009.png)


- 1）在 `Script` 下拉框选择 `Prompts from file or textbox`，开启批量生成任务功能，此后会在下面出现设置细分参数的操作区；
- 2）在 `List of prompt inputs` 输入框中输入提示词指令列表，一行一条，一条提示词指令对应一次生成任务；
- 3）点击 `Generate` 按钮启动批量生成任务；
- 4）等待生成结果。


我们在上面示例中使用的一条文本提示词指令示例如下：

```
--prompt "a young girl with brown eyes, wearing a white outfit, sitting outside cafe, side light, full body shot, by Vincent van Gogh"  --negative_prompt "worst quality, low quality, grayscale, monochrome, missing arms, extra legs, fused fingers, too many fingers, unclear eyes" --steps 20 --sampler_name "Euler a"  --cfg_scale 7 --seed 951816086 --width 512 --height 512 --batch_size 1
```

我们以此为例来介绍一下提示词指令语法：

- 提示词指令中的参数格式：`--[参数名] [参数值]`。
- `[参数名]` 常用的有：
	- `prompt`：提示词 
	- `negative_prompt`：负向提示词
	- `seed`：种子
	- `width`：宽度
	- `height`：高度
	- `cfg_scale`：提示词相关性
	- `sampler_name`：采样器名称
	- `steps`：采样步数
	- `batch_count`：生成图片次数
	- `batch_size`：一次生成图片数量
	- `restore_face`：人脸修复
- `[参数值]` 包括`数值`、`字符串`、`布尔值`几种类型。
	- 数值类型，直接跟在参数名后，空格间隔。比如：`--steps 20`。
	- 字符串类型，用 `"` 引用跟在参数名后，空格间隔。比如：`--sampler_name "Euler a"`。
	- 布尔值类型，值为 `true` 或 `false`，跟在参数名后，空格间隔。比如：`--restore_face true`。
















## 2、图生图中的批量处理功能



除了文生图（txt2img）中的批量处理功能，图生图（img2img）中也有一些批量处理功能可供我们使用：


- 1）使用 `Batch count` 批处理
- 2）使用 `Batch size` 批处理
- 3）使用 `X/Y/Z plot` 批处理
- 4）使用 `Prompt matrix` 批处理
- 5）使用 `Prompts from file or textbox` 批处理
- 6）使用 `Batch` 批处理


其中前 5 种批处理功能与我们在前面文生图（txt2img）中介绍的基本一致，这里就不再赘述了。这里我们主要介绍一下第 6 种批处理功能。


### 2.1、使用 `Batch` 批处理


`Batch` 批处理功能是在图生图（img2img）页面的子栏目 `Batch` 面板中使用，它可以指定将一个目录下（Input director）的所有图片依次作为图生图任务的引导图来生成对应的新图，并将新图存储到指定的输出目录（Output directory）。

使用 `Batch` 批处理功能的流程如下图所示：


![使用 Batch 批处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-018.png)

- 1）选择 `Batch` 子栏目面板；
- 2）在 `Input directory` 输入框设置引导图集合所在的目录；
- 3）在 `Output directory` 输入框设置新图输出的目录；
- 4）设置图生图相关的其他参数；
- 5）点击 `Genrate` 按钮开始批处理任务。



这里的 `Input directory` 和 `Output directory` 对应的路径可以是相对路径，也可以是绝对路径。如果是相对路径，是相对于 Stable Diffusion WebUI 的工作目录。如果是绝对路径，需要确保路径正确，且 Stable Diffusion 对该路径有读写权限。







## 3、Extra 中的批量处理功能




Extra 中的也有提供一些批量处理功能来支持一次处理多张图片，包括：

- 1）使用 `Batch Process` 批量处理
- 2）使用 `Batch from Directory` 批量处理


### 3.1、使用 Batch Process 批量处理


`Batch Process` 批量处理功能支持一次性上传多张图片进行批量处理，使用流程如下图：

![Batch Process 批量处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-021.png)

- 1）选择 `Batch Process` 子栏目面板；
- 2）在图像输入面板上传待处理的图片列表；
- 3）设置图片放大的相关参数；
- 4）点击 `Generate` 启动生成任务；
- 5）等待生成结果。




### 3.2、使用 Batch from Directory 批量处理

`Batch from Directory` 批量处理功能支持通过对指定文件夹的所有图片进行批量处理，使用流程如图：


![Batch from Directory 批量处理](assets/resource/aigc-tutorial/sd-batchprocess/sd-batchprocess-022.png)


- 1）选择 `Batch Process` 子栏目面板；
- 2）在 `Input directory` 输入框设置待处理图片所在的目录；
- 3）在 `Output directory` 输入框设置新图输出的目录；
- 4）设置图像放大的相关参数；
- 5）点击 `Generate` 启动生成任务。
- 6）等待生成结果。


这里的 `Input directory` 和 `Output directory` 对应的路径可以是相对路径，也可以是绝对路径。如果是相对路径，是相对于 Stable Diffusion WebUI 的工作目录。如果是绝对路径，需要确保路径正确，且 Stable Diffusion 对该路径有读写权限。

