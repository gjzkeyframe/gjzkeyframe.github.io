---
title: AI 教程：使用 ControlNet 精细控制生成结果
description: ControlNet 是一种通过添加额外条件来控制扩散模型的神经网络结构。
author: Keyframe
date: 2025-11-14 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion, ControlNet]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-006.png
  alt: Stable Diffusion ControlNet
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


ControlNet 是一种通过添加额外条件来控制扩散模型的神经网络结构。它提供了一种增强稳定扩散的方法，在文本到图像生成过程中使用条件输入（如边缘映射、姿势识别等），可以让生成的图像将更接近输入图像，这比传统的图像到图像生成方法有了很大的改进。

在 Stable Diffusion 的基础上使用 ControlNet 就相当于给 Stable Diffusion 加了一个插件用于引导 AI 模型按照输入的条件来生成图，从而实现更精细的生成控制。

要在 Stable Diffusion WebUI 中使用 ControlNet，需要先安装这个扩展，安装步骤如下：

- 1）打开 `Extensions` 栏。
- 2）打开 `Extensions` 栏下的 `Install from URL` 栏。
- 3）在 `URL for extension's git repository` 下的输入框中输入 `https://github.com/Mikubill/sd-webui-controlnet.git`。
- 4）点击 `Install` 按钮。
- 5）等待数秒，你应该会收到 `Installed into stable-diffusion-webui/extensions/sd-webui-controlnet. Use Installed tab to restart` 的消息。
- 6）打开 `Extensions` 栏下的 `Installed` 栏，点击 `Check for updates` 按钮，然后点击 `Apply and restart UI` 按钮。后面都可以用这种方式来更新 ControlNet 等扩展。
- 7）重启 Stable Diffusion WebUI。
- 8）从 `https://huggingface.co/lllyasviel/ControlNet-v1-1/tree/main` 下载后缀名为 `.pth` 的 ControlNet 的模型文件，并把它们放到 `stable-diffusion-webui/extensions/sd-webui-controlnet/models` 文件夹。
- 9）在 `txt2img` 栏下的 ControlNet 面板中，点击 `Model` 下拉框右边的刷新按钮，然后你就能在下拉框中看到模型了。





ControlNet 的能力有很多中，我们下面以 ControlNet V1.1.233 版本为例来分类一一介绍。



## 1、简稿控图


### 1.1、Canny 模型：轮廓线稿控图

通过 `Canny 模型`可以对原始图片进行边缘检测，识别图像内对象的边缘轮廓，从而生成原始图片对应的线稿图。接着，再基于线稿图和提示词来生成具有同样线稿结构的新图，这样就实现了对新图的控制。

接下来，我们来介绍一下如何在 Stable Diffusion WebUI 中通过 ControlNet 扩展来使用这个模型：


**第一步：我们在 WebUI 页面中找到 ControlNet 面板，用引导图来生成轮廓线稿图**

整个过程如图所示：

![Canny 模型生成线稿图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-002.png)

- 1）在 WebUI 页面中找到 ControlNet 面板，点击右上角的三角按钮打开面板；
- 2）在 ControlNet 面板的`引导图输入区`导入引导图，引导图对应着我们想要的轮廓；
- 3）在多选框列表中勾选 `Enable` 选择框以在后续绘图任务中启用 ControlNet；勾选 `Allow Preview` 选择框以开启预处理图的预览；
- 4）在 `Control Type` 单选区选择一种控制类型后，WebUI 将为我们匹配对应的预处理器（Preprocessor）和 ControlNet 模型（Model），我们这里选择 `Canny` 来确定使用 Canny 预处理器和模型进行后续的预处理和控制生成任务；
- 5）`预处理器（Preprocessor）`下拉框已经自动匹配了 canny 预处理器，但是这里可能会有其他适用的 canny 预处理器可供选择，可以点开下拉框选择即可；
- 6）`ControlNet 模型（Model）`下拉框已经自动匹配了 canny 模型，这里也可能会有其他类型的 canny 模型可供选择，可以点开下拉框选择即可；
- 7）在下面的参数设置区设置其他参数，我们这里通常用默认值即可；
- 8）点击 `💥` 按钮，启动预处理任务；
- 9）在`预处理预览（Preprocessor Preview）`区等待轮廓线稿图生成完成。




在上面第 7 步中可以设置的参数有这些：

- `Control Weight`：使用 ControlNet 生成图片的权重占比影响（多个 ControlNet 组合使用时，需要调整权重占比）。
- `Starting Control Step`：在生成任务的第几步采样中开始使用 ControlNet。
- `Ending Control Step`：在生成任务的第几步采样中停止使用 ControlNet。
- `Preprocessor Resolution`：预处理器分辨率，默认 512，数值越高线条越精细，数值越低线条越粗糙。
- `Canny Low Threshold`：该数值越高，生成的线稿细节越少，线稿越简单。
- `Canny High Threshold`：该数值应该高于 `Canny Low Threshold`。该数值越低，生成的线稿图细节越多，线稿越复杂。
- `Control Mode`：控制模式。`Balanced` 表示平衡提示词和 ControlNet 对结果的影响；`My prompt is more important` 表示设置提示词影响更大；`ControlNet is more important` 表示 ControlNet 影响更大。
- `Resize Mode`：当预处理线稿图跟生成任务的目标分辨率不一样时采用的裁剪模式，默认使用 `Crop and Resize`。


下面是我们使用的引导图和 ControlNet 生成的轮廓线稿图：

![引导图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-001.png)

![Canny 线稿图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-003.png)


**第二步：在轮廓线稿图的基础上，使用文生图生成同样姿势的新图**

整个过程如图所示：

![文生图 + Canny 生成新图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-005.png)


- 1）选择主模型，这里我们选了 `CripsMix_v1.0` 模型来生成与原图 3D 风格不同的清新插画风格图像；
- 2）选择 VAE 模型，这里主要是 `CripsMix_v1.0` 主模型需要配置 VAE 模型来提升图像颜色饱和度；
- 3）输入提示词，这里我们指定了要生成 `black hair` 等特征；
- 4）设置其他参数；
- 5）点击 `Generate` 启动生成任务；
- 6）等待生成结果。

最终的生成结果在结构上保持了和线稿图的一致性，同时又接受了提示词的引导。结果图如下：

![Canny 新图结果](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-006.png)




### 1.2、SoftEdge 模型：柔和线稿控图

通过 `SoftEdge 模型`也可以对原始图片进行边缘检测，从而基于原始图片生成对应的线稿图，但是边缘更柔和。

在 Stable Diffusion WebUI 中使用 SoftEdge 模型的过程和上面使用 Canny 模型基本上一致，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型，如图：

![SoftEdge 模型生成线稿图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-008.png)

- 1）在 `Control Type` 面板中更改为 `SoftEdge` 类型；
- 2）`预处理器（Preprocessor）`下拉框会自动匹配 `SoftEdge` 相关的预处理器；
- 3）`ControlNet 模型（Model）`下拉框会自动匹配 `SoftEdge` 相关的模型；
- 4）点击 `💥` 按钮生成的线稿图变成了边缘更柔和的线稿图。


SoftEdge 对应的有 4 个预处理器，如图：

![SoftEdge 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-004.png)


- `softedge_hed`
- `softedge_hedsafe`
- `softedge_pidinet`
- `softedge_pidisafe`

对应的模型是：

- `control_v11p_sd15_softedge_fp16`


按结果质量排序：`softedge_hed > softedge_pidinet > softedge_hedsafe > softedge_pidisafe`，其中名字结尾是 safe 的预处理器是为了防止生成的图像带有不良内容。SoftEdge 相比 Canny 生成的线稿边缘能够保留更多细节。下面是我们使用 `softedge_hed` 预处理器配合 `control_v11p_sd15_softedge_fp16` 模型生成的线稿图：


![SoftEdge 线稿图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-007.png)



剩下的基于 SoftEdge 模型生成新图的流程和使用 Canny 模型是一样的，我们这里就不再赘述了。结果图如下：

![SoftEdge 新图结果](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-009.png)






### 1.3、Lineart 模型：精细线稿控图


`Lineart 模型`是 ControlNet V1.1 版本新增的模型，Lineart 预处理器也能够提取图像的线稿，并且相比 Canny 线稿要更加精细。


在 Stable Diffusion WebUI 中使用 Lineart 模型的过程和上面使用 Canny 模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。

Lineart 对应的有 6 个预处理器、2 个模型，如图：

![Lineart 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-019.png)


- `lineart_standard`
- `lineart_realistic`
- `lineart_coarse`
- `invert`
- `lineart_anime`
- `lineart_anime_denoise`


![Lineart 模型](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-020.png)


- `control_v11p_sd15s2_lineart_fp16`
- `control_v11p_sd15s2_lineart_anime_fp16`


其中名字含 `anime` 的预处理器应该和 `control_v11p_sd15s2_lineart_anime_fp16` 模型搭配使用，其他预处理器则和 `control_v11p_sd15s2_lineart_fp16` 模型搭配使用。

下面是我们使用 `invert` 和 `lineart_anime` 预处理生成的线稿图：



![invert 线稿](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-015.png)


![lineart_anime 线稿](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-013.png)


下面是我们使用 `invert` 和 `lineart_anime` 线稿图控制生成的结果图：


![invert 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-017.png)


![lineart_anime 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-016.png)


其他 Lineart 预处理器大家可以自己试试效果。








### 1.4、Scribble 模型：涂鸦控图


`Scribble 模型`可以用来根据手绘涂鸦草图来生成图像，支持在空白画布上直接手绘涂鸦。

在 Stable Diffusion WebUI 中使用 Scribble 模型的过程和上面使用 Canny 模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


Scribble 对应的有 4 个预处理器，如图：

![Scribble 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-040.png)

- `scribble_hed`
- `scribble_pidinet`
- `scribble_xdog`
- `invert`

对应的模型是：

- `control_v11p_sd15_scribble_fp16`


下面是我们使用 `scribble_hed` 预处理生成的预处理图：

![Scribble 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-041.png)

下面是基于上面预处理涂鸦稿控制生成的结果图：

![Scribble 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-042.png)









### 1.5、Depth 模型：深度信息控图



`Depth 模型`可以提取原始图片中的深度信息，提取原始图片深度结构对应的深度图，这个深度图里，越亮的部分越靠前，越暗的部分越靠后。然后，基于深度图和提示词就可以生成具有同样深度结构的新图了。

在 Stable Diffusion WebUI 中使用 Depth 模型的过程和上面使用 Canny 模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


Depth 对应的有 4 个预处理器，如图：

![Depth 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-060.png)

- `depth_leres`
- `depth_leres++`
- `depth_midas`
- `depth_zoe`

对应的模型是：

- `control_v11f1p_sd15_depth_fp16`


下面是我们使用 `depth_leres` 预处理生成的预处理图：

![Depth 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-061.png)

下面是基于上面预处理深度信息图控制生成的结果图：

![Depth 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-062.png)







### 1.6、Normal 模型：法线信息控图



`Normal 模型`可以提取原始图片的凹凸信息，生成一张原图的法线贴图，这样便于 AI 给图片内容进行更好的光影处理，它比深度模型对于细节的保留更加的精确。法线贴图在游戏制作领域用的较多，常用于贴在低模上模拟高模的复杂光影效果。

在 Stable Diffusion WebUI 中使用 Normal 模型的过程和上面使用 Canny 模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


Normal 对应的有 4 个预处理器，如图：

![Normal 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-070.png)

- `normal_bae`
- `normal_midas`


对应的模型是：

- `control_v11p_sd15_normalbae_fp16`


下面是我们使用 `normal_bae` 预处理生成的预处理图：

![Normal 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-071.png)

下面是基于上面预处理深度信息图控制生成的结果图：

![Normal 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-072.png)







### 1.7、MLSD 模型：建筑线条控图

`MLSD 模型`通常用来检测建筑物的线条结构和几何形状，生成建筑物线框，再配合提示词、建筑及室内设计风格模型来生成图像。

下面我们将以下面这张毛胚房间图片为引导图来用 MLSD 预处理器对其进行预处理生成建筑房间线框图，然后在其基础上生成房间设计图。

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-030.png)


在 Stable Diffusion WebUI 中使用 MLSD 模型的过程如图：

![文生图 + MLSD 生成新图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-033.png)

- 1）在 ControlNet 面板的`引导图输入区`导入引导图，这里我们用的是一张毛胚房间图片；
- 2）在 ControlNet 面板的`参数设置区`选择 `MLSD` 控制类型、选择对应的`预处理器（Preprocessor）`和 `ControlNet 模型（Model）`、设置好其他参数，这里预处理器是用的 `mlsd`，模型用的是 `control_v11p_sd15_mlsd_fp16`；
- 3）点击 `💥` 按钮启动 ControlNet 预处理；
- 4）在`预处理预览（Preprocessor Preview）`区等待预处理的建筑线条图生成完成；
- 5）在`提示词输入区`输入房间描述的提示词；
- 6）点击 `Generate` 启动生成任务；
- 7）等待生成结果。

可以看到整个流程和上面使用 Canny 及其他模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型，另外提示词则是改成了房间描述。

下面是上面过程中预处理生成的建筑线条图和最终生成任务生成的结果图：

![MLSD 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-031.png)

![MLSD 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-032.png)

我们把毛胚房变成了精装房，这个在室内设计领域还是很有用的。




### 1.8、Seg 模型：分割区块控图


`Seg 模型`通过语义分割将画面标注为不同的区块颜色和结构，从而控制画面的构图和内容，其中不同颜色代表不同类型的对象。


在 Stable Diffusion WebUI 中使用 Seg 模型的过程和上面使用其他模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


Seg 对应的有 3 个预处理器，如图：

![Seg 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-051.png)

- `seg_ofade20k`
- `seg_ofcoco`
- `seg_ufade20k`


对应的模型是：

- `control_v11p_sd15_seg_fp16`


下面是原图，以及我们使用 `seg_ufade20k` 预处理生成的预处理图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-050.png)

![Seg 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-052.png)

下面是基于上面预处理深度信息图控制生成的结果图：

![Seg 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-054.png)








## 2、姿势绑定


## 2.1、OpenPose 模型：骨骼绑定


`OpenPose 模型`可以检测原始图片的骨骼形态信息，从而生成一张原图的骨骼姿势图。接着，再基于骨骼姿势图和提示词来生成具有同样骨骼姿势的新图，这样就实现了姿势控制。



在 Stable Diffusion WebUI 中使用 OpenPose 模型的过程和上面使用其他模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


OpenPose 对应的有 5 个预处理器，支持整体身体形态、面部、手指等信息的提取，如图：

![OpenPose 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-081.png)

- `openpose`
- `openpose_face`
- `openpose_faceonly`
- `openpose_full`
- `openpose_hand`

对应的模型是：

- `control_v11p_sd15_openpose_fp16`


下面是原图，以及我们使用 `openpose_full` 预处理生成的预处理图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-080.png)

![OpenPose 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-082.png)

下面是基于上面预处理骨骼姿势图控制生成的结果图：

![OpenPose 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-084.png)














## 3、特征控图


### 3.1、Shuffle 模型：风格迁移


`Shuffle 模型`可以提取出引导图的风格，再基于提示词将风格迁移到生成的新图上。



在 Stable Diffusion WebUI 中使用 Shuffle 模型的过程和上面使用其他模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


Shuffle 对应的预处理器，如图：

![Shuffle 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-101.png)

- `shuffle`

对应的模型是：

- `control_v11e_sd15_shuffle_fp16`


下面是原图，以及我们使用 `shuffle` 预处理生成的预处理图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-100.png)

![Shuffle 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-102.png)

下面是新生成的结果图：

![Shuffle 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-104.png)







### 3.2、T2IA Color 模型：颜色继承



`T2IA Color 模型`可以用网格的方式提取引导图的颜色分布图，然后在颜色分布的基础上结合提示词去将生成新图，从而控制新图保持对应的颜色分布。


在 Stable Diffusion WebUI 中使用 T2IA Color 模型的过程和上面使用其他模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


T2IA Color 对应的预处理器是 `t2ia_color_grid`、模型是 `t2iadapter_color_sd14v1`，如图：


![T2IA Color 预处理器和模型](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-091.png)



下面是原图，以及我们使用 `t2ia_color_grid` 预处理生成的预处理图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-090.png)

![T2IA Color 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-092.png)

下面是基于上面预处理的颜色分布图控制生成的结果图：

![T2IA Color 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-094.png)











### 3.3、Reference：相似重现


`Reference` 预处理器不使用模型，它可以在新图中尽量还原原图中的角色，作用和 Seed 有点像。


在 Stable Diffusion WebUI 中使用 Reference 预处理器时不需要选择模型，如图：

![Reference 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-111.png)


下面是原图，以及我们使用 `reference_only` 预处理器结合提示词生成的新图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-110.png)


![Reference 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-114.png)


原图和新图在人脸上会有一些相似。












## 4、细节增强


### 4.1、Tile 模型：细节增强

`Tile 模型`可以在原图的结构基础上对图像的细节进行增强。


在 Stable Diffusion WebUI 中使用 Tile 模型的过程和上面大部分模型也是基本一致的，主要的区别就是更换了 `Control Type` 以及对应的预处理算法和模型。


Tile 对应的有 3 个预处理器，如图：

![Tile 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-121.png)

- `tile_resample`
- `tile_colorfix`
- `tile_colorfix+sharp`


对应的模型是：

- `control_v11f1e_sd15_tile_fp16`


下面是原图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-120.png)


Tile 的是在原图的基础上增加更多细节，下面是生成的结果图：

![Tile 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-124.png)












### 4.2、Inpaint 模型：图像重绘


`Inpaint 模型`可以在原图的基础上添加蒙版，并对蒙版部分进行重绘。与我们在前面《图像局部重绘》一节介绍的功能类似。


在 Stable Diffusion WebUI 中使用 Inpaint 模型的过程如图：

![文生图 + Inpaint 生成新图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-133.png)

- 1）在 ControlNet 面板的`引导图输入区`导入引导图；
- 2）在引导图上涂绘出蒙版区域，我们将对蒙版区域进行重绘；
- 3）在 `Control Type` 单选区选择 `Inpaint` 类型，并配置对应的预处理器（Preprocessor）和 ControlNet 模型（Model）；
- 4）在 ControlNet 面板中设置其他相关参数；
- 5）点击 `💥` 按钮，启动预处理任务；
- 6）在`预处理预览（Preprocessor Preview）`区等待生成预处理图；
- 7）在`提示词输入区`输入提示词，我们这里输入 `wearing a flower on the head` 预期在女孩头上戴上一朵花；
- 8）点击 `Generate` 按钮启动生成任务；
- 9）等待生成结果。


这里的 Inpaint 类型的预处理器有 3 种，如图：

![Inpaint 预处理器](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-131.png)

- `inpaint_only`
- `inpaint_only+lama`
- `inpaint_global_harmonious`

对应的模型是：

- `control_v11p_sd15_inpaint_fp16`


下面分别是上述过程中用到的原图、生成的预处理图和最终生成的结果图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-130.png)

![Inpaint 预处理图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-132.png)

![Inpaint 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-134.png)








### 4.3、IP2P 模型：图片指令


`IP2P 模型`可以在原图的基础上通过提示词指令对其增加更多细节元素。


在 Stable Diffusion WebUI 中使用 IP2P 模型的过程和上面使用其他模型有一些不同：

- IP2P 模型不需要预处理器；
- 在`图生图（img2img）`中使用 IP2P 模型效果更好；
- 使用 IP2P 模型时，需要使用格式如 `make it ...` 的指令式提示词来对图片增加细节元素。

下图是在`图生图（img2img）`中使用 IP2P 模型的流程：


![图生图 + IP2P 生成新图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-143.png)


- 1）选择`图生图（img2img）`栏目；
- 2）在 `img2img` 子栏目输入引导图；
- 3）设置其他图生图相关参数；
- 4）在 ControlNet 面板的`引导图输入区`导入引导图，这里用的引导图与第 2 步一样；
- 5）在 `Control Type` 单选区选择 `IP2P` 类型，这里不需要配置的预处理器（Preprocessor），只用配置 ControlNet 模型（Model）即可；
- 6）在 ControlNet 面板中设置其他相关参数；
- 7）在`提示词输入区`输入提示词，我们这里输入 `make it snow` 指令提示词预期在原图中加入下雪效果；
- 8）点击 `Generate` 按钮启动生成任务；
- 9）等待生成结果。



下面分别是上述过程中用到的原图和最终生成的结果图：

![原图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-140.png)


![IP2P 结果图](assets/resource/aigc-tutorial/sd-use-controlnet/sd-use-controlnet-144.png)
















