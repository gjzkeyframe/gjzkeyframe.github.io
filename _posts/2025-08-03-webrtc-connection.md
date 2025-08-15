---
title: 探索 WebRTC 开发（3）：WebRTC 连接
description: 本文将介绍各种与 WebRTC 相关的协议如何相互作用，以创建连接并在对等方之间传输数据和/或媒体。
author: Keyframe
date: 2025-08-03 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 音频, 实时音视频, WebRTC]
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

本文档描述了各种与 WebRTC 相关的协议如何相互作用，以创建连接并在对等方之间传输数据和/或媒体。

**注意：**
此页面需要大量重写以确保结构完整性和内容完整性。这里的信息很多，但由于这是一个临时的存储库，所以组织结构很混乱。

## 信令

不幸的是，WebRTC 无法在没有某种中间服务器的情况下创建连接。我们称之为 **信令通道** 或 **信令服务** 。它可以是任何在建立连接之前交换信息的通信渠道，无论是通过电子邮件、明信片还是信鸽。由你决定。

我们需要交换的信息是包含下方提到的 SDP 的 Offer 和 Answer。

将发起连接的对等方 A 将创建一个 Offer。然后他们将通过选定的信令通道将此提议发送给对等方 B。对等方 B 将从信令通道接收提议并创建一个 Answer。然后他们将通过信令通道将此回答发送回对等方 A。

### 会话描述

WebRTC 连接中端点的配置称为 **会话描述** 。描述包括有关发送媒体的类型、格式、使用的传输协议、端点的 IP 地址和端口以及其他描述媒体传输端点所需的信息。此信息使用 **会话描述协议** (SDP) 进行交换和存储；如果你需要了解 SDP 数据格式的详细信息，可以在 RFC 8866 中找到。

当用户开始与另一用户进行 WebRTC 通话时，会创建一个特殊的描述，称为 **提议** 。此描述包括有关呼叫者提议的呼叫配置的所有信息。接收者随后会用一个 **回答** 来响应，该回答是他们端的呼叫描述。通过这种方式，双方设备共享了进行媒体数据交换所需的信息。此交换使用互动式连接建立 (ICE) 协议进行，该协议允许两个设备使用中间人来交换提议和回答，即使两个设备被网络地址转换 (NAT) 分隔也是如此。

然后，每个对等方都会保留两个描述：描述自身的 **本地描述** ，以及描述通话另一端的 **远程描述** 。

当首次建立呼叫时，以及在需要更改呼叫的格式或其他配置时，都会执行提议/回答过程。无论是新呼叫还是重新配置现有呼叫，以下是必须发生的基本步骤，以交换提议和回答，暂时不考虑 ICE 层：

01. 呼叫者通过 `MediaDevices.getUserMedia` 捕获本地媒体。
02. 呼叫者创建 `RTCPeerConnection` 并调用 `RTCPeerConnection.addTrack()`（由于 `addStream` 正在被弃用）。
03. 呼叫者调用 `RTCPeerConnection.createOffer()` 创建提议。
04. 呼叫者调用 `RTCPeerConnection.setLocalDescription()` 将该提议设置为 **本地描述** （即连接本地端的描述）。
05. 在设置本地描述后，呼叫者要求 STUN 服务器生成 ICE 候选。
06. 呼叫者使用信令服务器将提议传输给预期的接收者。
07. 接收者收到提议并调用 `RTCPeerConnection.setRemoteDescription()` 将其记录为 **远程描述** （即连接另一端的描述）。
08. 接收者进行其端呼叫所需的任何设置：捕获其本地媒体，并通过 `RTCPeerConnection.addTrack()` 将每个媒体轨道附加到对等连接。
09. 接收者通过调用 `RTCPeerConnection.createAnswer()` 创建回答。
10. 接收者调用 `RTCPeerConnection.setLocalDescription()`，将创建的回答作为其本地描述。接收者现在知道了连接两端的配置。
11. 接收者使用信令服务器将回答发送给呼叫者。
12. 呼叫者收到回答。
13. 呼叫者调用 `RTCPeerConnection.setRemoteDescription()` 将回答设置为其端呼叫的远程描述。它现在知道了两个对等方的配置。媒体开始按照配置流动。

### 待处理和当前描述

深入探讨这一过程，我们发现 `localDescription` 和 `remoteDescription` 这两个属性返回的描述并不像看上去那么简单。因为在重新协商期间，提议可能会因为提议的格式不兼容而被拒绝，所以每个端点都需要有能力提出新格式，但在另一对等方接受之前不要实际切换到新格式。为此，WebRTC 使用了 **待处理** 和 **当前** 描述。

**当前描述** （由 `RTCPeerConnection.currentLocalDescription` 和 `RTCPeerConnection.currentRemoteDescription` 属性返回）表示连接当前实际使用的描述。这是双方完全同意使用的最新连接。

**待处理描述** （由 `RTCPeerConnection.pendingLocalDescription` 和 `RTCPeerConnection.pendingRemoteDescription` 返回）表示在调用 `setLocalDescription()` 或 `setRemoteDescription()` 后处于考虑中的描述。

当读取描述（由 `RTCPeerConnection.localDescription` 和 `RTCPeerConnection.remoteDescription` 返回）时，如果存在待处理描述（即待处理描述不是 `null`），则返回值为 `pendingLocalDescription`/ `pendingRemoteDescription`；否则，返回当前描述（ `currentLocalDescription`/ `currentRemoteDescription`）。

当通过调用 `setLocalDescription()` 或 `setRemoteDescription()` 更改描述时，指定的描述被设置为待处理描述，WebRTC 层开始评估其是否可接受。一旦提议的描述被同意，`currentLocalDescription` 或 `currentRemoteDescription` 的值将更改为待处理描述，待处理描述将再次被设置为 null，表示没有待处理描述。

**注意：**
`pendingLocalDescription` 不仅包含正在考虑的提议或回答，还包括自提议或回答创建以来已经收集的任何本地 ICE 候选。同样，`pendingRemoteDescription` 包括通过调用 `RTCPeerConnection.addIceCandidate()` 提供的任何远程 ICE 候选。

有关这些属性和方法的更多详细信息，请参阅各自的文档，并参阅 WebRTC 使用的编解码器，了解 WebRTC 支持的编解码器以及与哪些浏览器兼容。编解码器指南还提供了一些建议，以帮助你选择最适合需求的编解码器。

## ICE 候选

除了交换有关媒体的信息（在上面的提议/回答和 SDP 中讨论过），对等方还必须交换有关网络连接的信息。这被称为 **ICE 候选** ，并详细说明了对等方能够通信的可用方法（直接或通过 TURN 服务器）。通常，每个对等方会首先提出其最佳候选，然后逐步提出更差的候选。理想情况下，候选是 UDP（因为它更快，而且媒体流能够相对容易地从中断中恢复），但 ICE 标准也允许 TCP 候选。

**注意：**
通常，只有在 UDP 不可用或受到限制，使其不适合媒体流时，才会使用 TCP 候选。然而，并非所有浏览器都支持 ICE over TCP。

ICE 允许候选代表 TCP 或 UDP 上的连接，通常更喜欢 UDP（并且 UDP 也更广泛地被支持）。每种协议都支持几种类型的候选，候选类型定义了数据如何从对等方传到对等方。

### UDP 候选类型

协议设置为 `udp` 的 UDP 候选可以是以下类型之一：

`host`

主机候选的 `ip` 地址是远程对等方的直接实际 IP 地址。

`prflx`

对等方反射候选的 IP 地址来自两个对等方之间的对称 NAT，通常是在 trickle ICE 期间（即主要信令之后但在连接验证阶段完成之前）作为额外候选提出的。

`srflx`

服务器反射候选是由 STUN/TURN 服务器生成的；连接的发起者向 STUN 服务器请求候选，STUN 服务器通过远程对等方的 NAT 转发请求，NAT 创建并返回一个 IP 地址属于远程对等方的候选。然后 STUN 服务器回复发起者的请求，提供一个 IP 地址与远程对等方无关的候选。

`relay`

中继候选的生成方式与服务器反射候选（`"srflx"`）相同，但使用 TURN 而不是 STUN。

### TCP 候选类型

协议为 `tcp` 的 TCP 候选可以是以下类型之一：

`active`

传输将尝试打开出站连接，但不会接收传入连接请求。这是最常见的类型，也是大多数用户代理将收集的唯一类型。

`passive`

传输将接收传入连接尝试，但不会尝试自己建立连接。

`so`

传输将尝试与对等方同时建立连接。

### 选择候选对

ICE 层选择两个对等方之一作为 **控制代理** 。这是将最终决定使用哪个候选对的 ICE 代理。另一个对等方称为 **受控代理** 。你可以通过检查 `RTCIceCandidate.transport.role` 的值来确定你的连接端是哪一种，尽管一般来说，哪一个是哪一种并不重要。

控制代理不仅负责最终决定使用哪个候选对，还负责通过 STUN 和更新的提议（如果需要）向受控代理发出该选择的信号。受控代理只需等待被告知使用哪个候选对。

需要注意的是，单个 ICE 会话可能导致控制代理选择多个候选对。每次它这样做并将该信息与受控代理共享时，两个对等方将重新配置其连接以使用新候选对描述的新配置。

一旦 ICE 会话完成，当前生效的配置是最终配置，除非发生 ICE 重置。

在每次生成候选后，会发送一个结束候选通知，形式为 `RTCIceCandidate`，其 `candidate` 属性为空字符串。此候选仍应像往常一样使用 `addIceCandidate()` 方法添加到连接中，以便将该通知传递给远程对等方。

如果在当前协商交换期间预计没有更多候选，将通过发送 `RTCIceCandidate` 结束候选通知，其 `candidate` 属性为 `null` 。此消息不需要发送给远程对等方。这是一种遗留通知，可以通过观察 `iceGatheringState` 是否更改为 `complete` 来检测该状态，方法是观察 `icegatheringstatechange` 事件。

## 当事情出错时

在协商过程中，有时事情会不顺利。例如，在重新协商连接时——例如，以适应不断变化的硬件或网络配置——协商可能会陷入僵局，或者发生某种错误，阻止协商 altogether。也可能存在权限问题或其他问题。

### ICE 回滚

当重新协商一个已经活跃的连接时，如果出现协商失败的情况，你可能不希望终止已经进行的通话。毕竟，你可能只是试图升级或降级连接，或者对正在进行的会话进行其他调整。在这种情况下，终止通话将是一种过度反应。

相反，你可以发起 **ICE 回滚** 。回滚将 SDP 提议（以及连接配置）恢复到连接的 `signalingState` 最后一次为 `stable` 时的配置。

要通过编程发起回滚，请发送一个类型为 `rollback` 的描述。描述对象中的其他属性将被忽略。

此外，当一个之前已经创建了提议的对等方收到远程对等方的提议时，ICE 代理将自动发起回滚。换句话说，如果本地对等方处于 `have-local-offer` 状态，表明本地对等方之前已经发送了提议，调用 `setRemoteDescription()` 并收到提议将触发回滚，以便协商从远程对等方作为呼叫者切换到本地对等方作为呼叫者。



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

