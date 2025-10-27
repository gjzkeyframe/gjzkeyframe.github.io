---
title: 使用 Cursor 的 20 个实用建议
description: 介绍使用 Cursor 进行 AI 编程的 20 条经验。
author: Keyframe
date: 2025-03-13 10:28:08 +0800
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





在将 Cursor 整合到我的日常编码流程中半年后，我发现了许多能够最大化发挥其潜力的宝贵策略。这段经历充满了发现、挑战和令人兴奋的时刻。我迫不及待地想分享我的经验，希望能帮助你在使用这个 AI 驱动的编码助手时少走弯路。



## 1、上下文是准确性的基石

我学到的最重要的教训之一是，Cursor 的效果取决于你提供的上下文。它不是读心术，你给它的相关信息越多，它的表现就越好。

我发现用 ‘@’ 标记相关文件和使用网络链接可以显著提高代码建议的准确性。例如，在处理一个新的 API 端点时，我可能会说：

> “@routes/api.js @models/user.js 创建一个用户注册端点，验证输入，对密码进行哈希处理，并将用户存储在数据库中。”

这种方法让 Cursor 全面了解你的项目结构，从而提供更相关和准确的建议。这就像给 Cursor 一张代码库的地图——地图越详细，它导航得越好。

不要低估 **.cursorrules** 文件的力量。它是设定项目特定指南和确保代码库一致性的绝佳方式。把它想象成项目的风格指南，但针对 AI。我发现定义良好的 .cursorrules 可以显著减少代码不一致。

![.cursorrules 文件有很大的价值](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*78YH0X4Sn9lLYQVcur3lPg.png)
_.cursorrules 文件有很大的价值_


## 2、作曲家（Composer）：你的编码副驾

Cursor 的作曲家功能是一个强大的工具，但我花了一段时间才弄清楚如何有效使用它。以下是我的心得：

- 用它来处理样板代码：作曲家在为组件或函数设置基本结构方面表现出色。就像在你的指尖有一个模板库。
- 对其输出进行迭代：不要指望第一次就完美。把作曲家当作起点，从那里进行完善。这是你和 AI 之间的协作过程。
- 始终审查生成的代码：尽管作曲家很厉害，但它并非无懈可击。我曾花费数小时调试本可以通过快速审查发现的问题。信任，但要验证。

我发现用作曲家进行初始设置，然后微调输出，显著减少了我花在重复编码任务上的时间。它在创建项目中一致的结构时特别有用。

## 3、聊天（Chat）：随时待命的编码伙伴

Cursor 的聊天功能已成我解决问题的首选。就像有一个编码伙伴随时待命，永远不会对你的问题感到厌倦。

![Cursor 的聊天功能](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*hX_Q4Yzr-gNGvTFJVHapbw.png)
_Cursor 的聊天功能_

我用聊天功能处理从头脑风暴到棘手漏洞的所有问题。有时，向聊天解释问题就能让我更清楚地看到解决方案——这就像数字版的橡皮鸭调试法。

小贴士：用 cmd+enter 让 Cursor 获取整个代码库的上下文。在处理跨越多个文件的问题时，这非常有帮助。就像给 Cursor 一个项目的鸟瞰图。

我发现用聊天功能解决问题帮助我更快、更高效地解决 issues。这不仅仅是获取答案——它是为你的想法提供一个回音板和调试过程中的伙伴。


## 4、笔记本（Notepads）：数字记忆库

笔记本在管理常见任务和模式方面改变了游戏规则。我为诸如“添加新路由”、“设置测试套件”和“常见漏洞修复”等事项创建了笔记本。

当我需要参考这些内容时，只需在聊天中使用 u/notepad 命令。这就像在指尖有一个个人编码食谱。这个系统显著减少了我花在重复任务上的时间，并帮助保持项目的一致性。

不要低估组织良好的笔记本的力量。它们不仅仅是存储片段——还可以用来记录最佳实践、常见陷阱和项目特定的约定。就像在 Cursor 内构建自己的知识库。

## 5、无缝工作流整合

Cursor 不仅仅是一个独立的工具——当整合到更广泛的发展过程中时，它表现最佳。我发现它在与 Git 一起使用时特别有用。

需要帮助撰写提交消息？问 Cursor。审查拉取请求？Cursor 可以帮助总结更改。就像在你的代码上有额外的眼睛，抓住你可能错过的东西，并帮助你保持代码库的整洁和有序。

![Cursor 整合 GitHub Actions 工作流](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QRZ4ErwWHR1kpK4QFGrUHA.png)
_Cursor 整合 GitHub Actions 工作流_

我还发现 Cursor 在 CI/CD 过程中有帮助。它可以协助优化构建脚本、建议改进工作流文件，甚至帮助调试管道问题。它已成我整个开发周期中不可或缺的一部分，而不仅仅是编码阶段。

## 6、模型选择：为任务选择合适的工具

Cursor 支持多种 AI 模型，我发现不同的模型在不同的任务上表现出色。关键不是找到“最佳”模型，而是为每个特定任务选择合适的模型：

- Claude 3.5 Sonnet：这是我的日常编码任务首选。它在速度和准确性之间提供了良好的平衡。我用它进行日常编码、快速重构和一般问题解决。
- GPT-4 (o1)：我用这个处理更复杂的问题或当我需要对代码库有特别细腻的理解时。它在架构决策、复杂算法设计或理解代码不同部分之间的复杂关系时非常棒。
- GPT-4 Mini：这对于不需要大型模型全部功能的快速任务来说很棒。我经常用它进行数据结构调整或简单的代码格式化任务。

不要害怕根据手头的任务切换模型。就像有一个工具箱，里面有不同的专业工具——你不会用锤子处理所有事情，AI 模型也是如此。

## 7、语音输入：意想不到的生产力提升

这听起来可能不太常规，但我开始在 Cursor 中使用语音转录工具，尤其是对于较长的提示。这出奇地高效，能让我更自然地表达复杂想法。

我发现这在头脑风暴或尝试解释复杂问题时特别有用。大声说出问题往往能帮助我更清晰地理清思路，而语音转录工具能快速高效地捕捉这些想法。

当然，这种方法并不适合所有情况。代码片段和技术术语可能对语音识别来说比较棘手。但对于高层次描述、架构讨论或解释特定方法背后的逻辑时，这确实节省了不少时间。

## 8、键盘快捷键：加速工作流

学习 Cursor 的键盘快捷键显著加快了我的工作速度。这看似是小事，但一天下来，这些时间节省累加起来相当可观。我最常用的快捷键包括：

- cmd+k：打开作曲家窗口
- cmd+L：打开聊天窗口
- cmd+i：将聊天建议移动到作曲家

花时间记住这些快捷键带来了生产力的提升。就像学习盲打——一开始可能觉得笨拙，但一旦成为肌肉记忆，你会怀疑没有它怎么工作。

## 9、集成终端：被忽视的宝石

Cursor 的集成终端是我最初忽略但现在已经频繁使用的功能。它非常适合快速的 Git 操作、包管理和运行测试，无需离开编码环境。

终端直接在编辑器里减少了上下文切换，让你保持编码状态。我发现它对于运行与正在处理的代码相关的快速测试或脚本特别有用。就像在编辑器里内置了一个瑞士军刀。



## 10、作曲家的难以置信的代理模式功能

Cursor 作曲家是一个高级功能，能够使用 AI 进行多文件编辑和完整应用程序生成。它超越了单行和单文件编辑，可以同时创建和修改多个文件。

**关键特性：**

- a. 多个并发作曲家
	- 同时运行多个作曲家来修复代码库的不同部分
	- 避免与 YOLO 模式一起使用，以防止出现复杂问题

![多 Composer 代理模式](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*yHjx0oPSEf4oaUJUHgffdA.png)
_多 Composer 代理模式_

- b. 多个作曲家代理
	- 为不同的编辑任务分配不同的代理
- c. Lint 迭代支持
	- 作曲家自动尝试修复生成代码中的 Linting 问题
	- 目前对大多数编程语言支持一次迭代
- d. YOLO 模式（谨慎使用）
	- 运行控制台命令并处理输出日志以进行持续改进。警告：AI 可能会出错。在工作电脑上尤其要格外小心，因为终端可能有权限访问私人信息。
	- 此功能允许 AI 自动运行终端命令、安装包、管理开发服务器，并根据命令输出尝试修复。

## 11、现代技术栈文档集成

Cursor 允许添加文档以确保遵循当前最佳实践：

- 使用 @Docs 符号访问第三方文档
- 在 URL 后添加尾部斜杠以索引所有子页面
- 在 Cursor 设置 > 功能 > 文档下管理文档
- 通过 @Docs > 添加新文档来添加自定义文档

![Cursor 项目中的文档](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dJS52COI9jzLh7vj1srwhA.png)
_Cursor 项目中的文档_

## 12、战略使用 .cursorrules 文件

过多的上下文可能会稀释焦点，尤其是在较长的对话中。保持规则的针对性，并考虑在 Readme.md 中添加部署和操作参考。

**示例 .cursorrules：**

```
- Use deploy.sh for frontend/backend deployment (Terraform-based)
- Optimize for Lambda code/infrastructure differences
- Avoid direct AWS CLI deployment; use only for obtaining logs for debugging
- Remember CORS and IAM permissions in AWS
- Ensure Linux-compatible builds (especially from MacOS)
- Configure AWS Bedrock and SES permissions properly
- Region: us-west-2
- Frontend in root, backend in /backend
```

## 13、默认项目规则

在 Cursor 设置 > 通用下配置通用规则：

```
1. I am on Mac with ARM64 Architecture. Ensure building of docker images, etc build in Linux where necessary, and use Rosetta if appropriate. I am usually building and deploying for Linux environments.
2. Use yarn over npm and other package managers.
3. Never use placholders in response text for code.
4. If new information conflicts with old information, check the documentation if available, and always favour new methods.
5. Consult documentation before running any commands, particular with aws cli.
6. If you learn anything interesting about the project that could be useful for the future, advise to add to .cursorrules.
```

## 14、CMD+SHIFT+V / CTRL+SHIFT+V

在引入带有额外说明的文本时，使用此快捷键粘贴而不附加上下文，特别适用于在同一消息中引入带有额外说明的文本。

## 15、Stack Overflow 集成

粘贴 Stack Overflow 或 GitHub 链接以获取增强指导。Cursor 处理这些内容以改进其推理，因为 LLM 通常不会基于这些数据进行训练，而这些数据是技术支持最有价值的信息来源之一。

## 16、经过验证的解决方案优先

经过实战检验的软件和基础设施由于广泛的训练数据，与 AI 的协作效果更好。较新的方法可能缺乏足够的 LLM 训练数据或在线故障排除资源。

## 17、专注的作曲家对话和逐步分解

对于复杂的项目，我使用这种迭代方法与 Claude Sonnet 3.5：

**1、初始规划**

在一次对话中询问 AI：

>“实现/解决 [问题] 的最佳方法是什么，分步骤详细说明？”

**2、第一步执行**

在同一对话中请求：

>“给我一个完整的提示，详细说明第一步需要做什么”

- 将此提示粘贴到一个新的作曲家对话中
- 执行任务
- 要求对已完成的工作进行总结

**3、迭代进展**

- 返回到第一个对话并说明：

>我已经完成了第一步，这是已完成工作的总结：{总结}。现在，请建议一个超级提示，用于第二步，我可以将其粘贴到一个新对话中，以获取需要完成的任务的指导。

**4、重复此过程**

- 用第二步的提示开始新对话
- 获取工作摘要
- 返回到原始对话
- 要求下一个步骤的提示

.... 

为每个步骤重复此过程。这可以发挥每个任务不同上下文的优势，避免一次性向 LLM 提供过多信息而造成混乱。


## 18、生成提交消息

你可能不知道 Cursor 可以为你生成提交消息。我总是忘记这个功能，但我真的应该更多地使用它。只需转到源代码控制标签页，点击那个小魔杖图标。

虽然它并不总是完美的，但可以为你的提交消息提供一个很好的起点。

## 19、查找漏洞功能

让我们来谈谈查找漏洞功能。它非常酷。你可以通过按下 `Command+Shift+P` 并输入“查找漏洞”来访问它。

查找漏洞功能会将你的更改与主分支进行比较，试图找出你可能引入的潜在漏洞。它并不完美，但可以捕捉到一些你可能遗漏的问题。

例如，它可能会发现诸如没有正确处理零值的问题，这在处理 UI 工作中的位置时尤其容易出错。


## 20、持续学习和实验

Cursor 不断发展，定期添加新功能和改进。保持好奇心和开放实验的态度是充分利用这个工具的关键。

我定期检查更新并参与社区讨论。总有新东西可学，用户之间分享技巧也很棒。

不要害怕尝试新方法或功能。适合一个开发者的未必适合另一个，找到自己的风格和工作流很重要。把每个新功能或更新当作潜在提升工作流的机会。


## 总结

使用 Cursor 半年后，我可以自信地说，它改变了我的编码方式。它不是魔法解决方案——没有工具是——但它已成为我开发过程中不可或缺的一部分。

Cursor 是增强技能的工具，而不是替代技能。始终审查它生成的代码并相信自己的判断。目标是提升能力，而不是依赖工具。

如果你有自己的技巧和方法，请在评论中分享。我花时间写了这篇经验谈，如果你从中受益，我也很乐意听到你的反馈。












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

