---
title: AI 教程：配置 Stable Diffusion 扩展
description: 这里我们就来讲一讲如何在 Stable Diffusion WebUI 中配置各种扩展。
author: Keyframe
date: 2025-11-12 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-001.png
  alt: Stable Diffusion 配置扩展
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



Stable Diffusion WebUI 除了支持配置各种类型的模型外，还支持配置各种扩展插件，从而为用户使用 AI 绘画时提供更多丰富的功能。这里我们就来讲一讲如何在 Stable Diffusion WebUI 中配置各种扩展。

我们在前面介绍过，在 Stable Diffusion WebUI 中配置扩展的目录是在 `stable-diffusion-webui/extensions`。

在 WebUI 中有两种方式来实现配置扩展：

- 在 WebUI 页面中配置扩展。这种方式在界面操作即可，但不利于我们理解 WebUI 的目录作用，有时候遇到问题可能不好解决。
- 在 WebUI 目录中配置扩展。这种方式需要在命令行操作，但是更利于我们理解 WebUI 的目录作用。我们更推荐使用这种方式。



下面我们分别介绍一下这两种方式：



## 1、在 WebUI 页面中配置扩展


1）启动 Stable Diffusion WebUI 进入页面后，打开 `Extensions` 栏，再打开 `Extensions` 栏下的 `Installed` 栏。

可以看到 `Extensions` 栏下已经列出了一些内置的扩展插件了。如下图：

![Extensions 栏](assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-001.png)


2）打开 `Extensions` 栏下的 `Install from URL` 栏，在 `URL for extension's git repository` 下的输入框中输入扩展插件的 Git 地址。


我们这里以 ControlNet 这款扩展为例，所以我们在输入框中输入：`https://github.com/Mikubill/sd-webui-controlnet.git`。如下图：


![输入 ControlNet 扩展 Git 地址](assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-002.png)


3）点击 `Install` 按钮。等待数秒，你应该会收到 `Installed into stable-diffusion-webui/extensions/sd-webui-controlnet. Use Installed tab to restart` 的消息。

4）打开 `Extensions` 栏下的 `Installed` 栏，点击 `Check for updates` 按钮，然后点击 `Apply and restart UI` 按钮，重启 WebUI。


5）重启 Stable Diffusion WebUI 后，再次打开 `Extensions` 栏下的 `Installed` 栏。

这时你应该可以看到列表中多了新配置的 ControlNet 扩展。如下图：

![ControlNet 扩展配置完成](assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-003.png)

针对 ControlNet 这款插件，我们还可以看到 `txt2img` 栏下多了 `ControlNet` 面板，我们可以在这里使用它的相关功能。如下图：

![ControlNet 扩展功能面板](assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-004.png)



## 2、在 WebUI 目录中配置扩展



1）通过命令行进到 `stable-diffusion-webui/extensions/` 目录下。


2）在 `stable-diffusion-webui/extensions/` 目录下将扩展通过 `git clone` 命令下载下来即可。

我们这里以 ControlNet 这款扩展为例，我们在命令行输入：

```sh
git clone https://github.com/Mikubill/sd-webui-controlnet.git
```

等待下载完成，可以看到目录下多了一个 `sd-webui-controlnet` 文件夹。这样就完成了。



3）重启 Stable Diffusion WebUI 后，再次打开 `Extensions` 栏下的 `Installed` 栏。

这时你应该可以看到列表中多了新配置的 ControlNet 扩展。如下图：

![ControlNet 扩展配置完成](assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-003.png)

针对 ControlNet 这款插件，我们还可以看到 `txt2img` 栏下多了 `ControlNet` 面板，我们可以在这里使用它的相关功能。如下图：

![ControlNet 扩展功能面板](assets/resource/aigc-tutorial/install-sd-extensions/install-sd-extensions-004.png)







## 3、常用的扩展

除了上面示例的 ControlNet 扩展，还有很多其他的扩展可以配置使用，这里我们列出常用的扩展以供大家选择：




| 扩展 | 说明 | Git 地址 |
| :---- | :---- | :---- |
| ControlNet | 精细控制出图 | https://github.com/Mikubill/sd-webui-controlnet |
| Civitai Helper/Model Info Helper | 管理和展示模型信息 | https://github.com/butaixianran/Stable-Diffusion-Webui-Civitai-Helper |
| Tag Complete | 提示词自动补全工具 | https://github.com/DominikDoom/a1111-sd-webui-tagcomplete |
| Openpose Editor | 骨骼姿势编辑工具 | https://github.com/fkunn1326/openpose-editor |
| Posex | 骨骼姿势编辑工具 | https://github.com/hnmr293/posex |
| Localization zh_CN | 中文汉化 | https://github.com/dtlnor/stable-diffusion-webui-localization-zh_CN |
| Bilingual Localization | 中文汉化 | https://github.com/journey-ad/sd-webui-bilingual-localization |
| Cutoff | 控制色彩提示词影响范围 | https://github.com/hnmr293/sd-webui-cutoff |
| Depth Library | 手势姿势选择 | https://github.com/jexom/sd-webui-depth-lib |
| LoRA Block Weight | LoRA 分层权重调控 | https://github.com/hako-mikan/sd-webui-lora-block-weight |
| Latent Couple | 画面分区绘制 | https://github.com/opparco/stable-diffusion-webui-two-shot |
| Composable LoRA | 多个 LoRA 分割控制 | https://github.com/opparco/stable-diffusion-webui-composable-lora |
| Images Browser | 已生成图库浏览器 | https://github.com/AlUlkesh/stable-diffusion-webui-images-browser |
| LyCORIS | LyCORIS 扩展支持对 LyCORIS 模型的使用 | https://github.com/KohakuBlueleaf/a1111-sd-webui-lycoris |




我们可以用上面配置扩展的方法，选择自己想用的扩展来配置使用。

