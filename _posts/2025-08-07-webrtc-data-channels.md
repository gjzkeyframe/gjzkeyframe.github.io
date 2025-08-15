---
title: 探索 WebRTC 开发（7）：WebRTC 数据通道
description: 本文将介绍 WebRTC 如何向对等连接添加数据通道，然后可以用来安全地交换任意数据，即任何我们想要的数据类型和格式。
author: Keyframe
date: 2025-08-07 18:08:08 +0800
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


在这个指南中，我们将探讨如何向对等连接添加数据通道，然后可以用来安全地交换任意数据，即任何我们想要的数据类型和格式。

**注意：**
由于所有 WebRTC 组件都必须使用加密，因此在 `RTCDataChannel` 上传输的任何数据都会自动使用数据报传输层安全（DTLS）加密。有关更多信息，请参阅下面的安全部分。

## 创建数据通道

`RTCDataChannel` 所使用的底层数据传输可以通过以下两种方式之一创建：

- 让 WebRTC 创建传输并为你向远程对等方宣布（通过触发远程对等方接收 `datachannel` 事件）。这是简单的方法，适用于多种使用场景，但可能不够灵活，无法满足你的需求。
- 编写自己的代码来协商数据传输，并编写自己的代码来通知另一个对等方需要连接到新通道。

让我们先来看第一种情况，这是最常见的。

### 自动协商

通常，你可以允许对等连接为你处理 `RTCDataChannel` 连接的协商。为此，调用 `createDataChannel()` 时不要指定 `negotiated` 属性的值，或者将该属性指定为 `false`。这将自动触发 `RTCPeerConnection` 为你处理协商，导致远程对等方创建一个数据通道，并通过网络将两者链接在一起。

`RTCDataChannel` 对象会立即由 `createDataChannel()` 返回；你可以通过观察 `open` 事件是否被发送到 `RTCDataChannel` 来判断连接是否成功建立。

```javascript
let dataChannel = pc.createDataChannel("MyApp Channel");

dataChannel.addEventListener("open", (event) => {
  beginTransmission(dataChannel);
});
```

### 手动协商

要手动协商数据通道连接，你需要首先使用 `RTCPeerConnection` 上的 `createDataChannel()` 方法创建一个新的 `RTCDataChannel` 对象，并在选项中指定 `negotiated` 属性设置为 `true`。这表示告诉对等连接不要试图为你协商通道。

然后通过带外方式（使用网络服务器或其他方式）协商连接。此过程应通知远程对等方，使其创建自己的具有相同 `id` 的 `RTCDataChannel`，并且也将 `negotiated` 属性设置为 `true`。这将通过 `RTCPeerConnection` 将两个对象链接在一起。

```javascript
let dataChannel = pc.createDataChannel("MyApp Channel", {
  negotiated: true,
});

dataChannel.addEventListener("open", (event) => {
  beginTransmission(dataChannel);
});

requestRemoteChannel(dataChannel.id);
```

在这个代码片段中，通道是用 `negotiated` 设置为 `true` 创建的，然后调用 `requestRemoteChannel()` 触发协商，以使用与本地通道相同的 ID 创建远程通道。

这样做可以让你使用不同的属性创建与每个对等方的数据通道，并通过使用相同的 `id` 值来声明性地创建通道。

## 缓冲

WebRTC 数据通道支持对传出数据进行缓冲。这是自动处理的。虽然无法控制缓冲区的大小，但可以了解当前缓冲了多少数据，并且可以选择在缓冲区开始缺少排队数据时通过事件通知。这使得编写高效的例程变得容易，以确保总是有数据准备好发送，而不会过度使用内存或完全淹没通道。

## 理解消息大小限制

对于通过网络传输的任何数据，都有大小限制。在基本层面上，单个网络数据包不能大于某个值（确切的数字取决于网络和所使用的传输层）。在应用层（即你的代码运行的用户代理的 WebRTC 实现中），WebRTC 实现了支持超过网络传输层最大数据包大小的消息的功能。

这可能会使事情变得复杂，因为你不一定知道各种用户代理的消息大小限制是多少，以及它们在发送或接收较大消息时如何响应。即使用户代理使用相同的底层库来处理流控制传输协议（SCTP）数据，由于调用库的方式和对返回错误的反应不同，仍然可能会有差异。例如，Firefox 和 Google Chrome 都使用 `usrsctp` 库来实现 SCTP，但在某些情况下，由于它们调用库的方式和对返回错误的反应不同，`RTCDataChannel` 上的数据传输可能会失败。

当两个运行 Firefox 的用户在数据通道上通信时，消息大小限制比 Firefox 和 Chrome 之间的通信要大得多，因为 Firefox 实现了一种现在已废弃的技术，用于在多个 SCTP 消息中发送大消息，而 Chrome 没有实现这一点。相反，Chrome 会看到一系列它认为是完整的消息，并将它们作为多个消息传递给接收的 `RTCDataChannel`。

小于 16 KiB 的消息可以放心发送，因为所有主要用户代理都以相同的方式处理它们。超过这个大小，事情就会变得复杂。

### 大消息的考虑

目前，使用 `RTCDataChannel` 发送超过 64 KiB 的消息（如果你希望支持跨浏览器的数据交换，则为 16 KiB）是不切实际的。问题在于 SCTP（用于在 `RTCDataChannel` 上发送和接收数据的协议）最初是作为信令协议设计的。预计消息会相对较小。支持超过网络层 MTU 的消息的功能几乎是事后再加的，以防信令消息需要大于 MTU。此功能要求消息的每个部分都有连续的序列号，因此必须一个接一个地传输，中间不能交错传输其他数据。

最终，这变成了一个问题。随着时间的推移，各种应用程序（包括实现 WebRTC 的应用程序）开始通过 SCTP 传输越来越大消息。最终意识到，当消息变得太大时，大消息的传输可能会阻塞该数据通道上的所有其他数据传输，包括关键的信令消息。

当浏览器正确支持当前标准中用于支持大消息的功能时，这将成为一个问题——即 end-of-record（EOR）标志，用于指示一系列消息中的最后一个消息应被视为单个有效载荷。Firefox 57 中实现了此功能，但 Chrome 尚未实现（请参阅 Chromium Bug 7774）。有了 EOR 支持，`RTCDataChannel` 有效载荷可以大得多（官方上限为 256 KiB，但 Firefox 的实现将其限制为高达 1 GiB）。即使在 256 KiB，也可能导致处理紧急流量的明显延迟。如果你发送更大的消息，除非你确定操作条件，否则延迟可能会变得难以忍受。

为了解决这个问题，已经设计了一种新的流调度程序系统（通常称为“SCTP ndata 规范”），以便在不同的流上交错传输消息，包括用于实现 WebRTC 数据通道的流。此提案仍处于 IETF 草案形式，但一旦实现，它将使发送几乎无大小限制的消息成为可能，因为 SCTP 层将自动交错底层子消息，以确保每个通道的数据都有机会通过。

Firefox 对 ndata 的支持正在实施中；请参阅 Firefox bug 1381145 以跟踪其对通用使用的可用性。Chrome 团队在其位中跟踪了 ndata 支持的实现进度。

**注意：**
本节中的部分信息基于 Lennart Grahl 撰写的博客文章《揭示 WebRTC 数据通道消息大小限制的奥秘》。他在那里进行了更详细的讨论，但自浏览器更新以来，其中部分内容可能已过时。此外，随着时间的推移，它会变得更加过时，特别是一旦主要浏览器完全集成了 EOR 和 ndata 支持。

## 安全

使用 WebRTC 传输的所有数据都是加密的。对于 `RTCDataChannel`，使用的加密是基于传输层安全（TLS）的数据报传输层安全（DTLS）。由于 TLS 用于保护每个 HTTPS 连接，因此通过数据通道发送的任何数据都与用户的浏览器发送或接收的其他数据一样安全。

更基本的是，由于 WebRTC 是两个用户代理之间的点对点连接，数据永远不会通过网络或应用程序服务器。这减少了数据被拦截的机会。



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

