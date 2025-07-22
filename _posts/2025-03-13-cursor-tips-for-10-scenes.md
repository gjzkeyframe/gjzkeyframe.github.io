---
title: Cursor AI：10 个实用示例指南
description: 介绍使用 Cursor 进行 AI 编程的 10 条经验。
author: Keyframe
date: 2025-03-13 08:28:08 +0800
categories: [AI 编程]
tags: [AI 编程, AI, AIGC, Cursor]
pin: false
math: true
mermaid: true
---


>想要学习 AI 技术的朋友，快来加入我们的<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">【AI 赚钱社群】</a>，加入后你就能：
>
>- 1）获得全套 Stable Diffusion WebUI AI 图片和视频制作教程
>- 2）获得全套 ComfyUI AI 图片和视频制作教程
>- 3）获得自动化批量生产 AI 套图的程序
>- 4）获得 <a href="https://www.aigchub.ai" target="_blank" rel="noopener noreferrer">AIGCHub</a> 创作者资格，用 AI 生产图片和视频来赚钱
>
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/aigc-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }

---



我常常将代码复制粘贴到 ChatGPT 中，询问代码为何无法运行、请求重构代码，或者只是让我解释代码。然而，在代码编辑器和 ChatGPT 之间来回切换确实有些繁琐。

AI 编辑器通过将 GPT 直接集成到代码编辑器中，解决了这个问题。通过与我们的代码直接集成，GPT 能够获得更多关于整个项目的上下文信息，这显著提升了它们的输出质量。

在本文中，我们将探索最受欢迎的代码编辑器之一：Cursor AI。Cursor AI 获得了 OpenAI 和 Perplexity 工程师的信任，它提供 AI 辅助编码、智能代码建议以及与各种开发工具的集成。

## 1、Cursor AI 是什么以及它能做什么？

Cursor AI 是一款旨在让软件开发更简单的 AI 驱动代码编辑器。作为 Visual Studio Code（VS Code）的分支，它保留了 VS Code 用户友好的界面和丰富的生态系统，这让已经熟悉该平台的开发者更容易过渡。

Cursor AI 通过 OpenAI 的 ChatGPT 和 Claude 集成了先进的 AI 功能。这种集成使 Cursor AI 能够提供智能代码建议、自动错误检测和动态代码优化。

## 2、关键自动完成功能

Cursor 提供了关键自动完成功能和预测性代码功能：

1. 自动完成和代码预测：Cursor 提供自动完成功能，能够预测多行编辑并根据最近的更改进行调整。
2. 代码生成：熟悉最近的更改后，Cursor 会预测我们接下来想要做什么，并相应地建议代码。
3. 多行编辑：它能够建议跨越多行的编辑。
4. 智能重写：编辑器可以自动纠正和改进我们的代码，即使我们输入得不够仔细。
5. 光标预测：它会预测下一个光标位置，允许在代码中无缝导航。

## 3、聊天功能

Cursor 还集成了高级聊天功能，以促进更好的交互：

1. 代码库问答：查询 Cursor 关于代码库，它会在文件中搜索以提供相关答案。
2. 代码引用：引用特定的代码块或文件，并将它们集成到查询的上下文中。
3. 图像支持：将图像拖放到聊天中或使用按钮添加视觉上下文。
4. 网络搜索：直接从互联网获取最新信息并将其集成到代码查询中。
5. 立即应用：只需点击一下按钮，即可将聊天中的代码建议直接应用到代码库中。
6. 文档集成：引用流行的库并添加我们自己的文档以快速访问。

## 4、如何安装 Cursor AI

Cursor AI 提供了适用于 Linux、Windows 和 MacOS 的安装文件，可以从其网站免费下载。

![](assets/resource/aigc-programming/ctf1s-1.jpg)

在 Windows 和 MacOS 上安装时，我们从其主页下载并像安装其他程序一样安装下载的文件。

在 Linux 上，它以 AppImage 文件的形式提供。在执行它之前，我们需要使用以下命令使其可执行：

```
chmod a+x cursor-0.40.3x86_64.AppImage
```

然后，我们可以使用以下命令执行它：

```
./cursor-0.40.3x86_64.AppImage
```

根据下载的版本，可能需要替换上述命令中的版本号 0.40.3x86_64。有关如何安装 AppImage 文件的更多说明，请查看他们的网站。
安装后，我们会看到以下配置屏幕：

![](assets/resource/aigc-programming/ctf1s-2.jpg)

- 键盘：此选项允许我们配置键盘快捷键。默认情况下，它使用 VS Code 的快捷键，我建议使用这些快捷键，除非你熟悉列表中的其他代码编辑器。
- AI 语言：在这里，我们有使用非英语语言与 AI 交互的选项。
- 代码库范围：启用此选项允许 AI 理解整个代码库的上下文。
- 添加终端命令：如果已安装，这些命令允许从终端运行 Cursor AI 编辑器。

## 5、如何使用 Cursor AI：10 个使用场景

在本指南中，我们将提供 MacOS 快捷键，使用 Cmd ⌘ 键。如果你在 Windows 或 Linux 上使用 Cursor，快捷键相同，只是使用 Ctrl 键代替。

### 5.1、内联代码生成

我们使用 Cmd+K 快捷键打开内联代码生成器。这会打开一个小提示窗口，我们在其中插入生成代码的提示：

![](assets/resource/aigc-programming/ctf1s-3.jpg)

要生成代码，我们输入提示，然后点击生成按钮：

![](assets/resource/aigc-programming/ctf1s-4.jpg)

这将生成代码，我们通过点击接受按钮将其添加到项目中：

![](assets/resource/aigc-programming/ctf1s-5.jpg)

在这个例子中，我们使用了 cloude-3.5-sonnet 模型。我们可以通过模型下拉选择器选择其他模型。

![](assets/resource/aigc-programming/ctf1s-6.jpg)

### 5.2、与现有代码交互

我们还可以使用内联聊天与现有代码交互，方法是在使用 Cmd+K 快捷键之前选择相关代码。这可以用于修改代码，例如重构代码，或询问有关代码的问题。输入提示后，我们点击提交编辑按钮以获取修改：

![](assets/resource/aigc-programming/ctf1s-7.jpg)

在 Cursor 中，代码更改以 diff 的形式呈现。红色线条表示将被更改删除的行，而绿色线条表示将被添加的新更改。

![](assets/resource/aigc-programming/ctf1s-8.jpg)

### 5.3、询问有关现有代码的问题

同样，我们可以通过选择代码并使用 Cmd+K 快捷键来询问有关代码段的问题。对于问题，我们点击快速问题按钮以提交提示：

![](assets/resource/aigc-programming/ctf1s-9.jpg)

提交问题后，系统将生成答案并以以下方式显示：

![](assets/resource/aigc-programming/ctf1s-10.jpg)

### 5.4、使用 Tab 自动完成

在编写代码时，Cursor 会建议使用 AI 生成的代码完成。与传统的代码完成类似，我们可以使用 Tab 键将这些建议集成到我们的代码中。

例如，假设我们开始实现一个名为 maximum() 的函数。Cursor 会识别我们的意图并建议适当的实现。通过按 Tab 键，我们可以添加建议的代码：

![](assets/resource/aigc-programming/ctf1s-11.jpg)

自动完成还适用于用自然语言编写的代码。例如，如果我们想创建一个双层 for 循环来遍历列表中的所有对，我们只需用普通文本描述这一点。Cursor 然后会提供相应的自动完成建议，通过按 Tab 键可以将其集成。

![](assets/resource/aigc-programming/ctf1s-12.jpg)

### 5.5、聊天界面概览

要打开聊天窗口，使用 Cmd+L 快捷键。聊天窗口比内联生成器更通用，因为它允许我们不仅生成代码，还可以提问。以下是聊天界面的概览：

![](assets/resource/aigc-programming/ctf1s-13.jpg)

### 5.6、使用聊天生成代码

与使用内联聊天生成类似，我们还可以使用聊天功能生成代码。聊天中生成的代码可以通过点击代码窗口右上角的应用按钮集成到项目中。

![](assets/resource/aigc-programming/ctf1s-14.jpg)

### 5.7、使用 @ 增强查询上下文

或许聊天窗口最重要的功能是 @ 提及选项。此选项使我们能够为 AI 提供更多数据以生成响应。这从简单的文件和文件夹到网络搜索或给 AI 访问 GitHub 仓库的权限。

例如，我们可以使用 @Web 允许 AI 在网络上搜索答案。

![](assets/resource/aigc-programming/ctf1s-15.jpg)

请注意，在某些情况下，与 AI 共享整个代码库或私人 GitHub 仓库可能存在问题。我们应该注意与 AI 共享的内容，避免共享敏感或私人数据。

### 5.8、全局代码库问题

当处理更大的项目时，我发现自己最有用的功能之一是能够通过以整个代码库为范围提问来快速找到一段代码。最近，我想在一个项目中找到一个计算应用中导航方向的函数。借助 Cursor，我可以通过描述函数的功能轻松找到它：

![](assets/resource/aigc-programming/ctf1s-16.jpg)

请注意，在这种情况下我们使用了代码库选项。尽管由于某些原因 Cursor 没有显示实际代码，但点击代码框仍然打开了正确的文件并滚动到我寻找的函数。

![](assets/resource/aigc-programming/ctf1s-17.jpg)

### 5.9、图像支持

Cursor 聊天还支持图像输入。例如，我们可以为网站设计一个 UI 草图，并要求它生成相应的 HTML 和 CSS 代码。要添加图像，我们可以将其拖放到聊天窗口中。

![](assets/resource/aigc-programming/ctf1s-18.jpg)

### 5.10、添加文档

Cursor AI 的一个非常有用的功能是能够添加文档引用。这对于不太知名或私有的库尤其有用，其文档可能未在 AI 训练过程中使用过。


要添加文档条目，我们使用 @ 符号，然后从下拉菜单中选择 Docs：

![](assets/resource/aigc-programming/ctf1s-19.jpg)


这将打开一个窗口，要求提供文档的 URL。让我们以添加 PyTorch 文档为例：

![](assets/resource/aigc-programming/ctf1s-20.jpg)


插入 URL 后，我们可以为文档条目命名。在这种情况下，我们使用 PyTorch。然后我们可以在聊天提示中使用 @PyTorch 引用此文档。

![](assets/resource/aigc-programming/ctf1s-21.jpg)

文档引用还可以在 Cursor 设置的特性选项卡中管理：

![](assets/resource/aigc-programming/ctf1s-22.jpg)

## 6、Cursor AI：其他特性和优势

### 6.1、语言支持

本文中展示的示例使用了 Python、HTML 和 CSS。但 Cursor 并不是针对任何特定语言设计的。由于其代码生成基于通用 LLM，Cursor 可以生成任何编程语言的代码。它会根据文件扩展名来猜测应使用哪种语言。

### 6.2、扩展

由于 Cursor 是基于 VS Code 构建的，它继承了其丰富的扩展生态系统。我们可以通过查看菜单访问这些扩展。

![](assets/resource/aigc-programming/ctf1s-23.jpg)

要设置 Cursor 以与 Python 一起使用，我建议按照 VSCode Python 设置教程进行，因为 Cursor 应该具备与 VSCode 相同的功能。

### 6.3、与他人协作

使用 Git 等协作工具与 Cursor 一起使用，与使用任何代码编辑器类似。这些工具不依赖于代码的编写方式。有专门设计用于协助 Git 的扩展。


请记住，Cursor 的聊天允许你在上下文中使用 Git 仓库，使用 @ 操作符。如果仓库包含私人数据，使用时应谨慎。

![](assets/resource/aigc-programming/ctf1s-24.jpg)

### 6.4、设置自定义 AI 规则

Cursor 允许我们使用特定规则引导 AI。这些规则在常规设置菜单中可访问：

![](assets/resource/aigc-programming/ctf1s-25.jpg)

这些规则可以在不重复提示的情况下修改 AI 的行为。例如，我们可以通过添加“在 Python 函数定义中始终使用类型提示”这样的规则，确保 AI 始终在 Python 中使用类型提示。

### 6.5、自定义 AI 模型

Cursor 的另一个有趣特性是能够添加其他 AI 模型。此选项在模型设置下可找到：

![](assets/resource/aigc-programming/ctf1s-26.jpg)

在这里，我们可以添加新模型。这些模型设置还使我们能够添加 API 密钥，如果我们想要的话。

## 7、Cursor AI 与 GitHub Copilot

Cursor AI 和 GitHub Copilot 都是 AI 驱动的代码助手，各自提供独特的功能。

Cursor AI 基于 VSCode 构建，作为一个独立的编辑器。它与编码环境紧密集成，以自动化任务并提供直观的代码建议，这有助于简化代码编写和重构。它特别适合喜欢与熟悉的 IDE 深度集成的开发者。

GitHub Copilot 由 GitHub 和 OpenAI 开发，与 Visual Studio Code 等各种流行的代码编辑器集成。它根据用户的编码风格和项目上下文提供上下文感知的代码建议。GitHub Copilot 擅长预测后续代码行，并支持广泛的编程语言和框架。

从集成角度来看，Cursor AI 在其基于 VSCode 的独立环境中提供了显著的定制化，这可能为某些用户增强工作流程。相比之下，GitHub Copilot 以其在广泛使用的 IDE 中的轻松设置和集成而闻名，这促进了许多开发者的采用。

这两种工具都提供实时代码建议，并支持多种语言和框架。Cursor AI 在深度集成方面可能具有优势，适合特定任务，而 GitHub Copilot 的广泛 IDE 支持和简单的设置使其对更广泛的受众更具可及性。

最终，选择 Cursor AI 还是 GitHub Copilot 可能取决于定制化需求、集成偏好和预算等因素。这两种工具旨在以不同的方式提高编码效率。

## 8、结论

像 ChatGPT 这样的工具通过允许用户仅通过用自然语言解释目标来编写代码，使编程变得更加 accessible。Cursor 通过直接与代码编辑器集成，进一步消除了在编辑器和聊天界面之间切换的需要。

虽然 Cursor AI 提供了一个专门设计用于与 VSCode 环境深度集成的全面独立解决方案，但像 GitHub Copilot 这样的工具在各种流行的 IDE 中提供了灵活性。最终选择取决于个人需求和对定制化及设置 ease 的偏好。

总之，Cursor AI 是一个功能强大的 AI 驱动代码编辑器，有望改变开发者编写、重构和调试代码的方式。






---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

