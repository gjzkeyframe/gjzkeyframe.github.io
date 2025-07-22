---
title: 音视频面试题集锦第 28 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 04:58:18 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦,  面试, 音视频]
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




我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。


下面是第 28 期面试题精选，我们来看看在跨平台音视频 SDK 开发常用到的 WebRTC 的几道面试题：

- **1、WebRTC 中的 ICE 作用？**
- **2、WebRTC 中媒体协商过程？**
- **3、WebRTC NAT 有几种类型？**
- **4、WebRTC 中的 GCC 机制？**




## 1、WebRTC 中的 ICE 作用？

WebRTC 中的 ICE（Interactive Connectivity Establishment）服务器起着至关重要的作用，主要负责帮助位于不同网络环境（尤其是 NAT 之后）的两个端点建立直接的连接。ICE 服务器通常结合 STUN（Session Traversal Utilities for NAT）和 TURN（Traversal Using Relays around NAT）两种服务来实现这一目标。

- **STUN 服务器**：它允许客户端发现其在 NAT 后的公共地址和端口，从而尝试直接通信。STUN 协议通过简单的请求-响应机制，使得客户端能够获取到其在 NAT 之后的公网地址和端口信息。
- **TURN 服务器**：当直接通信无法实现时，TURN 服务器作为中继服务，接收一端的数据并转发给另一端，从而绕过 NAT 的限制。这在 STUN 无法找到公网 IP 或者 NAT 类型为对称型 NAT 的情况下尤其重要。


ICE 服务器的作用可以总结为以下几点：

- 通过 STUN 和 TURN 服务，帮助不同网络环境下的端点发现彼此并建立连接。
- 在 NAT 穿透过程中，ICE 框架通过一系列交互操作，尝试找到最佳的连接路径。
- 通过信令服务器交换候选地址信息，以便两端能够互相找到对方并建立连接。

ICE 服务器的搭建和配置相对复杂，涉及到多个步骤，包括但不限于：

- 下载并安装 Coturn 服务，Coturn 是一个开源的 ICE 服务器，同时具备 STUN 和 TURN 的功能。
- 处理证书问题，配置服务器以使用 TLS 加密。
- 配置监听 IP、端口以及用户名和密码等安全设置。
- 测试 ICE 服务器是否能够正常工作，确保 STUN 和 TURN 服务均已正确配置并提供服务。

总的来说，ICE 服务器在 WebRTC 中扮演着至关重要的角色，通过结合 STUN 和 TURN 服务，它能够适应不同的网络环境，实现端点间的有效通信。





## 2、WebRTC 中媒体协商过程？


在 WebRTC 中媒体协商（Media Negotiation）的过程是建立音视频通话的关键步骤之一。这个过程允许两个对等端（Peers）交换媒体会话的元数据，例如编解码器信息、媒体类型（音频或视频）、分辨率、帧率等。以下是媒体协商过程的一般步骤：

- **创建 Offer**：一个对等端（通常是发起方）使用 RTCPeerConnection 对象的 createOffer 方法创建一个会话描述（Session Description）。这个描述包含了媒体流的初始参数。
- **设置 Local Description**：一旦 Offer 被创建，它将被设置为本地描述，使用 setLocalDescription 方法。这一步会触发一个事件，告知对等端一个新的 Offer 已经准备好了。
- **发送 Offer**：创建的 Offer 通过信令服务器（Signaling Server）发送给另一个对等端。信令服务器通常是一个外部服务，负责协调两个对等端之间的通信。
- **接收 Offer 和创建 Answer**：接收方收到 Offer 后，使用 RTCPeerConnection 对象的 createAnswer 方法创建一个 Answer。这个 Answer 响应了 Offer 中的提议，并可能包含自己的媒体参数。
- **设置 Remote Description**：接收方使用 setRemoteDescription 方法将接收到的 Offer 设置为远程描述。这会告知 RTCPeerConnection 对方提出的媒体会话参数。
- **设置 Answer 的 Local Description**：Answer 创建后，接收方使用 setLocalDescription 方法将其设置为本地描述。
- **发送 Answer**：Answer 同样通过信令服务器发送给原始的对等端。
- **接收 Answer 和设置 Remote Description**：原始的对等端接收到 Answer 后，使用 setRemoteDescription 方法将其设置为远程描述，完成媒体协商过程。
- **ICE Candidate 交换**：在媒体协商的同时，两个对等端还会交换 ICE Candidate信息，以便通过 ICE 框架找到最佳的通信路径。
- **连接建立和媒体流传输**：一旦媒体协商完成，两个对等端就可以开始通过建立的连接传输媒体流了。

媒体协商过程是 WebRTC 中实现点对点通信的核心机制之一，它确保了两个对等端能够就媒体会话的参数达成一致，并开始实时的音视频通信。




## 3、WebRTC NAT 有几种类型？

在 WebRTC 中 NAT 穿越是一个关键技术，它允许位于不同网络地址转换（NAT）设备后的两个端点建立直接的连接。NAT穿越主要涉及以下几种类型：

- **完全锥型（Full Cone NAT）**：这种类型的 NAT 设备会将来自同一内部 IP 地址和端口的所有请求映射到同一个外部 IP 地址和端口。任何外部主机通过向映射的外部地址发送数据包，都可以实现与内部主机的通信。
- **地址限制锥型（Address-Restricted Cone NAT）**：与完全锥型类似，但只允许与内部主机先前通信过的特定外部 IP 地址通过映射的外部端口进行通信。
- **端口限制锥型（Port-Restricted Cone NAT）**：这种 NAT 类型进一步限制了端口限制锥型 NAT，只允许与内部主机先前通信过的特定外部IP地址和端口通过映射的外部端口进行通信。
- **对称型 NAT（Symmetric NAT）**：这是最严格的 NAT 类型，它要求每次从内部主机到外部特定主机和端口的连接都在 NAT 上创建一个新的映射。这使得每个新的连接都有不同的外部地址和端口映射，因此，除非两个端点都使用中继服务，否则它们无法直接通信。

为了实现 NAT 穿越，WebRTC 采用了 ICE（Interactive Connectivity Establishment）框架，它利用了 STUN（Session Traversal Utilities for NAT）和 TURN（Traversal Using Relays around NAT）协议。STUN 协议帮助端点发现它们在 NAT 后的公网地址和端口，而 TURN 协议则在直接连接无法建立时，通过中继服务器来转发数据包。在实际应用中，NAT 穿越的成功与否取决于端点所处的 NAT 类型以及它们所使用的 NAT 穿越技术。ICE 框架会尝试使用多种方法来建立连接，包括直接连接、通过 STUN 和 TURN 服务中继，以确保通信的建立。





## 4、WebRTC 中的 GCC 机制？


WebRTC 中的 GCC（Google Congestion Control）算法是一种重要的拥塞控制机制，用于自适应地调整视频流的码率，以适应网络条件的变化，确保音视频通信的流畅性和清晰度。以下是一些关键点来实现 WebRTC 中的码率自适应优化：

- **基于RTCP反馈的码率控制**：WebRTC 通过 RTCP（Real-time Transport Control Protocol）反馈信息来评估网络状况。发送端根据接收端的 RTCP 反馈调整码率，以响应网络的拥塞情况。
- **REMB（Receiver Estimated Maximum Bitrate）**：接收端估算其能够承受的最大码率，并通过 RTCP REMB 消息将此信息反馈给发送端，发送端据此调整码率。
- **发送端的码率控制**：发送端使用基于丢包率的算法来评估带宽，并与接收端的 REMB 信息结合，选择较小的码率作为目标发送码率。


![码率计算和生效过程](assets/resource/av-interview-qa/gcc-1.webp)


![发送端码率计算过程](assets/resource/av-interview-qa/gcc-2.webp)

- **拥塞窗口调整**：PacingController 使用拥塞窗口来控制发送速率，当检测到发送中的数据量超过拥塞窗口时，会减少发送速率以避免拥塞。
- **码率分配**：BitrateAllocator 负责将目标码率分配给不同的视频流，考虑每个流的最小和最大码率限制，以及媒体和保护（如 FEC 和重传）的码率需求。
- **自适应策略**：WebRTC 提供了不同的自适应策略，如 MAINTAIN_FRAMERATE、MAINTAIN_RESOLUTION 和 BALANCED，根据当前的使用场景和资源状况动态调整帧率和分辨率。
- **资源检测与处理器**：WebRTC 内部通过实时检查多种资源情况，使用资源检测器（如 EncodeUsageResource、QualityScalerResource 等）和处理器（ResourceAdaptationProcessor）来决定输入源应该采取哪些动作来自适应资源变化。

通过这些机制，WebRTC 可以在不同网络环境下动态调整视频质量，优化用户体验。






---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![限时优惠，扫码加入](assets/img/keyframe-zsxq.png)











---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

