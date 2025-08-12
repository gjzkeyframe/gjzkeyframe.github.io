---
title: 探索 WebRTC 开发（1）：WebRTC 协议简介
description: 本文将介绍 WebRTC API 所依赖的协议。
author: Keyframe
date: 2025-08-01 18:08:08 +0800
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



## ICE

交互式连接建立（Interactive Connectivity Establishment，简称 ICE）是一种框架，允许你的网络浏览器与对等方建立连接。直接从对等方 A 连接到对等方 B 可能会遇到诸多问题，比如需要绕过阻止建立连接的防火墙，在大多数设备没有公网 IP 地址的情况下为你提供一个唯一的地址，以及在路由器不允许直接连接对等方时通过服务器中继数据。ICE 使用 STUN 和/或 TURN 服务器来实现这些功能，如下所述。

## STUN

会话穿越实用工具（Session Traversal Utilities for NAT，简称 STUN）是一种协议，用于发现你的公网地址，并确定路由器中的任何限制，这些限制可能会阻止与对等方的直接连接。

客户端会向互联网上的 STUN 服务器发送请求，服务器会回复客户端的公网地址以及客户端是否在路由器的 NAT 后可访问。

![STUN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Protocols/webrtc-stun.png)
_STUN_

## NAT

网络地址转换（Network Address Translation，简称 NAT）用于为你的设备分配一个公网 IP 地址。路由器会有一个公网 IP 地址，连接到路由器的每个设备都会有一个私有 IP 地址。请求将从设备的私有 IP 转换为路由器的公网 IP，并附带一个唯一的端口。这样，即使你没有为每个设备分配唯一的公网 IP，仍然可以在互联网上被发现。

有些路由器会对可以连接到网络中设备的对等方进行限制。这意味着即使我们通过 STUN 服务器找到了公网 IP 地址，也可能不是任何人都能建立连接。在这种情况下，我们需要使用 TURN。

## TURN

一些使用 NAT 的路由器采用了一种称为“对称 NAT”的限制。这意味着路由器只接受你之前连接过的对等方的连接。

Traversal Using Relays around NAT（简称 TURN）旨在通过与 TURN 服务器建立连接并中继所有信息来绕过对称 NAT 的限制。你会与 TURN 服务器建立连接，并告诉所有对等方将数据包发送到服务器，服务器再将数据包转发给你。这显然会带来一些开销，因此只有在没有其他替代方案时才使用。

![TURN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Protocols/webrtc-turn.png)
_TURN_

## SDP

会话描述协议（Session Description Protocol，简称 SDP）是一种用于描述连接的多媒体内容的标准，例如分辨率、格式、编解码器、加密等，以便在数据传输时双方对等方能够理解彼此。本质上，SDP 是描述内容的元数据，而不是媒体内容本身。

从技术上讲，SDP 并非真正意义上的协议，而是一种用于描述设备之间共享媒体的连接的数据格式。

记录 SDP 超出了本文档的范围；然而，这里有一些值得注意的地方。

### 结构

SDP 由一个或多个 UTF-8 文本行组成，每行以一个字符类型开头，后跟一个等号（`"="`），接着是构成值或描述的结构化文本，其格式取决于类型。以给定字母开头的文本行通常被称为`_letter_-lines`。例如，提供媒体描述的行的类型为`"m"`，因此这些行被称为`m-lines。`


---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

