---
title: 探索 WebRTC 开发（8）：WebRTC DTMF
description: 本文将介绍 DTMF 如何在 WebRTC 上工作。
author: Keyframe
date: 2025-08-08 18:08:08 +0800
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

为了更全面地支持音频/视频会议，WebRTC 支持在 RTCPeerConnection 上向远程对等方发送 DTMF。本文提供了关于 DTMF 如何在 WebRTC 上工作的高层次概述，然后为日常开发者提供了如何在 RTCPeerConnection 上发送 DTMF 的指南。DTMF 系统通常被称为“触摸音调”，这是该系统的一个旧商标名。

WebRTC 不会将 DTMF 代码作为音频数据发送。相反，它们作为 RTP 负载带外发送。然而，虽然可以通过 WebRTC 发送 DTMF，但目前没有方法检测或接收传入的 DTMF。WebRTC 目前会忽略这些负载；这是因为 WebRTC 的 DTMF 支持主要旨在与依赖 DTMF 音调执行任务的遗留电话服务一起使用，例如：

- 电话会议系统
- 菜单系统
- 语音邮件系统
- 信用卡或其他支付信息的输入
- 密码输入

**注意：**
虽然 DTMF 不会作为音频发送到远程对等方，但浏览器可能会选择为本地用户播放相应的音调，作为用户体验的一部分，因为用户通常习惯于听到手机播放的音调。

## 在 RTCPeerConnection 上发送 DTMF

一个给定的 RTCPeerConnection 可以在上面发送或接收多个媒体轨道。当你希望传输 DTMF 信号时，你首先需要决定在哪个轨道上发送它们，因为 DTMF 作为一系列带外负载在负责传输该轨道数据的 RTCRtpSender 上发送。

一旦选择了轨道，就可以从其 RTCDTMFSender 中获取 RTCDTMFSender 对象，用于发送 DTMF。从那里，可以调用 RTCDTMFSender.insertDTMF() 将 DTMF 信号排队发送到该轨道上的对等方。RTCRtpSender 然后将音调作为数据包与轨道的音频数据一起发送到对等方。

每次发送音调时，RTCPeerConnection 会收到一个 tonechange 事件，其 tone 属性指定了哪个音调停止播放，这是一个机会来更新界面元素，例如。当音调缓冲区为空时，表明所有音调都已发送，一个 tonechange 事件将被传递到连接对象，其 tone 属性设置为""（空字符串）。

如果您想了解更多关于 DTMF 如何工作的信息，请阅读 RFC 3550：RTP：实时应用程序的传输协议和 RFC 4733：RTP 负载用于 DTMF 数字、电话音调和电话信号。DTMF 负载在 RTP 上的处理细节超出了本文的范围。相反，我们将专注于如何在 RTCPeerConnection 的上下文中使用 DTMF，通过研究一个示例来学习。

## 简单示例

这个简单示例构建了两个 RTCPeerConnection，并在它们之间建立连接，然后等待用户点击“拨号”按钮。当按钮被点击时，使用 RTCDTMFSender.insertDTMF() 在连接上发送 DTMF 字符串。一旦音调传输完成，连接将关闭。

**注意：**
这个示例显然是有点作弊的，因为它在一个代码流中生成了两个对等方，而不是每个对等方作为一个真正独立的实体。

### HTML

这个示例的 HTML 非常基础；只有三个元素是重要的：

- 一个 `<audio>` 元素用于播放被“呼叫”的 RTCPeerConnection 接收到的音频。
- 一个 `<button>` 元素用于触发创建和连接两个 RTCPeerConnection 对象，然后发送 DTMF 音调。
- 一个 `<div>` 用于接收和显示日志文本以显示状态信息。

```html
<p>
  此示例演示了 WebRTC 中 DTMF 的使用。请注意，此示例通过在一个代码流中生成两个对等方来作弊，而不是每个对等方作为一个真正独立的实体。
</p>

<audio id="audio" autoplay controls></audio><br />
<button name="dial" id="dial">Dial</button>

<div class="log"></div>
```

### JavaScript

让我们看看 JavaScript 代码。请注意，建立连接的过程在这里是有点虚构的；你通常不会在同一文档中构建连接的两个端点。

#### 全局变量

首先，我们建立全局变量。

```javascript
let dialString = "12024561111";

let callerPC = null;
let receiverPC = null;
let dtmfSender = null;

let hasAddTrack = false;

let mediaConstraints = {
  audio: true,
  video: false,
};

let offerOptions = {
  offerToReceiveAudio: 1,
  offerToReceiveVideo: 0,
};

let dialButton = null;
let logElement = null;
```

这些是按顺序的：

`dialString`

呼叫方在点击“拨号”按钮时发送的 DTMF 字符串。

`callerPC` 和 `receiverPC`

分别表示呼叫方和接收方的 RTCPeerConnection 对象。这些将在连接启动时在 `connectAndDial()` 函数中初始化，如下文“开始连接过程”部分所示。

`dtmfSender`

用于连接的 RTCDTMFSender 对象。这将在设置连接时在 `gotStream()` 函数中获取，如下文“将音频添加到连接”部分所示。

`hasAddTrack`

由于某些浏览器尚未实现 RTCPeerConnection.addTrack()，因此需要使用已过时的 addStream() 方法，我们使用这个布尔值来确定用户代理是否支持 addTrack()；如果它不支持，我们将回退到 addStream()。这将在 `connectAndDial()` 中确定，如下文“开始连接过程”部分所示。

`mediaConstraints`

一个指定启动连接时使用的约束的对象。我们希望一个仅音频的连接，所以 `video` 为 `false`，而 `audio` 为 `true`。

`offerOptions`

一个提供选项的对象，用于指定调用 RTCPeerConnection.createOffer() 时的行为。在这种情况下，我们声明我们希望接收音频但不接收视频。

`dialButton` 和 `logElement`

这些变量用于存储对拨号按钮和日志输出框元素的引用。它们将在页面加载时设置。参见下文“初始化”部分。

#### 初始化

当页面加载时，我们进行一些基本设置：我们获取对拨号按钮和日志输出框元素的引用，并使用 addEventListener() 为拨号按钮添加一个事件监听器，以便点击它时调用 `connectAndDial()` 函数以开始连接过程。

```javascript
window.addEventListener("load", () => {
  logElement = document.querySelector(".log");
  dialButton = document.querySelector("#dial");

  dialButton.addEventListener("click", connectAndDial, false);
});
```

#### 开始连接过程

当拨号按钮被点击时，将调用 `connectAndDial()`。这将开始构建 WebRTC 连接，为发送 DTMF 代码做准备。

```javascript
function connectAndDial() {
  callerPC = new RTCPeerConnection();

  hasAddTrack = callerPC.addTrack !== undefined;

  callerPC.onicecandidate = handleCallerIceEvent;
  callerPC.onnegotiationneeded = handleCallerNegotiationNeeded;
  callerPC.oniceconnectionstatechange = handleCallerIceConnectionStateChange;
  callerPC.onsignalingstatechange = handleCallerSignalingStateChangeEvent;
  callerPC.onicegatheringstatechange = handleCallerGatheringStateChangeEvent;

  receiverPC = new RTCPeerConnection();
  receiverPC.onicecandidate = handleReceiverIceEvent;

  if (hasAddTrack) {
    receiverPC.ontrack = handleReceiverTrackEvent;
  } else {
    receiverPC.onaddstream = handleReceiverAddStreamEvent;
  }

  navigator.mediaDevices
    .getUserMedia(mediaConstraints)
    .then(gotStream)
    .catch((err) => log(err.message));
}
```

在创建了呼叫方的 `RTCPeerConnection`（`callerPC`）后，我们查看它是否有 `addTrack()` 方法。如果它有，我们将 `hasAddTrack` 设置为 `true`；否则，将其设置为 `false`。这个变量将允许示例即使在不支持较新的 `addTrack()` 方法的浏览器上也能运行；我们将回退到较旧的 `addStream()` 方法。

接下来，为呼叫方建立事件处理程序。我们将在后面详细讨论这些。

然后，我们创建了第二个 `RTCPeerConnection`，这个代表呼叫接收端，并将其存储在 `receiverPC` 中；其 `onicecandidate` 事件处理程序也被设置。

如果支持 `addTrack()`，我们设置接收方的 `ontrack` 事件处理程序；否则，我们设置 `onaddstream`。当媒体被添加到连接时，会发送 `track` 和 `addstream` 事件。

最后，我们调用 `getUserMedia()` 以获取对呼叫方麦克风的访问权限。如果成功，将调用 `gotStream()` 函数，否则我们将记录错误，因为呼叫失败。

#### 将音频添加到连接

如上所述，当从麦克风获取音频输入时，将调用 `gotStream()`。它的任务是构建发送给接收方的流，以便真正开始传输过程。它还获取我们将用于在连接上发出 DTMF 的 RTCDTMFSender。

```javascript
function gotStream(stream) {
  log("Got access to the microphone.");

  let audioTracks = stream.getAudioTracks();

  if (hasAddTrack) {
    if (audioTracks.length > 0) {
      audioTracks.forEach((track) => callerPC.addTrack(track, stream));
    }
  } else {
    log(
      "您的浏览器不支持 RTCPeerConnection.addTrack()。回退到已过时的 addStream() 方法...",
    );
    callerPC.addStream(stream);
  }

  if (callerPC.getSenders) {
    dtmfSender = callerPC.getSenders()[0].dtmf;
  } else {
    log(
      "您的浏览器不支持 RTCPeerConnection.getSenders()，因此回退到使用已过时的 createDTMFSender()。",
    );
    dtmfSender = callerPC.createDTMFSender(audioTracks[0]);
  }

  dtmfSender.ontonechange = handleToneChangeEvent;
}
```

在将用户麦克风流中的音频轨道设置为 `audioTracks` 后，是时候将媒体添加到呼叫方的 `RTCPeerConnection` 中了。如果 `RTCPeerConnection` 上有 `addTrack()` 方法，我们将使用 `RTCPeerConnection.addTrack()` 将流中的每个音频轨道逐一添加到连接中。否则，我们将调用 `RTCPeerConnection.addStream()` 以将流作为整体添加到通话中。

接下来，我们查看 `RTCPeerConnection.getSenders()` 方法是否已实现。如果已实现，我们将其调用在 `callerPC` 上，并获取返回的发送者列表中的第一个条目；这是负责传输通话中第一个音频轨道数据的 `RTCRtpSender`（我们将通过该轨道发送 DTMF）。然后我们获取 `RTCRtpSender` 的 `dtmf` 属性，这是一个可以发送连接 DTMF 的 `RTCDTMFSender` 对象，从呼叫方发送到接收方。

如果 `getSenders()` 不可用，我们将调用 `RTCPeerConnection.createDTMFSender()` 来获取 `RTCDTMFSender` 对象。尽管此方法已过时，但此示例将其作为回退以支持旧版浏览器（以及尚未更新以支持当前 WebRTC DTMF API 的浏览器）。

最后，我们设置了 DTMF 发送者的 `ontonechange` 事件处理程序，以便在每次 DTMF 音调停止播放时收到通知。

您可以在文档底部找到日志函数。

#### 当音调停止播放时

每次 DTMF 音调停止播放时，将向 `callerPC` 发送一个 `tonechange` 事件。这些事件的处理程序是 `handleToneChangeEvent()` 函数。

```javascript
function handleToneChangeEvent(event) {
  if (event.tone !== "") {
    log(`播放的音调：${event.tone}`);
  } else {
    log("所有音调已播放。正在断开连接。");
    callerPC.getLocalStreams().forEach((stream) => {
      stream.getTracks().forEach((track) => {
        track.stop();
      });
    });
    receiverPC.getLocalStreams().forEach((stream) => {
      stream.getTracks().forEach((track) => {
        track.stop();
      });
    });

    audio.pause();
    audio.srcObject = null;
    receiverPC.close();
    callerPC.close();
  }
}
```

`tonechange` 事件用于指示单个音调何时播放完毕以及所有音调何时播放完毕。事件的 `tone` 属性是一个字符串，指示刚刚停止播放的音调。如果所有音调已播放完毕，`tone` 是一个空字符串；在这种情况下，`RTCDTMFSender.toneBuffer` 是空的。

在示例中，我们将屏幕上的日志记录为刚刚停止播放的音调。在更高级的应用中，您可能会更新用户界面，例如，指示当前播放的音符。

另一方面，如果音调缓冲区为空，我们的示例设计为断开通话。这通过停止两个 `RTCPeerConnection` 上的每个流来完成，方法是迭代每个 `RTCPeerConnection` 的轨道列表（由其 `getTracks()` 方法返回）并调用每个轨道的 `stop()` 方法。

一旦呼叫方和接收方的所有媒体轨道都已停止，我们将暂停 `<audio>` 元素并将其 `srcObject` 设置为 `null`。这将音频流从 `<audio>` 元素中分离。

然后，最后，通过调用每个 `RTCPeerConnection` 的 `close()` 方法来关闭连接。

#### 为呼叫方添加候选

当呼叫方的 RTCPeerConnection ICE 层提出新的候选时，它会向 callerPC 发出 icecandidate 事件。icecandidate 事件处理程序的工作是将候选传输到接收方。在示例中，我们直接控制呼叫方和接收方，因此我们可以通过调用接收方的 addIceCandidate() 方法直接将候选添加到接收方。这由 handleCallerIceEvent() 处理。

```javascript
function handleCallerIceEvent(event) {
  if (event.candidate) {
    log(`为接收方添加候选：${event.candidate.candidate}`);

    receiverPC
      .addIceCandidate(new RTCIceCandidate(event.candidate))
      .catch((err) => log(`为接收方添加候选时出错：${err}`));
  } else {
    log("呼叫方没有更多候选。");
  }
}
```

如果 icecandidate 事件的 candidate 属性不是 null，我们将从 event.candidate 字符串中创建一个新的 RTCIceCandidate 对象，并通过将其传递给 receiverPC.addIceCandidate() 来将其“传输”到接收方。如果 addIceCandidate() 失败，catch() 子句将错误输出到我们的日志框。

如果 event.candidate 是 null，这表明没有更多候选可用，我们将此信息记录下来。

#### 拨号后建立连接

我们的设计要求一旦连接建立，我们立即发送 DTMF 字符串。为此，我们等待呼叫方收到 iceconnectionstatechange 事件。此事件在 ICE 连接过程的状态发生变化时发送，包括成功建立连接时。

```javascript
function handleCallerIceConnectionStateChange() {
  log(`呼叫方的连接状态已更改为 ${callerPC.iceConnectionState}`);
  if (callerPC.iceConnectionState === "connected") {
    log(`发送 DTMF："${dialString}"`);
    dtmfSender.insertDTMF(dialString, 400, 50);
  }
}
```

iceconnectionstatechange 事件实际上并不在其内部包含新状态，因此我们从 callerPC 的 RTCPeerConnection.iceConnectionState 属性获取连接过程的当前状态。在记录新状态后，我们查看状态是否为 "connected"。如果是，我们记录即将发送的 DTMF，然后调用 dtmf.insertDTMF() 在与音频数据相同的方法上发送 DTMF。

我们对 insertDTMF() 的调用不仅指定了要发送的 DTMF（dialString），还指定了每个音调的持续时间（400 毫秒）以及音调之间的间隔时间（50 毫秒）。

#### 协商连接

当呼叫方 RTCPeerConnection 开始接收媒体（在麦克风的流添加到它之后），will 发生协商所需事件，通知呼叫方是时候开始与接收方协商连接了。如前所述，我们的示例简化了一些过程，因为我们同时控制呼叫方和接收方，因此 handleCallerNegotiationNeeded() 可以快速构建连接，将所需的调用链在一起，如以下所示。

```javascript
function handleCallerNegotiationNeeded() {
  log("正在协商…");
  callerPC
    .createOffer(offerOptions)
    .then((offer) => {
      log(`设置呼叫方的本地描述为：${offer.sdp}`);
      return callerPC.setLocalDescription(offer);
    })
    .then(() => {
      log("将接收方的远程描述设置为与呼叫方的本地描述相同");
      return receiverPC.setRemoteDescription(callerPC.localDescription);
    })
    .then(() => {
      log("正在创建应答");
      return receiverPC.createAnswer();
    })
    .then((answer) => {
      log(`将接收方的本地描述设置为 ${answer.sdp}`);
      return receiverPC.setLocalDescription(answer);
    })
    .then(() => {
      log("将呼叫方的远程描述设置为匹配");
      return callerPC.setRemoteDescription(receiverPC.localDescription);
    })
    .catch((err) => log(`协商期间出错：${err.message}`));
}
```

由于涉及协商连接的各种方法返回的是承诺，因此可以像这样将它们链接在一起：

1. 调用 callerPC.createOffer() 获取一个提议。
2. 然后，取该提议并通过调用 callerPC.setLocalDescription() 将呼叫方的本地描述设置为与提议匹配。
3. 然后，通过调用 receiverPC.setRemoteDescription() 将提议“传输”到接收方。这配置了接收方，使其了解呼叫方的配置。
4. 然后，接收方通过调用 receiverPC.createAnswer() 创建一个应答。
5. 然后，接收方通过调用 receiverPC.setLocalDescription() 将其本地描述设置为与新创建的应答匹配。
6. 然后，应答被“传输”到呼叫方，通过调用 callerPC.setRemoteDescription()。这使呼叫方了解接收方的配置。
7. 如果在任何时候发生错误，catch() 子句将错误消息输出到日志。

#### 跟踪其他状态变化

我们还可以通过接受 signalingstatechange 事件（用于信令状态变化）和 icegatheringstatechange 事件（用于 ICE 收集状态变化）来跟踪状态变化。我们没有使用这些来做任何事情，所以只是记录它们。我们本可以不设置这些事件监听器。

```javascript
function handleCallerSignalingStateChangeEvent() {
  log(`呼叫方的信令状态已更改为 ${callerPC.signalingState}`);
}

function handleCallerGatheringStateChangeEvent() {
  log(`呼叫方的 ICE 收集状态已更改为 ${callerPC.iceGatheringState}`);
}
```

#### 为接收方添加候选

当接收方的 RTCPeerConnection ICE 层提出新的候选时，它会向 receiverPC 发出 icecandidate 事件。icecandidate 事件处理程序的工作是将候选传输到呼叫方。在示例中，我们直接控制呼叫方和接收方，因此我们可以通过调用其 addIceCandidate() 方法直接将候选添加到呼叫方。这由 handleReceiverIceEvent() 处理。

```javascript
function handleReceiverIceEvent(event) {
  if (event.candidate) {
    log(`为呼叫方添加候选：${event.candidate.candidate}`);

    callerPC
      .addIceCandidate(new RTCIceCandidate(event.candidate))
      .catch((err) => log(`为呼叫方添加候选时出错：${err}`));
  } else {
    log("接收方没有更多候选。");
  }
}
```

如果 icecandidate 事件的 candidate 属性不是 null，我们将从 event.candidate 字符串中创建一个新的 RTCIceCandidate 对象，并通过将其传递给 callerPC.addIceCandidate() 来将其“传输”到呼叫方。如果 addIceCandidate() 失败，catch() 子句将错误输出到我们的日志框。

如果 event.candidate 是 null，这表明没有更多候选可用，我们将此信息记录下来。

#### 为接收方添加媒体

当接收方开始接收媒体时，将向接收方的 RTCPeerConnection 发送事件，receiverPC。如开始连接过程部分所述，当前 WebRTC 规范使用 track 事件为此目的。由于某些浏览器尚未更新以支持此功能，因此还需要处理 addstream 事件。这在下面的 handleReceiverTrackEvent() 和 handleReceiverAddStreamEvent() 方法中演示。

```javascript
function handleReceiverTrackEvent(event) {
  audio.srcObject = event.streams[0];
}

function handleReceiverAddStreamEvent(event) {
  audio.srcObject = event.stream;
}
```

track 事件包括一个 streams 属性，其中包含该轨道所属的流数组（一个轨道可以属于多个流）。我们取第一个流并将其附加到 <audio> 元素。

addstream 事件包括一个 stream 属性，指定添加到轨道的单个流。我们将其附加到 <audio> 元素。

#### 日志记录

在整个代码中使用了一个简单的 log() 函数，用于将文本追加到 `<div>` 框中，以向用户显示状态和错误信息。

```javascript
function log(msg) {
  logElement.innerText += `${msg}\n`;
}
```

### 结果

您可以在此处尝试这个示例。当您点击“拨号”按钮时，您应该会看到一系列日志消息输出；然后拨号将开始。如果您的浏览器作为用户体验的一部分播放音调，您应该会在传输时听到它们。

一旦音调传输完成，连接将关闭。您可以再次点击“拨号”以重新连接并发送音调。


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

