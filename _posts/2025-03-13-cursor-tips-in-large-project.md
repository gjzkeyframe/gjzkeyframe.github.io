---
title: 当 AI 码农遇上大型项目：Cursor 的花式搬砖指南
description: 大型项目中的 Cursor 使用指南。
author: Keyframe
date: 2025-03-13 08:08:08 +0800
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
>- 4）获得 <a href="https://aigchub.ai" target="_blank" rel="noopener noreferrer">AIGCHub</a> 创作者资格，用 AI 生产图片和视频来赚钱
>
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/aigc-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }

---





许多开发者可能认为 Cursor 和 Claude 只适用于原型开发。虽然 Cursor 在编写新代码方面表现出色，但它在结构化代码、标准化、重构和维护大型项目方面同样高效。这非常令人兴奋，因为它能让你的软件开发速度提升 5-30 倍。

本文将分享我在 Cursor 上的工作流程，以及如何将其用于大型项目。背景信息：我所在的技术团队为超过十亿终端用户提供聊天、活动信息流和视频服务。我们的代码库包含约 80 万行 Go 代码。


## 1、Cursor 编辑、测试循环

有效使用 AI 的关键在于良好的编辑和测试循环。你通常希望 AI 能够编写代码、编写测试，然后在修复发现的任何错误的同时执行测试。只有在 AI 完成这些步骤后，我才会开始审查。

让我们来了解一下这个编辑循环的基本步骤。

### 第一步：Cursor 设置/代理模式

你希望使用代理模式（cmd + I）+ Claude 3.7 sonnet。（注意左下角的小下拉菜单）。代理模式会持续调用 Claude，直到目标达成。因此，它会搜索文件、查找更多上下文、运行测试、安装包等。

![Cursor 设置/代理模式](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/d40148d7a174bb048d9439ecd99cc610/image7.png?auto=format&auto=compress)
_Cursor 设置/代理模式_

### 第二步：为 AI 准备文档

上面的例子有些简化。通常，你希望有一个文档文件夹，用于教授 AI 在你的代码库中执行常见任务的最佳实践。例如：

- 我如何编写测试？
- 我如何设置新的数据库模型并应用迁移？
- 我如何创建新的控制器/状态层等？

我们为 AI 保留一个单独的文档文件夹。它的结构大致如下。

![AI 单独文件夹](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/69ce0c6d6d06ed344d71b47c3fbb209b/image3.png?auto=format&auto=compress)
_AI 单独文件夹_

这与培训工程团队的方式颇为相似。但我们为 AI 保留单独的文档，以便在 AI 出错时能够轻松纠正其方向。

### 第三步：在设置中启用 YOLO 模式

你需要启用 YOLO 模式，这样 Cursor 就能在不询问确认的情况下运行测试。另外，你也可以仅允许运行测试等常用命令。

![YOLO 模式](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/cba8b6092266e8f4a171ff3215b6f6ee/image2.png?auto=format&auto=compress)
_YOLO 模式_

### 第四步：Cursor/Claude 运行测试（这是关键部分）

这是关键部分。你要告诉 Cursor 运行测试。因为它在运行测试时，会检测到在生成代码时所犯的错误。

![Cursor 运行测试](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/2be0b14b1258afa43926e64445ce4d27/image5.png?auto=format&auto=compress)
_Cursor 运行测试_

当然，AI 并非完美无缺；它会遗漏一些问题，但通过这种测试循环，结果会比仅生成代码好得多。

### 对于前端/其他平台

我主要将 Cursor 用于 Golang。但你也可以为前端开发设置类似的系统。查看 [@tedx_ai](https://x.com/tedx_ai) 的 BrowserTools，用于截图和控制台集成。你可以在 `https://cursor.directory/mcp` 找到更多 MCP 选项。目前，我尚未看到针对 Android、Swift、Flutter 和 React Native 开发的良好 MCP 选项。


## 2、Cursor 项目文件

编辑/测试循环是有效使用 Cursor 的关键。另一个重要的工作流程是创建项目文件。

### 项目步骤

以下是一个用于创建消息书签/提醒功能的项目文件示例。

![项目步骤](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/ea475ee66b467b94217be9dec226f765/image4.png?auto=format&auto=compress)
_项目步骤_

请注意，每个步骤都引用了相关文档。你也可以使用 Cursor 规则来实现，但我更倾向于手动指定正确的文档。

### 项目验证检查

现在你有了这个项目文件，还可以利用 AI 检查你的规范是否存在问题。我们的项目检查文件会审查模型，以确认指令是否明确了主键。对于控制器步骤，它会要求你澄清所需的权限。因此，你可以利用 AI 验证提供给 AI 的指令，这听起来有些疯狂，但确实可行。 🙂

### 生成项目文件

当然，你也可以使用 AI 生成项目描述文件。给它一个示例项目描述文件，并要求它为不同功能生成类似文件。目前，Grok 模型在这一任务上表现最佳。你还可以结合深度搜索进一步明确项目需求。


### Git 是你的检查点 - 重复此过程

Cursor 有一个内置的检查点系统，但我更倾向于不使用它。Git 对我来说更得心应手。要重置工作区，你可以使用以下命令：

```
git stash --include-untracked # 暂存所有更改，包括未被跟踪的文件
git stash pop # 恢复最后一次暂存
git clean -fd # 删除所有未提交的文件（使用此命令需谨慎）
```

因此，如果 Claude 走偏了，只需重置并重试即可。这也是保留项目文件的原因。它让你能够非常轻松地重新开始，使用不同的文档/最佳实践等。

## 3、其他 Cursor & Claude 使用技巧

在使用 Cursor 时，我们发现采取特定步骤并应用某些技巧可以显著提高 Cursor 生成输出的质量。

### 限制 Cursor 组合窗口中的步骤数量

有时，我会在单个组合窗口中运行 5-7 步。对话进行得越长，Claude 越有可能忘记部分指令。因此，有时需要创建一个新的 Cursor 代理窗口。

### Cursor 设置技巧

- 在 Cursor 设置中，你可以添加文档。这对于不太常用的包特别有用，因为 Claude 默认对这些包了解不多
- 与 Linear 或其他工具的 MCP 集成非常酷
- “/将打开的文件添加到上下文”非常方便

![Cursor 设置技巧](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/a8eb576487a7dcb4f1199b82c58bb227/image1.png?auto=format&auto=compress)
_Cursor 设置技巧_

### Goland

Cursor 的 AI 功能令人惊叹。我同时运行 Goland，用于调试、重构和一般的编辑器设计。对于 iOS/Android 开发等需要强大工具支持的领域，你可能也需要这样做。

### Cursor 工具

有一个很酷的 Cursor 工具项目 `https://github.com/eastlondoner/cursor-tools`，由 @EastlondonDev 开发。Cursor 工具启用了浏览器使用、大上下文窗口、文档和规划功能。

### Cursor 规则

你可以在 Cursor 设置中添加规则，从而自动包含文档。例如：

![Cursor 规则](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/c71790442657dcd41d599e7e6ed5d2d3/image6.png?auto=format&auto=compress&w=1600&fit=max)
_Cursor 规则_

有一个常见的 Cursor 规则目录：`https://cursor.directory/`。

### 代码标准化

这感觉几乎像人类，因为如果你使用令人困惑的名称、重复实现等，AI 会感到困惑。因此，你希望拥有干净、标准化的代码，以获得 AI 的最高成功率。

### 检查一切

如果你给初级工程师分配任务，你会双重检查一切。你也应该以类似的方式对待 AI，并了解每一段代码的作用。




## 4、重构、文档和搜索

这不仅仅是代码生成。你还可以使用 Cursor & Claude 进行文档编写、搜索和重构。

### 重构示例

你可以一次性对数百个文件进行复杂调整。如果是简单更改，我仍然更喜欢 Goland 的重构工具。但对于复杂更改，这可以节省数天的工作时间。

![重构示例](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/f9696d4b86d688c4fdb004bde69dc124/image2.png?auto=compress%2Cformat&fit=scale&h=289&ixlib=php-3.3.0&w=1024&wpsize=large)
_重构示例_

### 搜索与文档

每个大型代码库最终都会有一些难以理解的部分。你可以要求 Cursor 为你编写文档，以帮助解释这些部分。

![搜索与文档](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/7fdbf4a003cf9fa4b111c1ecf8fd9feb/image1.png?auto=compress%2Cformat&fit=scale&h=282&ixlib=php-3.3.0&w=1024&wpsize=large)
_搜索与文档_

### 技术与理解

当你遇到代码库中不理解底层技术的部分时，也可以将其作为 Google/Stackoverflow 的替代方案。

![技术与理解](https://stream-blog-v2.imgix.net/blog/wp-content/uploads/a1338cbec5bdfc74418572fd5a5eec08/image3.png?auto=compress%2Cformat&fit=scale&h=282&ixlib=php-3.3.0&w=1024&wpsize=large)
_技术与理解_


## 5、结论

Cursor 不仅适用于原型开发，对于维护大型项目也非常出色。要有效使用它：

- 设置生成/测试/运行测试周期，让 AI 自我修正
- 制定项目计划，并让 AI 检查和改进此计划
- 调整你的 Cursor 设置，适应不同的工作流程
- 它不仅仅是代码生成工具。你还可以用它进行重构、创建文档和作为强大的搜索引擎

有了正确的设置，你可以以 5-30 倍的速度工作。我特别喜欢的是，作为工程师，你可以将更多精力集中在更难的问题上，而让 AI 生成所有基础代码。希望本指南对你有所帮助。









---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群和更多同行朋友来交流和讨论：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

