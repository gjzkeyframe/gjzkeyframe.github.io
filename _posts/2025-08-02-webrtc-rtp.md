---
title: 探索 WebRTC 开发（2）：WebRTC 实时传输协议
description: 本文将介绍 WebRTC 的实时传输协议。
author: Keyframe
date: 2025-08-02 18:08:08 +0800
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


**实时传输协议**（**RTP**，Real-time Transport Protocol），定义于 RFC 3550 中，是 IETF 的标准协议，用于实现需要实时优先级的数据交换的实时连接。本文将概述 RTP 的功能及其在 WebRTC 中的应用。

**注意：**
WebRTC 实际上使用 **SRTP**（安全实时传输协议）来确保数据的安全性和认证。

对于 WebRTC 而言，尽量降低延迟尤为重要，因为面对面的交流需要尽可能减少延迟。如果一个用户说话和另一个用户听到之间的时间差越大，就越容易出现互相打断和其他形式的混乱。

## RTP 的主要特性

在探讨 RTP 在 WebRTC 中的应用之前，先大致了解 RTP 的功能和不提供的功能是有益的。RTP 是一种数据传输协议，其使命是在当前条件下尽可能高效地在两个端点之间移动数据。这些条件可能受到网络堆栈的底层、物理网络连接、中间网络、远程端点的性能、噪声水平、流量水平等因素的影响。

由于 RTP 是一种数据传输协议，因此它得到了紧密相关的 **RTP 控制协议**（**RTCP**）的增强，RTCP 定义在 RFC 3550 的第 6 节中。RTCP 添加了包括 **服务质量**（**QoS**）监测、参与者信息共享等功能。它不足以完全管理用户、成员资格、权限等，但为不受限制的多用户通信会话提供了所需的基本功能。

RTCP 和 RTP 在同一个 RFC 中定义，这表明这两个协议之间的紧密关联。

### RTP 的功能

RTP 在 WebRTC 中的主要优势包括：

- 通常低延迟。
- 数据包按序列编号和时间戳进行重组，如果数据包乱序到达，这可以让使用 RTP 发送的数据在不保证顺序甚至不保证交付的传输中进行交付。
- 这意味着 RTP 可以 — 但不是必须 — 用于 UDP 之上，以利用其性能以及其复用和校验和特性。
- RTP 支持多播；虽然这对于 WebRTC 目前还不重要，但随着 WebRTC（希望）增强以支持多用户会话，这在未来可能会变得重要。
- RTP 并不限于在视听通信中使用。它可以用于任何形式的连续或活动数据传输，包括数据流、活动徽章或状态显示更新，或控制和测量信息传输。

### RTP 不提供的功能

RTP 本身并不提供所有可能的功能，这就是为什么 WebRTC 还使用其他协议。RTP 不包括的一些值得注意的功能：

- RTP 并不保证 **服务质量**（**QoS**）。
- 虽然 RTP 是为延迟关键场景设计的，但它本身并不提供任何确保 QoS 的功能。相反，它只提供必要的信息，以便在堆栈的其他地方实现 QoS。
- RTP 不处理可能需要的资源分配或预留。

对于 WebRTC 的目的，这些问题在 WebRTC 基础设施内的各个地方得到处理。例如，RTCP 处理 QoS 监测。

## RTCPeerConnection 和 RTP

每个 `RTCPeerConnection` 都有方法可以访问服务对等连接的 RTP 传输列表。这些方法对应于 `RTCPeerConnection` 支持的以下三种传输类型：

`RTCRtpSender`

`RTCRtpSender` 处理将 `MediaStreamTrack` 数据编码并传输给远程对等方。可以通过调用 `RTCPeerConnection.getSenders()` 获取特定连接的发送器。

`RTCRtpReceiver`

`RTCRtpReceiver` 提供检查和获取传入 `MediaStreamTrack` 数据信息的功能。可以通过调用 `RTCPeerConnection.getReceivers()` 获取连接的接收器。

`RTCRtpTransceiver`

`RTCRtpTransceiver` 是一个 RTP 发送器和一个 RTP 接收器的组合，它们共享一个 SDP `mid` 属性，这意味着它们共享同一个 SDP 媒体 m-line（表示一个双向 SRTP 流）。这些通过 `RTCPeerConnection.getTransceivers()` 方法返回，每个 `mid` 和收发器之间有一对一的关系，每个 `mid` 在每个 `RTCPeerConnection` 中是唯一的。

### 利用 RTP 实现保留功能

由于 `RTCPeerConnection` 的流是使用 RTP 和上述接口实现的，因此可以利用对流内部的访问来进行调整。可以做的最简单的事情之一是实现一个“保留”功能，其中通话中的参与者可以点击一个按钮，关闭他们的麦克风，开始向另一个对等方发送音乐，而不是接受传入的音频。

**注意：**
这个例子使用了现代 JavaScript 特性，包括异步函数和 `await` 表达式。这极大地简化并使处理 WebRTC 方法返回的承诺的代码更具可读性。

在下面的例子中，我们将把打开和关闭“保留”模式的对等方称为本地对等方，而被保留的用户称为远程对等方。

#### 激活保留模式

##### 本地对等方

当本地用户决定启用保留模式时，将调用下面的 `enableHold()` 方法。它接受一个包含保留时要播放的音频的 `MediaStream` 作为输入。

```javascript
async function enableHold(audioStream) {
  try {
    await audioTransceiver.sender.replaceTrack(audioStream.getAudioTracks()[0]);
    audioTransceiver.receiver.track.enabled = false;
    audioTransceiver.direction = "sendonly";
  } catch (err) {
    /* 处理错误 */
  }
}
```

`try` 块中的三行代码执行以下步骤：

1. 将他们的传出音频轨道替换为包含保留音乐的 `MediaStreamTrack`。
2. 禁用传入音频轨道。
3. 将音频收发器切换为仅发送模式。

这会触发 `RTCPeerConnection` 的重新协商，通过向其发送 `negotiationneeded` 事件，你的代码将响应生成 SDP 提供，使用 `RTCPeerConnection.createOffer` 并通过信令服务器将其发送给远程对等方。

包含要播放的音频而不是本地对等方麦克风音频的 `audioStream` 可以来自任何地方。一种可能性是使用隐藏的 `<audio>` 元素，并使用 `HTMLAudioElement.captureStream()` 获取其音频流。

##### 远程对等方

在远程对等方，当我们收到方向设置为 `"sendonly"` 的 SDP 提供时，我们将使用 `holdRequested()` 方法处理它，该方法接受一个 SDP 提供字符串作为输入。

```javascript
async function holdRequested(offer) {
  try {
    await peerConnection.setRemoteDescription(offer);
    await audioTransceiver.sender.replaceTrack(null);
    audioTransceiver.direction = "recvonly";
    await sendAnswer();
  } catch (err) {
    /* 处理错误 */
  }
}
```

这里执行的步骤是：

1. 通过调用 `RTCPeerConnection.setRemoteDescription()` 将远程描述设置为指定的 `offer`。
2. 将音频收发器的 `RTCRtpSender` 的轨道替换为 `null`，这意味着没有轨道。这停止了收发器上的音频发送。
3. 将音频收发器的 `direction` 属性设置为 `"recvonly"`，指示收发器仅接受音频，不发送任何音频。
4. 使用名为 `sendAnswer()` 的方法生成并发送 SDP 回答，该方法使用 `createAnswer()` 生成回答，然后通过信令服务将结果 SDP 发送给另一个对等方。

#### 取消保留模式

##### 本地对等方

当本地用户点击界面小部件以禁用保留模式时，将调用 `disableHold()` 方法，以开始恢复正常功能的过程。

```javascript
async function disableHold(micStream) {
  await audioTransceiver.sender.replaceTrack(micStream.getAudioTracks()[0]);
  audioTransceiver.receiver.track.enabled = true;
  audioTransceiver.direction = "sendrecv";
}
```

这通过执行以下步骤来逆转 `enableHold()` 中的操作：

1. 将音频收发器的 `RTCRtpSender` 的轨道替换为指定流的第一个音频轨道。
2. 重新启用收发器的传入音频轨道。
3. 将音频收发器的方向设置为 `"sendrecv"`，表示它应该恢复发送和接收流式音频，而不仅仅是发送。

就像启用保留时一样，这会再次触发协商，导致你的代码向远程对等方发送新的提供。

##### 远程对等方

当远程对等方收到 `"sendrecv"` 提供时，它将调用其 `holdEnded()` 方法：

```javascript
async function holdEnded(offer, micStream) {
  try {
    await peerConnection.setRemoteDescription(offer);
    await audioTransceiver.sender.replaceTrack(micStream.getAudioTracks()[0]);
    audioTransceiver.direction = "sendrecv";
    await sendAnswer();
  } catch (err) {
    /* 处理错误 */
  }
}
```

这里 `try` 块中的步骤是：

1. 通过调用 `setRemoteDescription()` 将收到的提供存储为远程描述。
2. 使用音频收发器的 `RTCRtpSender` 的 `replaceTrack()` 方法将传出音频轨道设置为麦克风音频流的第一个轨道。
3. 将收发器的方向设置为 `"sendrecv"`，表示它应该恢复发送和接收音频。

从这一点开始，麦克风重新启用，远程用户再次能够听到本地用户并与之交谈。

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

