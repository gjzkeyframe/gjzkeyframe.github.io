---
title: 探索 Metal 音视频技术（1）：命令的组织与执行模型
description: 本文将介绍 Metal 命令的组织与执行模型。
author: Keyframe
date: 2025-09-01 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 渲染, Metal]
pin: false
math: true
mermaid: true
---

>想要学习和提升音视频技术的朋友，快来加入我们的<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">【音视频技术社群】</a>，加入后你就能：
>
>- 1）下载 30+ 个开箱即用的「音视频及渲染 Demo 源代码」
>- 2）下载包含 500+ 知识条目的完整版「音视频知识图谱」
>- 3）下载包含 200+ 题目的完整版「音视频面试题集锦」
>- 4）技术和职业发展咨询 100% 得到回答
>- 5）获得简历优化建议和大厂内推
>  
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/keyframe-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }

---



这个系列文章我们来介绍一位海外工程师如何探索 Metal 音视频技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，本篇介绍命令的组织与执行模型。


在 Metal 架构中，`MTLDevice` 协议定义了代表单个 GPU 的接口。`MTLDevice` 支持以下功能：

- 查询设备属性；
- 创建设备专属对象（如 buffer、texture）；
- 编码并排队渲染/计算命令，以便提交给 GPU 执行。
    

**命令队列（command queue）**由一系列**命令缓冲区（command buffer）**组成，并决定这些缓冲区的执行顺序。

一个命令缓冲区包含针对某个特定设备的已编码命令；**命令编码器（command encoder）**把渲染、计算或 blit 命令追加到命令缓冲区，最终命令缓冲区被提交给设备执行。

## 1、协议与对象概览


协议：

- `MTLCommandQueue`：命令队列接口，主要提供创建命令缓冲区的方法。
- `MTLCommandBuffer`：命令缓冲区接口，提供创建命令编码器、排队/提交执行、查询状态等方法。
- `MTLRenderCommandEncoder`：一次渲染通道（render pass）的图形命令编码器。
- `MTLComputeCommandEncoder`：数据并行计算的命令编码器。
- `MTLBlitCommandEncoder`：负责 buffer ↔ texture 拷贝、mipmap 生成等简单操作的命令编码器。

> 任何时刻，每个命令缓冲区只能有一个活跃的命令编码器；必须先结束（`endEncoding`）当前编码器，才能为同一缓冲区创建新的编码器。  
> 唯一例外：`MTLParallelRenderCommandEncoder`，可在多线程环境下并行编码一次渲染通道。

## 2、对象关系图

如图：

![Metal Object Relationships](assets/resource/av-basic-knowledge/metal-1-1.png)
_Metal Object Relationships_


## 3、GPU 的抽象：MTLDevice


- 代表一块可执行命令的 GPU。
- 提供创建命令队列、分配 buffer、创建 texture、查询设备能力等方法。
- 通过 `MTLCreateSystemDefaultDevice()`获取系统默认设备。
    

## 4、短暂与持久对象


- 短暂对象（Transient）
	- 命令缓冲区（`MTLCommandBuffer`）
	- 各种命令编码器
- 持久对象（Non-transient）
	- 命令队列（`MTLCommandQueue`）
	- buffer、texture、采样器状态
	- library、计算/渲染管线状态、深度/模板状态

> 短暂对象生命周期极短，创建/销毁开销极低，使用 autorelease 返回。  
> 持久对象创建开销较大，应在性能敏感代码中复用，避免反复创建。



## 5、命令队列（Command Queue）


- 维护一条有序的命令缓冲区列表，GPU 按入队顺序执行。
- 线程安全，可同时编码多个命令缓冲区。
- 创建接口：
	- `newCommandQueue`
	- `newCommandQueueWithMaxCommandBufferCount:`
- 建议长驻，不应频繁创建/销毁。
    

## 6、命令缓冲区（Command Buffer）


- 存储已编码命令，直到被提交给 GPU。
- 通常一帧渲染对应一个命令缓冲区，可含多个渲染通道、计算或 blit 命令。
- 一次性对象，提交后只能等待完成或查询状态，不可重用。
- 创建接口：
	- `commandBuffer`（默认 retain）
	- `commandBufferWithUnretainedReferences`（性能极限场景使用，需自行保证对象生命周期）
    

### 6.1、执行顺序控制


- `enqueue`：在队列中占位，**不立即提交**；稍后 `commit`时按入队顺序执行。
- `commit`：提交命令缓冲区；若未 `enqueue`则隐式先 `enqueue`。

### 6.2、执行回调与同步

- `addScheduledHandler:`，命令缓冲区被调度（scheduled）时异步回调，可注册多个。
- `waitUntilScheduled`，同步等待调度完成。
- `addCompletedHandler:`，GPU 执行完成后异步回调，可注册多个。
- `waitUntilCompleted`，同步等待 GPU 执行完成。
- `presentDrawable:`，GPU 调度完成即呈现 `CAMetalDrawable`，属于特殊 completed handler。


### 6.3、状态与错误

- `status`只读属性，类型为 `MTLCommandBufferStatus`枚举。
- `error`只读属性，成功为 `nil`；失败时为 `MTLCommandBufferStatusError`，并给出错误码。
    

## 7、命令编码器（Command Encoder）


- 一次性、短暂对象，用于向命令缓冲区写入 GPU 可执行命令。
- 活跃时拥有所属命令缓冲区的独占写入权；完成后必须 `endEncoding`。
- 创建方式：从目标 `MTLCommandBuffer`请求对应类型的编码器。
    

### 7.1、创建接口



| 方法      | 返回 |
| ----------- | ----------- |
| `renderCommandEncoderWithDescriptor:`      | `MTLRenderCommandEncoder`       |
| `computeCommandEncoder`   | `MTLComputeCommandEncoder`        |
| `blitCommandEncoder`      | `MTLBlitCommandEncoder`       |
| `parallelRenderCommandEncoderWithDescriptor:`   | `MTLParallelRenderCommandEncoder`        |




### 7.2、渲染命令编码器（Render Command Encoder）

- 代表一次渲染通道（render pass）。
- 依赖 `MTLRenderPassDescriptor`，包含颜色、深度、模板附着。
- 功能：
	- 绑定资源（buffer、texture）
	- 设置渲染管线状态（`MTLRenderPipelineState`）
	- 设置固定功能状态（viewport、裁剪、深度/模板测试等）
	- 绘制 3D 图元
    

### 7.3、计算命令编码器（Compute Command Encoder）

- 用于数据并行计算。
- 可设置计算函数、绑定资源（texture、buffer、采样器）并分派执行。
- 通过 `MTLCommandBuffer.computeCommandEncoder`创建。
    

### 7.4、Blit 命令编码器（Blit Command Encoder）

- 处理 buffer ↔ texture 之间的拷贝、纹理填充纯色、mipmap 生成等。
- 通过 `MTLCommandBuffer.blitCommandEncoder`创建。
    

## 8、多线程、命令缓冲区与命令编码器


- 单线程模式：每帧单命令缓冲区，顺序编码、提交。
- 并行编码：
	- 同时创建多个命令缓冲区；每条线程独立编码一条缓冲区。
	- 预先知道执行顺序时，用 `enqueue`声明顺序，无需等待编码完成。
- 线程规则：同一时间仅允许一个 CPU 线程访问某个命令缓冲区。
- `MTLParallelRenderCommandEncoder`：允许一次渲染通道拆分到多线程并行编码。
    
![多线程模式](assets/resource/av-basic-knowledge/metal-1-2.webp)
_多线程模式_




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

