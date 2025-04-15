---
title: 深入了解 Cursor 的内部规则系统
description: 研究 Cursor 内部规则的一些经验。
author: Keyframe
date: 2025-04-06 08:08:08 +0800
categories: [AI 编程]
tags: [AI 编程, AI, AIGC, Cursor]
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


最近一位推友 [@xiaokedada](https://x.com/xiaokedada/status/1908530319856460282?s=46&t=f08eAX6xkfMpRQuEJ6qyQA "@xiaokedada") 分享了他对 Cursor 系统提示词词的一些研究，让我们来看看。


## 1、Cursor 的规则系统

Cursor 规则比想象中设计得要复杂，在 > 0.45 的版本中，Cursor 共有两类的规则：

**（1）User Rules**

通过 `Cursor Settings > General > Rules for AI` 进行配置。这里设置的是自定义的全局偏好，这些规则应用于所有项目。

**（2）Project Rules (MDC)**

项目规则是针对特定项目的，旨在满足各个项目的需求。Cursor 将其组织为结构化系统中的 `.mdc` 文件，当添加新规则时，系统会自动创建目录和文件。

```
└── .cursor
    └── rules
        ├── global.mdc
        └── only-html.mdc
```

MDC 的结构类似于下面：

```
---
description: this is an example rule
globs: *.tsx
alwaysApply: true/false
---

# Your rule content

- You can @ files here
- You can use markdown but dont have to
```

## 2、规则如何生效

Coding Agent 的本质是**提示词工程**，Cursor 设计的 Rules 的相关提示词如下：

```md
Please also follow these instructions in all of your responses if relevant to my query. No need to acknowledge these instructions directly in your response.
<custom_instructions>

// user rules
You are a Senior Front-End Developer and an Expert in ReactJS, ...

<available_instructions>
Cursor rules are user provided instructions for the AI to follow to help work with the codebase.
They may or may not be relevent to the task at hand. If they are, use the fetch_rules tool to fetch the full rule.
Some rules may be automatically attached to the conversation if the user attaches a file that matches the rule's glob, and wont need to be fetched.

// all your project rules placed here
project-rules-auto-attached: rules-auto-attached.description
project-rules-agent-requested: rules-agent-requested.description

</available_instructions>
</custom_instructions>

<cursor_rules_context>
Cursor Rules are extra documentation provided by the user to help the AI understand the codebase.
Use them if they seem useful to the users most recent query, but do not use them if they seem unrelated.

// always applied rules placed here
Rule Name: always-applied-rules.mdc
Description: always-applied-rules.description

This is an always applied rule...(content of the rule)

// auto attached rules matched by glob placed here
Rule Name: project-rules-auto-attached
Description: rules-auto-attached.description

This is agent requested rules matched index.tsx

</cursor_rules_context>

<additional_data>
Below are some potentially helpful/relevant pieces of information for figuring out to respond

<attached_files>
<file_contents>
```src/index.tsx (lines 1-18)
</file_contents>
</attached_files>
</additional_data>

<user_query>
// user query placed here
</user_query>
```

规则的生效要经过下面过程：

**（1）规则注入**

Rules 会被注入到系统提示词中，但分类两类：

- `alwaysApply` - 会直接将名字、描述和内容进行注入
- `glob`  - 基于文件模式匹配文件，如果匹配成功，则 （1）如果是 `auto-attached` 模式，则自动注入到提示词中；（2）如果是 `agent-requested` 模式，则不会。

**（2）规则激活**

规则的激活依赖其 `description` 字段，在 Rules 的提示词中，规则的激活规则是：

- 已经注入到提示词的规则

```
Use them if they seem useful to the users most recent query, but do not use them if they seem unrelated.
```

- 需要 Agent 主动获取的规则

```
Some rules may be automatically attached to the conversation if the user attaches a file that matches the rule's glob, and wont need to be fetched.
```

可见，规则的描述对于激活非常重要，它应该详细定义适当的场景，模型会评估上下文以决定是否使用 `fetch_rules` 来应用规则。另外，模型必须具备足够的智能来正确解释规则的描述，能力较弱的模型（例如 GPT-4-mini）可能无法有效理解和应用规则。

不过，规则过多可能不是个好注意。如果规则过多很有可能超出上下文限制，也会因为模型不够智能，很多无法频繁获取获取规则信息。



## 3、一个探究 cursor 背后 prompt 的方法

为了研究 cursor 背后的秘密，有一个实践是通过 Proxy 来代理 cursor 发往 OpenAI 的 prompt，一览无余。

实现步骤如下：

1. 租一台在新加坡 or 其他国家的服务器
2. 让 chatgpt 给你写一个 node.js 的 proxy 服务

也试过 cloudflare 的 AI Gateway，不过好像会暴露 ip 导致被墙。



