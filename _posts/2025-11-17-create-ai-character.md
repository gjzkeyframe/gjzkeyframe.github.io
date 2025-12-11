---
title: AI 教程：生成 AI 模特
description: 这一篇我们来介绍如何基于 Stable Diffusion WebUI 的各种功能来生成 AI 模特。
author: Keyframe
date: 2025-11-17 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-001.png
  alt: Stable Diffusion 生成 AI 模特
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



我们在前面的章节里介绍了 Stable Diffusion WebUI 的各种基础功能和高级技巧，现在我们开始做一些特定需求的实践。

这一篇我们来介绍如何基于 Stable Diffusion WebUI 的各种功能来生成 AI 模特。



## 1、复刻 AI 模特

当我们想要通过 Stable Diffusion WebUI 创作一位 AI 模特时，一种方法是搭配模型、编写提示词、调整生成参数，然后交给 Stable Diffusion 去生成，这个过程有点像开盲盒，因为你不知道每次生成的精确结果会是什么样。另一种办法是把在网上看到的满意的 AI 模特复刻出来，在其基础上进行微调。我们这里来讲讲第二种方法，比如，下图是我看到觉得满意的一张模特图，我们现在来把她复刻出来。


![原图](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-001.png)


复刻 AI 模特的办法就是使用前面介绍过的 `PNG Info` 功能。不过需要注意，使用 PNG Info 的前提是图片中需要包含 AI 生产任务的提示词及参数信息，所以在使用 PNG Info 复刻模特时要使用该图的原图，经过微信等其他工具传输后的图，都可能被平台转码而导致图像的信息被清理，而无法使用 PNG Info 提取出提示词。


你可以微信扫描下面二维码关注公众号『关键帧Keyframe』后，发送消息『AI 模特』来获得一些 AI 模特原图。

![公众号：关键帧Keyframe](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-000.jpg)


获得原图后，复刻 AI 模特的流程如下：

**1）使用 WebUI 的 PNG Info 获取原图提示词和参数**

过程如下图：


![使用 PNG Info 提取提示词和参数](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-002.png)

- 1）在 WebUI 界面点击 `PNG Info` 栏；
- 2）将原图导入左边图像区；
- 3）然后等待右边提示词区抽取出原图的提示词和参数。
- 4）点击 `Send to txt2img`；


在上面提取的提示词和参数信息如下：

```
masterpiece, best quality, (realistic, photo-realistic:1.4), (RAW photo:1.2), extremely detailed CG unity 8k wallpaper, an extremely delicate and beautiful, amazing, finely detail, official art, absurdres, incredibly absurdres, huge filesize, ultra-detailed, extremely detailed, beautiful detailed girl, (perfect female figure), extremely detailed eyes and face, beautiful detailed eyes, (wearing school uniform and grey plaid skirt), collared shirt, (soft light), light on face, looking at viewer, slim waist, (full body), (small breasts), standing, midriff, bangs, <lora:koreanDollLikeness_v15:0.2>, pureerosface_v1:0.8
Negative prompt: (worst quality:2), (low quality:2), (normal quality:2), lowres, ((monochrome)), ((grayscale)), paintings, sketches, nipples, skin spots, acnes, skin blemishes, bad anatomy, facing away, looking away, tilted head, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, blurry, bad feet, cropped, poorly drawn hands, poorly drawn face, mutation, deformed, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, extra fingers, fewer digits, extra limbs, extra arms, extra legs, malformed limbs, fused fingers, too many fingers, long neck, cross-eyed, mutated hands, polar lowres, bad body, bad proportions, gross, nipples, ng_deepnegative_v1_75t, badhandv4
Steps: 20, Sampler: DPM++ 2M Karras, CFG scale: 7, Seed: 365013168, Size: 512x768, Model: Chilloutmix-Ni-pruned-fp32-fix, Version: v1.3.1
```


其中，有两处需要注意的地方：

- 参数中的 `Model: Chilloutmix-Ni-pruned-fp32-fix`：表示使用了 `Chilloutmix-Ni-pruned-fp32-fix` 主模型。
- 提示词中的 `<lora:koreanDollLikeness_v15:0.2>`：表示使用了 `koreanDollLikeness_v15` 这个 LoRA 模型；


**2）配置相关模型**


在上面点击 `Send to txt2img` 后，WebUI 会切换到`图生图（txt2img）`栏目，并自动将提示词和参数填充好。这里我们需要检查参数中用到的主模型和 LoRA 模型是否已经配置到位。如图：


![配置相关模型](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-004.png)


- 1）检查主模型是否为参数中对应的 `Chilloutmix-Ni-pruned-fp32-fix`；
- 2）点击 `Show/hide extra networks` 按钮展开额外网络面板，在面板中选择 `Lora` 子栏目，检查其中是否已经配置 `koreanDollLikeness_v15` 这个 LoRA 模型。

如果上述模型没有配置，则可以下载相关模型并按照之前《配置 Stable Diffusion 相关模型》一文中介绍的方法配置好对应的模型。其中 `Chilloutmix-Ni-pruned-fp32-fix` 主模型放在 `stable-diffusion-webui/models/Stable-diffusion` 目录下，`koreanDollLikeness_v15` LoRA 模型放在 `stable-diffusion-webui/models/Lora` 目录下。



**3）复刻生成**

接下来，开始复刻，如图：


![复刻生成](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-005.png)


- 1）点击 `Generate` 按钮，开始启动 AI 画图；
- 2）等待生成结果。


可以看到，我们新生成的图跟原图完全一样，这样我们就完成了 AI 模特的复刻。





为什么我们可以原封不动的复刻原图呢？

用 AI 模型生成图的过程其实是一个幂等的过程，也就是对于一个指定的模型，当我们设置的所有绘画参数都一样时，模型生成出来的图就是一样的。

大家按照我们上面的流程操作，有没有可能生成出不一样的图呢？

也是有可能的，原因还是在于出现了和原参数不一样的变动，比如：使用的模型可能不是同一个版本；某些参数受 WebUI 版本的影响没有生效或发生了变化等等。


复刻一张原图有什么意义呢？

复刻一张图其实是一个开始。当我们有一个 AI 模特图库，我们可以选择一张我们喜欢的图（包括颜值、身姿、穿搭、背景），我们把这张图复刻出来后，可以在这张图的基础上对提示词或其他参数做微调，从而可以生成与原图近似的新图，这样就能生成人设和风格比较一致的 AI 模特套图，这样具有一致性的内容在我们现实生产中有很多用途。






## 2、调试 AI 模特人脸

当然，如果你没有从网上找到喜欢的模特，或者由于图片中不包含 AI 生成任务的提示词和参数信息导致无法复刻，又或者你想在复刻的模特的基础上继续调试出新的人脸，那我们可以试试下面这些方法：

- 修改提示词中人脸五官相关的描述调试人脸
- 随机 Seed 调试人脸
- 调整 LoRA 模型权重调试人脸
- 组合 LoRA 调试人脸


下面是几个示例：

**1）修改提示词中人脸五官相关的描述调试人脸**

下面是一些供参考的人脸五官的描述词，大家可以按需使用：

![人脸五官提示词](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-100.png)




**2）随机 Seed 调试人脸**

使用随机 Seed 生成新的人脸，其实就是开盲盒，把生成结果交给模型。

方法很简单：找到`随机种子（Seed）输入栏`，后面点击`骰子（🎲）按钮`，可以发现随机种子被设置为 `-1`，然后不用改变提示词和参数，直接点 `Generate` 按钮开始生成即可，不满意就继续生成下一个。

随机种子（Seed）为 `-1` 表示：每次都使用一个新的随机数。由于 Seed 值对图的变化影响很大，所以这样就可以生成新的人脸了。



**3）调整 LoRA 模型权重调试人脸**

我们在前面复刻的 AI 模特的基础上来调整，原 AI 模特用到的 LoRA 信息是 `<lora:koreanDollLikeness_v15:0.2>`，对应的权重是 `0.2`。这里我们将权重调整到 `0.4`，即 `<lora:koreanDollLikeness_v15:0.4>`。

此外，我们将随机种子（Seed）设置为 `-1` 来增加随机性。

如图：

![调整 LoRA 模型权重调试人脸](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-006.png)

下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-007.png)


**4）组合 LoRA 调试人脸**

我们还是在前面复刻的 AI 模特的基础上来调整，这次我们组合了 `koreanDollLikeness_v15` 和 `japaneseDollLikeness_v15` 这两个 LoRA 模型，将原来的提示词中的 `<lora:koreanDollLikeness_v15:0.2>` 替换为 `<lora:koreanDollLikeness_v15:0.2>, <lora:japaneseDollLikeness_v15:0.3>`。

同样的，我们将随机种子（Seed）设置为 `-1` 来增加随机性。

如图：

![组合 LoRA 调试人脸](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-008.png)

下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-009.png)









## 3、给 AI 模特换装

当我们通过上面的步骤确定了 AI 模特的人脸后，我们还可以给模特换装生成不同装扮的新图。

给 AI 模特换装扮有几种方式：

- 通过提示词给 AI 模特换装
- 通过装扮 LoRA 给 AI 模特换装
- 通过 Inpaint 给 AI 模特换装

下面是几个示例：

**1）通过提示词给 AI 模特换装**

我们还是在前面复刻的 AI 模特的基础上来调整，我们将原提示词中描述模特服装的部分 `(wearing school uniform and grey plaid skirt), collared shirt` 更换为 `(wearing white shirt and leather skirt:1.2)` 来给模特换装，如图：

![提示词给 AI 模特换装](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-010.png)

下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-011.png)



还有一些描述模特装扮的提示词，大家可以参考使用：

![人物装饰提示词](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-101.png)

![人物服装提示词](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-102.png)

![人物鞋袜提示词](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-103.png)


**2）通过装扮 LoRA 给 AI 模特换装**

我们还可以通过一些专门影响装扮的 LoRA 模型来给 AI 模特换装。比如我们这里要用到的汉服唐风 `hanfuTang_v35` 这个 LoRA 模型。

我们还是在前面复刻的 AI 模特的基础上来调整。我们在 Stable Diffusion WebUI 中配置 `hanfuTang_v35` 模型，并在提示词中增加 `<lora:hanfuTang_v35:0.6>`，然后把原提示词中描述模特服装的部分 `(wearing school uniform and grey plaid skirt), collared shirt` 更换为 `(hanfu, tang style outfits, orange upper shan, green chest pleated skirt, red with green waistband)` 来给模特换装。

如图：

![装扮 LoRA 给 AI 模特换装](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-012.png)


下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-013.png)

装扮已经换为汉服了，细节还有一些优化空间。



**3）通过 Inpaint 给 AI 模特换装**

我们在《图像局部重绘》一篇中介绍的 `Inpaint` 功能也可以用来给模特换装。

我们继续在前面复刻的 AI 模特的基础上来尝试。我们 `PNG Info` 功能页面可以直接将提示词及参数信息同步到 `Inpaint` 功能模块中去，如图：

![从 PNG Info 到 Inpaint](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-020.png)

切到图生图的 Inpaint 功能后，我们通过如图流程给模特换装：

![Inpaint 给 AI 模特换装](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-022.png)

- 1）在模特的裙子上涂上蒙版层，对这个部分进行重绘；
- 2）将提示词中的 `(wearing school uniform and grey plaid skirt), collared shirt` 更换为 `(wearing school uniform and black leggings), collared shirt` 来引导重绘；
- 3）点击 `Generate` 按钮开始生成；
- 4）等待生成结果。


下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-023.png)

用 Inpaint 给 AI 模特换装可以实现按需局部换装。




## 4、控制 AI 模特姿势

控制 AI 模特的姿势是非常常见的需求，这里使用 ControlNet 是一个很好的选择。我们可以基于一些已有图片来用 ControlNet 提取任务姿势再引导 AI 生成对应的模特图。


我们继续在前面复刻的 AI 模特的基础上来尝试。在使用 PNG Info 的 `Send to txt2img` 切到图生图功能页面后，我们按照下图流程来使用 ControlNet 控制模特姿势：


![控制 AI 模特姿势](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-024.png)

- 1）打开 ControlNet 面板，导入预期姿势引导图；
- 2）选中 `Enable`，开启 ControlNet；
- 3）选择 ControlNet 预处理器和模型，我们这里选择了 `openpose` 预处理器只控制身体骨骼姿势；
- 4）点击 `Generate` 按钮开始生成；
- 5）等待生成结果。


下面是生成的结果：

![结果](assets/resource/aigc-tutorial/sd-create-ai-character/sd-create-ai-character-025.png)

可以看到生成的新图姿势保持了与引导图一致。


