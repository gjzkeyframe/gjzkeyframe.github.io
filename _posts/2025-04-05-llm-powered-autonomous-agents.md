---
title: LLM 驱动的自主代理系统
description: 关于自主智能体的思考。
author: Keyframe
date: 2025-04-05 08:08:08 +0800
categories: [AI Agent]
tags: [AI Agent, AI, AIGC]
pin: false
math: true
mermaid: true
---


> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
>_微信扫码关注我们_
>
>您还可以加入知识星球 `关键帧的音视频开发圈` 来一起交流工作中的**技术难题、职场经验**：
>
>![知识星球](assets/img/keyframe-zsxq.png)
>_微信扫码加入星球_
{: .prompt-tip }


这篇文章是从 lilianweng 在 2023 年写的一篇关于自主智能体的文章翻译而来，写的很好，读完后能让我们对整个自主智能体的知识框架有了一个初步的了解。而且从最近火爆的 AI Agent 项目中，我们似乎都能看到这篇文章的影子。这篇文章的结构如下：


- 代理系统概述
- 组件一：规划
	- 任务分解
	- 自我反思
- 组件二：记忆
	- 记忆类型
	- 最大内积搜索（MIPS）
- 工具使用
- 案例研究
	- 科学发现代理
	- 生成代理模拟
	- 概念验证示例
- 挑战


以下是正文内容：

---


基于大型语言模型（LLM）构建自主代理系统是一个非常酷的概念。像 AutoGPT、GPT-Engineer 和 BabyAGI 这样的概念验证示例为我们提供了启发。大型语言模型的潜力远远超出了生成精美的文案、故事、文章和程序；它可以被构建成一个强大的通用问题求解器。

## 代理系统概述

在基于 LLM 的自主代理系统中，LLM 作为代理的核心控制器，并辅以几个关键组件：

- **规划**
	- **子目标分解**：代理将大型任务分解为更小、更易管理的子目标，从而高效处理复杂任务。
	- **反思与改进**：代理可以对过去的行动进行自我批评和反思，从错误中学习并改进后续步骤，从而提高最终结果的质量。
- **记忆**
	- **短期记忆**：我将上下文学习视为利用模型的短期记忆。
	- **长期记忆**：这使代理能够长期存储和回忆（无限）信息，通常通过外部向量存储和快速检索实现。
- **工具使用**
	- 代理学习调用外部 API 以获取模型权重中缺失的信息（通常在预训练后难以更改），包括最新信息、代码执行能力、访问专有信息源等。

![基于 LLM 的自主代理系统概览](https://lilianweng.github.io/posts/2023-06-23-agent/agent-overview.png)


## 组件一：规划

复杂任务通常涉及多个步骤。代理需要知道这些步骤是什么，并提前进行规划。

### 任务分解

[思维链（Chain of Thought, CoT）](https://arxiv.org/abs/2201.11903 "CoT - Wei et al. 2022")已成为一种标准的提示词技术，用于提升模型在复杂任务上的表现。通过指示模型“一步步思考”，可以利用更多的测试时计算资源将复杂任务分解为更小、更简单的步骤。CoT 将大型任务转化为多个可管理的任务，并揭示了模型思维过程的解释。

[思维树（Tree of Thoughts, ToT）](https://arxiv.org/abs/2305.10601 "ToT - Yao et al. 2023")扩展了 CoT，通过在每一步探索多种推理可能性。它首先将问题分解为多个思维步骤，并在每一步生成多个思维，形成树状结构。搜索过程可以是广度优先搜索（BFS）或深度优先搜索（DFS），每个状态通过分类器（通过提示）或多数投票进行评估。

任务分解可以通过以下方式实现：

1. 使用简单的提示，例如："Steps for XYZ.\n1.", "What are the subgoals for achieving XYZ?"
2. 使用特定任务的指令，例如：用 "Write a story outline." 指令来写小说。
3. 结合人类输入。

另一种截然不同的方法是 [LLM+P](https://arxiv.org/abs/2304.11477 "LLM+P - Liu et al. 2023")，它依赖外部经典规划器进行长期规划。这种方法使用规划领域定义语言（PDDL）作为描述规划问题的中间接口。具体步骤包括：

1. LLM 将问题翻译为“问题 PDDL”。
2. 请求经典规划器根据现有的“领域 PDDL”生成 PDDL 计划。
3. 将 PDDL 计划翻译回自然语言。

本质上，规划步骤被外包给外部工具，假设存在领域特定的 PDDL 和合适的规划器（这在某些机器人设置中很常见，但在其他领域并不常见）。

### 自我反思

自我反思是自主代理迭代改进的关键，它允许代理通过改进过去的行动决策和纠正错误来不断提升性能。在现实世界的任务中，试错是不可避免的，因此自我反思尤为重要。

[ReAct](https://arxiv.org/abs/2210.03629 "Yao et al. 2023") 通过扩展 LLM 的行动空间（结合任务特定的离散行动和语言空间）将推理和行动整合到 LLM 中。前者使 LLM 能够与环境交互（例如使用维基百科搜索 API），而后者通过提示 LLM 生成自然语言的推理轨迹。

ReAct 的提示模板包含 LLM 的显式步骤，大致格式如下：

```
Thought: ...
Action: ...
Observation: ...
... (Repeated many times)
```

![ReAct 示例](https://lilianweng.github.io/posts/2023-06-23-agent/react.png)

上图是知识密集型任务（例如 HotpotQA、FEVER）和决策任务（例如 AlfWorld 环境、WebShop）的推理轨迹示例。图片来源：[Yao et al. 2023](https://arxiv.org/abs/2210.03629 "Yao et al. 2023")。

在知识密集型任务和决策任务的实验中，ReAct 的表现优于仅包含“行动”步骤的基线（即移除了“Thought: …”步骤）。

[Reflexion](https://arxiv.org/abs/2303.11366 "Shinn & Labash，2023") 是一个为代理配备动态记忆和自我反思能力的框架，以提升推理能力。Reflexion 采用标准的强化学习设置，其中奖励模型提供简单的二元奖励，行动空间遵循 ReAct 的设置，即任务特定的行动空间通过语言增强以支持复杂推理步骤。每次行动后，代理会计算一个启发式函数，并根据自我反思的结果决定是否重置环境以开始新的尝试。

![Reflexion 框架](https://lilianweng.github.io/posts/2023-06-23-agent/reflexion.png)


启发式函数用于确定轨迹是否低效或包含幻觉，并决定是否停止。低效规划指的是长时间未成功的轨迹，而幻觉被定义为连续相同的行动导致相同的环境观察结果。

自我反思通过向 LLM 展示两个示例对来实现，每个示例对包含（失败的轨迹，理想的反思以指导未来的计划更改）。然后将反思添加到代理的工作记忆中，最多三个，作为查询 LLM 的上下文。

![Reflexion 实验](https://lilianweng.github.io/posts/2023-06-23-agent/reflexion-exp.png)

上图是 AlfWorld 环境和 HotpotQA 的实验。在 AlfWorld 中，幻觉比低效规划更常见的失败类型。

[后见之明链（Chain of Hindsight, CoH）](https://arxiv.org/abs/2302.02676 "CoH - Liu et al. 2023") 通过明确向模型展示一系列过去的输出（每个输出附带反馈），鼓励模型通过自我反思改进输出。人类反馈数据是一个集合，其中包含提示、每个模型完成项、人类评分以及人类提供的后见之明反馈。假设反馈元组按奖励排序，CoH 的过程是监督微调，数据形式为序列，模型被微调为仅预测基于序列前缀的输出，从而能够根据反馈序列自我反思以产生更好的输出。

为了避免过拟合，CoH 添加了一个正则化项以最大化预训练数据集的对数似然。为了避免捷径和复制（因为反馈序列中有很多常见词），在训练期间随机屏蔽 0% - 5% 的过去标记。

实验中的训练数据集结合了 WebGPT 比较、人类反馈的总结和人类偏好数据集。

![CoH 示例](https://lilianweng.github.io/posts/2023-06-23-agent/CoH.png)

上图展示了经过 CoH 微调后，模型可以按照指令在序列中逐步改进输出。

CoH 的思想是通过上下文展示逐步改进的输出历史，并训练模型沿着这一趋势产生更好的输出。[算法蒸馏（Algorithm Distillation, AD）](https://arxiv.org/abs/2210.14215 "AD - Laskin et al. 2023") 将相同的思想应用于强化学习任务中的跨剧集轨迹，其中“算法”被封装在一个长历史条件策略中。考虑到代理与环境多次交互，每次剧集代理都会略有改进，AD 将这些学习历史串联起来并输入模型。因此，我们期望下一个预测的行动将比之前的尝试表现更好。

AD 的目标是学习强化学习的过程，而不是训练特定任务的策略本身。

![算法蒸馏](https://lilianweng.github.io/posts/2023-06-23-agent/algorithm-distillation.png)

上图是算法蒸馏（AD）的工作原理说明。

论文假设任何生成学习历史的算法都可以通过行为克隆将动作输入神经网络进行蒸馏。历史数据由一组源策略生成，每个策略针对特定任务进行训练。在训练阶段，每次强化学习运行时，都会随机采样一个任务，并使用多剧集历史的子序列进行训练，使得学习的策略与任务无关。

实际上，模型的上下文窗口长度有限，因此剧集需要足够短以构建多剧集历史。需要 2-4 个剧集的多剧集上下文才能学习到接近最优的上下文化强化学习算法。上下文化强化学习的出现需要足够长的上下文。

与三个基线（专家蒸馏 ED、源策略、RL²）相比，AD 在需要记忆和探索的环境中表现出接近 RL² 的上下文化强化学习性能（尽管仅使用离线强化学习），并且比其他基线学习速度更快。当基于源策略的部分训练历史进行条件训练时，AD 的改进速度也明显快于 ED 基线。

![AD 和 ED 的比较](https://lilianweng.github.io/posts/2023-06-23-agent/algorithm-distillation-results.png)

上图展示了在仅分配二元奖励的环境中，AD、ED、源策略和 RL² 的比较。源策略使用 A3C 训练“黑暗”环境，使用 DQN 训练水迷宫环境。

## 组件二：记忆

（非常感谢 ChatGPT 帮助我起草这一部分。通过与 ChatGPT 的对话，我学到了很多关于人脑和快速最大内积搜索（MIPS）的数据结构知识。）

### 记忆类型

记忆可以定义为获取、存储、保持和随后检索信息的过程。人类大脑中有几种记忆类型：

1. **感觉记忆**：这是记忆的最早阶段，能够在原始刺激结束后保留感官信息（视觉、听觉等）的印象。感觉记忆通常只持续几秒钟。子类别包括图像记忆（视觉）、回声记忆（听觉）和触觉记忆（触觉）。
2. **短期记忆（STM）或工作记忆**：它存储我们当前意识到的信息，用于执行复杂认知任务（如学习和推理）。短期记忆被认为可以存储大约 7 个项目（Miller，1956），持续时间为 20-30 秒。
3. **长期记忆（LTM）**：长期记忆可以存储信息的时间非常长，从几天到几十年不等，具有几乎无限的存储容量。长期记忆有两个子类型：
   - 显性/陈述性记忆：这是可以有意识回忆的事实和事件的记忆，包括情景记忆（事件和经历）和语义记忆（事实和概念）。
   - 隐性/程序性记忆：这种记忆是无意识的，涉及自动执行的技能和常规操作，如骑自行车或打字。

![人类记忆的分类](https://lilianweng.github.io/posts/2023-06-23-agent/memory.png)


我们可以大致进行以下映射：

- 感觉记忆作为原始输入（包括文本、图像或其他模态）的嵌入表示学习。
- 短期记忆作为上下文化学习。它短暂且有限，因为它受到 Transformer 模型有限上下文窗口长度的限制。
- 长期记忆作为代理可以在查询时访问的外部向量存储，通过快速检索实现。

### 最大内积搜索（MIPS）

外部记忆可以缓解有限注意力跨度的限制。一种标准做法是将信息的嵌入表示保存到向量存储数据库中，该数据库支持快速的最大内积搜索（MIPS）。为了优化检索速度，通常选择近似最近邻（ANN）算法以换取一点精度损失，从而实现巨大的速度提升。

一些常见的 ANN 算法包括：

- **LSH（局部敏感哈希）**：它引入了一个哈希函数，使得相似的输入项以高概率映射到相同的桶中，桶的数量远小于输入项的数量。
- **ANNOY（Approximate Nearest Neighbors Oh Yeah）**：其核心数据结构是随机投影树，一组二叉树，每个非叶节点表示将输入空间分成两半的超平面，每个叶节点存储一个数据点。树是独立且随机构建的，在某种程度上模仿了哈希函数。ANNOY 在所有树中进行搜索，迭代地搜索最接近查询的半部分，然后聚合结果。其思想与 KD 树非常相似，但更具可扩展性。
- **HNSW（Hierarchical Navigable Small World）**：它受到小世界网络思想的启发，其中大多数节点可以通过少量步骤到达其他节点（例如社交网络的“六度分隔”特性）。HNSW 构建这些小世界图的层次层，底层包含实际数据点。中间层创建快捷方式以加速搜索。执行搜索时，HNSW 从顶层的随机节点开始，向目标导航。当无法更接近时，它移动到下一层，直到到达底层。每次在上层的移动可能在数据空间中覆盖较大的距离，而每次在底层的移动则细化搜索质量。
- **FAISS（Facebook AI Similarity Search）**：它基于高维空间中节点距离遵循高斯分布的假设，因此数据点应该存在聚类。FAISS 通过矢量量化将矢量空间划分为聚类，然后在聚类内进一步细化量化。搜索首先查找粗量化中的聚类候选，然后进一步查看每个聚类中的细节。
- **ScaNN（Scalable Nearest Neighbors）**：ScaNN 的主要创新是各向异性矢量量化。它将数据点量化为，使得内积尽可能接近原始距离，而不是选择最近的量化中心点。


![MIPS 算法的比较](https://lilianweng.github.io/posts/2023-06-23-agent/mips.png)

上图展示了 MIPS 算法的比较，以 recall@10 为衡量标准。

更多 MIPS 算法及其性能比较，请参见 [ann-benchmarks.com](https://ann-benchmarks.com "ann-benchmarks.com")。

## 工具使用

工具使用是人类的一个显著且独特的特征。我们创造、修改和使用外部对象来完成超出我们身体和认知限制的事情。为 LLM 配备外部工具可以显著扩展模型的能力。

![](https://lilianweng.github.io/posts/2023-06-23-agent/sea-otter.png)

上图展示了海獭使用石头打开贝壳的图片，同时漂浮在水中。虽然其他一些动物也会使用工具，但其复杂性无法与人类相比。（图片来源：动物使用工具）

[MRKL](https://arxiv.org/abs/2205.00445 "MRKL - Karpas et al. 2022") 是“模块化推理、知识和语言”的缩写，是一种用于自主代理的神经符号架构。MRKL 系统包含一组“专家”模块，通用的 LLM 作为路由器，将查询路由到最合适专家模块。这些模块可以是神经网络（例如深度学习模型）或符号系统（例如数学计算器、货币转换器、天气 API）。

他们在使用计算器调用的 LLM 微调实验中发现，与明确陈述的数学问题相比，解决口头数学问题更困难，因为 LLM（7B Jurassic1-large 模型）无法可靠地提取基本算术的正确参数。结果强调了当外部符号工具可以可靠工作时，知道何时以及如何使用工具至关重要，这取决于 LLM 的能力。

[TALM](https://arxiv.org/abs/2205.12255 "Tool Augmented Language Models - Parisi et al. 2022") 和 [Toolformer](https://arxiv.org/abs/2302.04761 "Toolformer - Schick et al. 2023") 通过扩展数据集（基于是否添加新的 API 调用注释可以提高模型输出质量）来微调语言模型以学习使用外部工具 API。更多详细信息请参见提示工程的“外部 API”部分。

ChatGPT 插件和 OpenAI API 函数调用（function calling）是 LLM 增强工具使用能力的实际工作示例。工具 API 集可以由其他开发人员提供（如插件）或自定义（如函数调用）。

[HuggingGPT](https://arxiv.org/abs/2303.17580 "HuggingGPT - Shen et al. 2023") 是一个框架，它使用 ChatGPT 作为任务规划器，根据模型描述选择 HuggingFace 平台上的模型，并根据执行结果总结响应。

![HuggingGPT 的工作说明](https://lilianweng.github.io/posts/2023-06-23-agent/hugging-gpt.png)


该系统包含四个阶段：

**(1) 任务规划**：LLM 作为大脑，将用户请求解析为多个任务。每个任务有四个属性：任务类型、ID、依赖项和参数。他们使用几个示例来指导 LLM 进行任务解析和规划。

指令：

```
The AI assistant can parse user input to several tasks: [{"task": task, "id", task_id, "dep": dependency_task_ids, "args": {"text": text, "image": URL, "audio": URL, "video": URL}}]. The "dep" field denotes the id of the previous task which generates a new resource that the current task relies on. A special tag "-task_id" refers to the generated text image, audio and video in the dependency task with id as task_id. The task MUST be selected from the following options: {{ Available Task List }}. There is a logical relationship between tasks, please note their order. If the user input can't be parsed, you need to reply empty JSON. Here are several cases for your reference: {{ Demonstrations }}. The chat history is recorded as {{ Chat History }}. From this chat history, you can find the path of the user-mentioned resources for your task planning.
```

**(2) 模型选择**：LLM 将任务分配给专家模型，请求被构建成一个多选问题。由于上下文长度有限，需要基于任务类型进行过滤。

指令：

```
Given the user request and the call command, the AI assistant helps the user to select a suitable model from a list of models to process the user request. The AI assistant merely outputs the model id of the most appropriate model. The output must be in a strict JSON format: "id": "id", "reason": "your detail reason for the choice". We have a list of models for you to choose from {{ Candidate Models }}. Please select one model from the list.
```

**(3) 任务执行**：专家模型执行特定任务并记录结果。

指令：

```
With the input and the inference results, the AI assistant needs to describe the process and results. The previous stages can be formed as - User Input: {{ User Input }}, Task Planning: {{ Tasks }}, Model Selection: {{ Model Assignment }}, Task Execution: {{ Predictions }}. You must first answer the user's request in a straightforward manner. Then describe the task process and show your analysis and model inference results to the user in the first person. If inference results contain a file path, must tell the user the complete file path.
```

**(4) 响应生成**：LLM 接收执行结果并向用户提供更多总结结果。

为了将 HuggingGPT 投入实际使用，需要解决几个挑战：(1) 需要提高效率，因为 LLM 推理轮次和与其他模型的交互会减慢流程；(2) 需要长上下文窗口来传达复杂任务内容；(3) 需要提高 LLM 输出和外部模型服务的稳定性。

[API-Bank](https://arxiv.org/abs/2304.08244 "API-Bank - Li et al. 2023") 是一个评估工具增强型 LLM 性能的基准测试，包含 53 个常用的 API 工具、一个完整的工具增强型 LLM 工作流程以及 264 个涉及 568 次 API 调用的注释对话。API 的选择非常多样化，包括搜索引擎、计算器、日历查询、智能家居控制、日程管理、健康数据管理、账户认证流程等。由于 API 数量众多，LLM 首先可以访问 API 搜索结果引擎以找到正确的 API 调用，然后使用相应的文档进行调用。

![LLM 在 API-Bank 中进行 API 调用的伪代码](https://lilianweng.github.io/posts/2023-06-23-agent/api-bank-process.png)


在 API-Bank 工作流程中，LLM 需要在每一步做出几个决策，我们可以在每个步骤评估这些决策的准确性。决策包括：

1. 是否需要 API 调用。
2. 确定调用正确的 API：如果不够好，LLM 需要迭代修改 API 输入（例如决定搜索引擎 API 的搜索关键词）。
3. 根据 API 结果进行响应：如果结果不满意，模型可以选择细化并再次调用。

该基准测试在三个层次评估代理的工具使用能力：

- Level-1 评估 调用 API 的能力。给定 API 的描述，模型需要确定是否调用给定的 API，正确调用它，并适当地响应 API 返回结果。
- Level-2 检查 检索 API 的能力。模型需要搜索可能满足用户需求的 API，并通过阅读文档学习如何使用它们。
- Level-3 评估 超越检索和调用的 API 计划。对于不明确的用户请求（例如安排小组会议、为旅行预订航班/酒店/餐厅），模型可能需要进行多次 API 调用以解决问题。

## 案例研究

### 科学发现代理

[ChemCrow](https://arxiv.org/abs/2304.05376 "ChemCrow - Bran et al. 2023") 是一个特定领域的例子，其中 LLM 被增强为 13 个专家设计的工具，以完成有机合成、药物发现和材料设计任务。在 LangChain 中实现的工作流程反映了之前描述的 ReAct 和 MRKL，并结合了与任务相关的 CoT 推理：

- 向 LLM 提供工具名称、工具效用的描述以及预期的输入/输出详细信息。
- 然后指示 LLM 在必要时使用提供的工具回答用户给定的提示。指示建议模型遵循 ReAct 格式 - `Thought, Action, Action Input, Observation`。

一个有趣的观察是，尽管基于 LLM 的评估得出 GPT-4 和 ChemCrow 表现几乎相当的结论，但专家对解决方案的完整性和化学正确性的评估显示，ChemCrow 大幅超越了 GPT-4。这表明在需要深厚专业知识的领域，使用 LLM 评估其自身性能可能存在潜在问题。缺乏专业知识可能导致 LLM 无法发现其缺陷，从而无法很好地判断任务结果的正确性。

[Boiko 等](https://arxiv.org/abs/2304.05332 "Boiko et al. (2023)")还研究了用于科学发现的 LLM 驱动代理，以自主设计、规划和执行复杂的科学实验。该代理可以使用工具浏览互联网、阅读文档、执行代码、调用机器人实验 API 并利用其他 LLM。

例如，当被要求“开发一种新型抗癌药物”时，模型提出了以下推理步骤：

1. 询问抗癌药物发现的当前趋势；
2. 选择一个目标；
3. 请求针对这些化合物的支架；
4. 一旦确定了化合物，模型尝试合成它。

他们还讨论了风险，特别是非法药物和生物武器。他们开发了一个包含已知化学武器剂列表的测试集，并要求代理合成它们。在 11 个请求中，有 4 个（36%）被接受以获得合成解决方案，代理尝试咨询文档以执行程序。在 7 个被拒绝的案例中，其中 5 个是在网络搜索后发生的，另外 2 个仅基于提示被拒绝。

### 生成代理模拟

[生成代理](https://arxiv.org/abs/2304.03442 "Generative Agents - Park, et al. 2023")是一个非常有趣的实验，其中 25 个虚拟角色，每个角色由 LLM 驱动的代理控制，在沙盒环境中生活和互动，灵感来自《模拟人生》。生成代理结合了 LLM 与记忆、规划和反思机制，使代理能够根据过去的经历表现，并与其他代理互动。

- **记忆流**：是一个长期记忆模块（外部数据库），记录代理用自然语言表达的全面经历。
	- 每个元素是一个观察，即代理直接提供的事件。
	- 代理之间的通信可以触发新的自然语言陈述。
- **检索模型**：根据相关性、最近性和重要性提供上下文，以指导代理的行为。
	- 最近性：最近事件的得分更高。
	- 重要性：区分平凡记忆与核心记忆。直接询问 LM。
	- 相关性：基于与当前情况/查询的相关性。
- **反思机制**：随时间将记忆综合为更高层次的推断，并指导代理的未来行为。它们是过去事件的更高层次的总结（注意这与上述自我反思略有不同）。
	- 用最近的 100 个观察结果提示 LM，并生成给定一组观察/陈述的 3 个最突出的高层次问题。然后要求 LM 回答这些问题。
- **规划与反应**：将反思和环境信息转化为行动。
	- 规划本质上是为了优化当前与未来的可信度。
	- 提示模板：`{代理 X 的介绍}。以下是 X 今天的粗略计划：1)`
	- 考虑代理之间的关系以及一个代理对另一个代理的观察，以进行规划和反应。
	- 环境信息以树结构呈现。

![生成代理架构](https://lilianweng.github.io/posts/2023-06-23-agent/generative-agents.png)


这个有趣的模拟产生了涌现的社会行为，例如信息传播、关系记忆（例如两个代理继续对话主题）以及社会活动的协调（例如举办派对并邀请许多人）。

### 概念验证示例

AutoGPT 引起了人们对设置以 LLM 为主要控制器的自主代理可能性的关注。由于自然语言接口，它存在许多可靠性问题，但仍然是一个很酷的概念验证演示。AutoGPT 的许多代码涉及格式解析。

以下是 AutoGPT 使用的系统消息，其中 `{{...}}` 是用户输入：

```
You are {{ai-name}}, {{user-provided AI bot description}}.
Your decisions must always be made independently without seeking user assistance. Play to your strengths as an LLM and pursue simple strategies with no legal complications.

GOALS:

1. {{user-provided goal 1}}
2. {{user-provided goal 2}}
3. ...
4. ...
5. ...

Constraints:
1. ~4000 word limit for short term memory. Your short term memory is short, so immediately save important information to files.
2. If you are unsure how you previously did something or want to recall past events, thinking about similar events will help you remember.
3. No user assistance
4. Exclusively use the commands listed in double quotes e.g. "command name"
5. Use subprocesses for commands that will not terminate within a few minutes

Commands:
1. Google Search: "google", args: "input": "<search>"
2. Browse Website: "browse_website", args: "url": "<url>", "question": "<what_you_want_to_find_on_website>"
3. Start GPT Agent: "start_agent", args: "name": "<name>", "task": "<short_task_desc>", "prompt": "<prompt>"
4. Message GPT Agent: "message_agent", args: "key": "<key>", "message": "<message>"
5. List GPT Agents: "list_agents", args:
6. Delete GPT Agent: "delete_agent", args: "key": "<key>"
7. Clone Repository: "clone_repository", args: "repository_url": "<url>", "clone_path": "<directory>"
8. Write to file: "write_to_file", args: "file": "<file>", "text": "<text>"
9. Read file: "read_file", args: "file": "<file>"
10. Append to file: "append_to_file", args: "file": "<file>", "text": "<text>"
11. Delete file: "delete_file", args: "file": "<file>"
12. Search Files: "search_files", args: "directory": "<directory>"
13. Analyze Code: "analyze_code", args: "code": "<full_code_string>"
14. Get Improved Code: "improve_code", args: "suggestions": "<list_of_suggestions>", "code": "<full_code_string>"
15. Write Tests: "write_tests", args: "code": "<full_code_string>", "focus": "<list_of_focus_areas>"
16. Execute Python File: "execute_python_file", args: "file": "<file>"
17. Generate Image: "generate_image", args: "prompt": "<prompt>"
18. Send Tweet: "send_tweet", args: "text": "<text>"
19. Do Nothing: "do_nothing", args:
20. Task Complete (Shutdown): "task_complete", args: "reason": "<reason>"

Resources:
1. Internet access for searches and information gathering.
2. Long Term memory management.
3. GPT-3.5 powered Agents for delegation of simple tasks.
4. File output.

Performance Evaluation:
1. Continuously review and analyze your actions to ensure you are performing to the best of your abilities.
2. Constructively self-criticize your big-picture behavior constantly.
3. Reflect on past decisions and strategies to refine your approach.
4. Every command has a cost, so be smart and efficient. Aim to complete tasks in the least number of steps.

You should only respond in JSON format as described below
Response Format:
{
    "thoughts": {
        "text": "thought",
        "reasoning": "reasoning",
        "plan": "- short bulleted\n- list that conveys\n- long-term plan",
        "criticism": "constructive self-criticism",
        "speak": "thoughts summary to say to user"
    },
    "command": {
        "name": "command name",
        "args": {
            "arg name": "value"
        }
    }
}
Ensure the response can be parsed by Python json.loads
```

GPT-Engineer 是另一个项目，根据自然语言指定的任务创建整个代码库。GPT-Engineer 被指示思考需要构建的一系列较小组件的列表，并根据需要询问用户澄清问题。

以下是 GPT-Engineer 用于任务澄清的示例对话，发送到 OpenAI ChatCompletion 端点。用户输入用 `{{user input text}}` 包裹。

```
[
  {
    "role": "system",
    "content": "You will read instructions and not carry them out, only seek to clarify them.\nSpecifically you will first summarise a list of super short bullets of areas that need clarification.\nThen you will pick one clarifying question, and wait for an answer from the user.\n"
  },
  {
    "role": "user",
    "content": "We are writing {{a Super Mario game in python. MVC components split in separate files. Keyboard control.}}\n"
  },
  {
    "role": "assistant",
    "content": "Summary of areas that need clarification:\n1. Specifics of the Super Mario game (e.g. level design, characters, gameplay mechanics)\n2. Details about the MVC components (e.g. which components are in each file)\n3. Keyboard control implementation (e.g. which keys to use, how to handle input)\n\nClarifying question:\nCan you provide more details about the Super Mario game, such as level design, characters, and gameplay mechanics?"
  },
  {
    "role": "user",
    "content": "{{There are 10 levels in total. The main character is a plumber named Mario, who can walk and jump. It is a classical platform game just like Super Mario. The main character moves from left to right, trying to get to the destination, where there are many obstacles and attacks from enemies in the process.}}\n\nIs anything else unclear? If yes, only answer in the form:\n{remaining unclear areas} remaining questions.\n{Next question}\nIf everything is sufficiently clear, only answer \"Nothing more to clarify.\"."
  },
  {
    "role": "assistant",
    "content": "Remaining unclear areas: 2 remaining questions.\nCan you provide more information about how the MVC components are split into separate files?"
  },
  {
    "role": "user",
    "content": "{{Make your own assumptions and state them explicitly before starting}}"
  }
]
```

然后在这些澄清之后，代理进入代码编写模式，使用不同的系统消息。

系统消息：

```
You will get instructions for code to write. You will write a very long answer. Make sure that every detail of the architecture is, in the end, implemented as code. Make sure that every detail of the architecture is, in the end, implemented as code.
Think step by step and reason yourself to the right decisions to make sure we get it right. You will first lay out the names of the core classes, functions, methods that will be necessary, as well as a quick comment on their purpose.

Then you will output the content of each file including ALL code. Each file must strictly follow a markdown code block format, where the following tokens must be replaced such that FILENAME is the lowercase file name including the file extension, LANG is the markup code block language for the code’s language, and CODE is the code:

FILENAME

---
CODE
---

You will start with the “entrypoint” file, then go to the ones that are imported by that file, and so on. Please note that the code should be fully functional. No placeholders.

Follow a language and framework appropriate best practice file naming convention. Make sure that files contain all imports, types etc. Make sure that code in different files are compatible with each other. Ensure to implement all code, if you are unsure, write a plausible implementation. Include module dependency or package manager dependency definition file. Before you finish, double check that all parts of the architecture is present in the files.

Useful to know: You almost always put different classes in different files. For Python, you always create an appropriate requirements.txt file. For NodeJS, you always create an appropriate package.json file. You always add a comment briefly describing the purpose of the function definition. You try to add comments explaining very complex bits of logic. You always follow the best practices for the requested languages in terms of describing the code written as a defined package/project.

Python toolbelt preferences:

- pytest
- dataclasses
```



对话示例：

````
[
  {
    "role": "system",
    "content": "You will get instructions for code to write.\nYou will write a very long answer. Make sure that every detail of the architecture is, in the end, implemented as code.\nMake sure that every detail of the architecture is, in the end, implemented as code.\n\nThink step by step and reason yourself to the right decisions to make sure we get it right.\nYou will first lay out the names of the core classes, functions, methods that will be necessary, as well as a quick comment on their purpose.\n\nThen you will output the content of each file including ALL code.\nEach file must strictly follow a markdown code block format, where the following tokens must be replaced such that\nFILENAME is the lowercase file name including the file extension,\nLANG is the markup code block language for the code's language, and CODE is the code:\n\nFILENAME\n```LANG\nCODE\n```\n\nYou will start with the \"entrypoint\" file, then go to the ones that are imported by that file, and so on.\nPlease note that the code should be fully functional. No placeholders.\n\nFollow a language and framework appropriate best practice file naming convention.\nMake sure that files contain all imports, types etc. Make sure that code in different files are compatible with each other.\nEnsure to implement all code, if you are unsure, write a plausible implementation.\nInclude module dependency or package manager dependency definition file.\nBefore you finish, double check that all parts of the architecture is present in the files.\n\nUseful to know:\nYou almost always put different classes in different files.\nFor Python, you always create an appropriate requirements.txt file.\nFor NodeJS, you always create an appropriate package.json file.\nYou always add a comment briefly describing the purpose of the function definition.\nYou try to add comments explaining very complex bits of logic.\nYou always follow the best practices for the requested languages in terms of describing the code written as a defined\npackage/project.\n\n\nPython toolbelt preferences:\n- pytest\n- dataclasses\n"
  },
 #  … same conversation as earlier, ended with "Make your own assumptions and state them explicitly before starting".
  {
    "role": "assistant",
    "content": "Assumptions:\n1. Model: The model will contain the game's data, such as level information, character states, and enemy positions.\n2. View: The view will handle the game's visuals, including rendering the game objects, backgrounds, and updating the display.\n3. Controller: The controller will manage user input, such as keyboard controls, and update the model accordingly.\n\nWith these assumptions in mind, is there any specific aspect of the keyboard control implementation you would like me to clarify?"
  },
  {
    "role": "user",
    "content": "Please now remember the steps:\n\nThink step by step and reason yourself to the right decisions to make sure we get it right.\nFirst lay out the names of the core classes, functions, methods that will be necessary, As well as a quick comment on their purpose.\n\nThen you will output the content of each file including ALL code.\nEach file must strictly follow a markdown code block format, where the following tokens must be replaced such that\nFILENAME is the lowercase file name including the file extension,\nLANG is the markup code block language for the code's language, and CODE is the code:\n\nFILENAME\n```LANG\nCODE\n```\n\nPlease note that the code should be fully functional. No placeholders.\n\nYou will start with the \"entrypoint\" file, then go to the ones that are imported by that file, and so on.\nFollow a language and framework appropriate best practice file naming convention.\nMake sure that files contain all imports, types etc. The code should be fully functional. Make sure that code in different files are compatible with each other.\nBefore you finish, double check that all parts of the architecture is present in the files.\n"
  }
]
````

## 挑战

在了解了构建以 LLM 为中心的代理的关键思想和演示之后，我开始看到一些常见的限制：

- **有限的上下文长度**：受限的上下文容量限制了历史信息、详细指令、API 调用上下文和响应的包含。系统设计必须在这种有限的通信带宽下工作，而像自我反思这样的机制如果能够从过去的错误中学习，将大大受益于长或无限的上下文窗口。尽管向量存储和检索可以提供更大的知识库访问，但其表示能力不如全注意力强大。

- **长期规划和任务分解的挑战**：在漫长的历史中进行规划并有效探索解决方案空间仍然具有挑战性。当遇到意外错误时，LLM 调整计划的能力较差，使其与从试错中学习的人类相比不够鲁棒。

- **自然语言接口的可靠性**：当前的代理系统依赖自然语言作为 LLM 与外部组件（如记忆和工具）之间的接口。然而，模型输出的可靠性值得怀疑，因为 LLM 可能会犯格式错误，偶尔表现出叛逆行为（例如拒绝遵循指令）。因此，许多代理演示代码专注于解析模型输出。


## 原文

- [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)

