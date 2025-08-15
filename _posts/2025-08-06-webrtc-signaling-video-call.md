---
title: 探索 WebRTC 开发（6）：WebRTC 信令与视频通话
description: 本文将介绍 WebRTC 如何建立双向视频通话。
author: Keyframe
date: 2025-08-06 18:08:08 +0800
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

WebRTC 允许在两个设备之间进行实时的点对点媒体交换。通过一个称为**信令**的发现和协商过程建立连接。本教程将指导你建立双向视频通话。

WebRTC 是一种完全点对点的技术，用于实时交换音频、视频和数据，但有一个核心限制。如其他地方所述，必须进行某种形式的发现和媒体格式协商，以便不同网络上的两个设备能够找到彼此。这个过程称为**信令**，涉及双方连接到一个约定的第三方服务器。通过这个第三方服务器，两个设备可以找到彼此并交换协商消息。

在本文中，我们将进一步增强我们之前创建的 WebSocket 聊天（本文链接即将推出；它实际上还没有上线），以支持用户之间的双向视频通话。你可以在 Glitch 上尝试这个示例，并可以混合示例进行实验。你还可以在 GitHub 上查看完整项目。

**注意：**
如果你在 Glitch 上尝试示例，请注意对代码的任何更改将立即重置任何连接。此外，有一个短暂的超时期间；Glitch 实例仅用于快速实验和测试。

## 信令服务器

在两个设备之间建立 WebRTC 连接需要使用**信令服务器**来解决如何通过互联网连接它们。信令服务器的作用是作为中间人，让两个对等方找到并建立连接，同时尽量减少暴露潜在的私人信息。我们如何创建这个服务器，信令过程实际上是如何工作的？

首先，我们需要信令服务器本身。WebRTC 不指定信令信息的传输机制。你可以使用任何你喜欢的方式，从 WebSocket 到 `fetch()`，甚至使用信鸽来交换信令信息。

需要注意的是，服务器不需要理解或解释信令数据内容。即使它是 SDP，这也不太重要：通过信令服务器传输的消息内容实际上是一个黑盒。重要的是，当 ICE 子系统指示你将信令数据发送给另一个对等方时，你这样做了，另一个对等方知道如何接收这些信息并将其传递给自己的 ICE 子系统。你所要做的就是将信息来回传递。信令服务器的内容完全无关紧要。

### 为信令准备聊天服务器

我们的聊天服务器使用 WebSocket API 在每个客户端和服务器之间发送信息作为 JSON 字符串。服务器支持几种消息类型来处理任务，例如注册新用户、设置用户名和发送公共聊天消息。

为了允许服务器支持信令和 ICE 协商，我们需要更新代码。我们需要允许将消息发送到一个特定用户而不是广播到所有连接用户，并确保未识别的消息类型被传递并交付，而无需服务器知道它们是什么。这让我们可以使用同一个服务器发送信令消息，而无需单独的服务器。

让我们看看需要对聊天服务器进行的更改。这在文件 `chatserver.js` 中。

首先，添加了函数 `sendToOneUser()`。正如其名称所暗示的，这个函数将一个字符串化的 JSON 消息发送给一个特定用户名的用户。

```javascript
function sendToOneUser(target, msgString) {
  connectionArray.find((conn) => conn.username === target).send(msgString);
}
```

这个函数遍历连接用户列表，直到找到与指定用户名匹配的用户，然后将消息发送给该用户。参数 `msgString` 是一个字符串化的 JSON 对象。在这个示例中，这样更高效。由于消息已经字符串化，我们可以直接发送，无需进一步处理。`connectionArray` 中的每个条目都是一个 `WebSocket` 对象，因此我们可以直接调用其 `send()` 方法。

我们最初的聊天演示不支持向特定用户发送消息。下一个任务是更新主 WebSocket 消息处理程序以支持这样做。这涉及在 `"connection"` 消息处理程序的末尾进行更改：

```javascript
if (sendToClients) {
  const msgString = JSON.stringify(msg);

  if (msg.target && msg.target.length !== 0) {
    sendToOneUser(msg.target, msgString);
  } else {
    for (const connection of connectionArray) {
      connection.send(msgString);
    }
  }
}
```

此代码现在查看待处理消息是否具有 `target` 属性。如果存在该属性，它指定要发送消息的用户名，我们调用 `sendToOneUser()` 将消息发送给他们。否则，消息通过遍历连接列表广播给所有用户。

由于现有代码允许发送任意消息类型，因此无需额外更改。我们的客户端现在可以向任何特定用户发送未知类型的消息，让他们按需发送信令消息。

这就是我们在服务器端需要更改的内容。现在让我们考虑我们将实现的信令协议。

### 设计信令协议

现在我们已经建立了一个消息交换机制，我们需要一个协议来定义这些消息的外观。这可以通过多种方式完成；这里展示的只是信令消息的一种可能结构。

此示例的服务器使用字符串化的 JSON 对象与客户端通信。这意味着我们的信令消息将是 JSON 格式，其内容指定消息的类型以及处理消息所需的其他信息。

#### 交换会话描述

当开始信令过程时，发起通话的用户创建一个**提议**。此提议包含一个会话描述，以 SDP 格式，并需要传递给接收用户，我们称之为**被叫方**。被叫方用一个**回答**消息响应提议，也包含一个 SDP 描述。我们的信令服务器将使用 WebSocket 传输提议消息，类型为 `"video-offer"`，以及回答消息，类型为 `"video-answer"`。这些消息具有以下字段：

`type`

消息类型；要么是 `"video-offer"`，要么是 `"video-answer"`。

`name`

发送者的用户名。

`target`

接收描述的用户名（如果发送者是发起方，则指定被叫方，反之亦然）。

`sdp`

SDP（会话描述协议）字符串，描述从发送者角度看连接的本地端（或从接收者角度看的远程端）。

此时，两个参与者知道此次通话将使用哪些编解码器和编解码器参数。他们仍然不知道如何传输媒体数据本身。这就是互动式连接建立（ICE）的作用。

### 交换 ICE 候选

两个对等方需要交换 ICE 候选以协商它们之间的实际连接。每个 ICE 候选描述了发送方可以使用的通信方法。每个对等方在发现候选时按顺序发送，并在耗尽建议之前一直发送候选，即使媒体已经启动。

当每个对等方的 ICE 层开始发送候选时，它会进入一个交换过程，如下所示：

![ICE 候选交换](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling/webrtc_-_ice_candidate_exchange.svg)

每一方在从本地 ICE 层接收到候选时将它们发送给另一方；没有轮流或批量处理候选。一旦两个对等方同意一个双方都能使用的候选，媒体开始通过该候选的 SDP 流式传输。如果他们后来同意一个更好的（通常是更高性能）候选，流可能会根据需要更改格式。

尽管目前不支持，但在媒体已经开始流式传输后收到的候选理论上也可以用于在需要时降级到更低带宽的连接。

每个 ICE 候选通过信令服务器发送类型为 `"new-ice-candidate"` 的 JSON 消息传递给另一方。每个候选消息包含以下字段：

`type`

消息类型：`"new-ice-candidate"`。

`target`

正在进行协商的用户的用户名；服务器将消息仅定向给此用户。

`candidate`

SDP 候选字符串，描述提议的连接方法。通常你不需要查看此字符串的内容。你的代码需要做的就是通过信令服务器将它传递给远程对等方。

每个 ICE 消息建议一个通信协议（TCP 或 UDP）、IP 地址、端口号、连接类型（例如，指定的 IP 是否是对等方本身或中继服务器），以及其他将两台计算机连接在一起所需的信息。这包括 NAT 或其他网络复杂性。

**注意：**
需要注意的是，在 ICE 协商过程中，你的代码唯一需要做的就是在 `onicecandidate` 处理程序执行时，从 ICE 层接受传出候选并将它们通过信令连接发送给另一方，以及在收到信令服务器的 ICE 候选消息（当收到 `"new-ice-candidate"` 消息时）将它们传递给 ICE 层，通过调用 `RTCPeerConnection.addIceCandidate()`。仅此而已。

在几乎所有情况下，SDP 的内容与你无关。在你真正知道你在做什么之前，避免让事情变得比这更复杂。这样做只会带来疯狂。

你的信令服务器现在需要做的就是发送它被要求发送的消息。你的工作流程可能还需要登录/身份验证功能，但这些细节会有所不同。

**注意：**
`onicecandidate` 事件和 `createAnswer()` 承诺都是异步调用，需要分别处理。确保你的信令不会改变顺序！例如，在用 `setRemoteDescription()` 设置回答后，必须调用 `addIceCandidate()` 与服务器的 ICE 候选。

### 信令事务流程

信令过程涉及两个对等方通过中间人（信令服务器）交换消息。确切的过程当然会有所不同，但通常有一些关键点，信令消息在这些点上被处理：

- 每个用户在 Web 浏览器中运行的客户端
- 每个用户的 Web 浏览器
- 信令服务器
- 承载聊天服务的 Web 服务器

假设 Naomi 和 Priya 正在使用聊天软件进行讨论，Naomi 决定在他们之间打开一个视频通话。预期的事件序列如下：

![信令流程图](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling/webrtc_-_signaling_diagram.svg)

我们将在本文的过程中更详细地看到这一点。

### ICE 候选交换过程

当每个对等方的 ICE 层开始发送候选时，它会进入一个交换过程，如下所示：

![ICE 候选交换](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling/webrtc_-_ice_candidate_exchange.svg)

每一方在从本地 ICE 层接收到候选时立即将它们发送给另一方；没有轮流或批量处理候选。一旦两个对等方同意一个双方都能使用的候选，媒体开始流式传输。每个对等方在耗尽选项之前会继续发送候选，即使媒体已经开始流式传输。这是为了希望找到比最初选择的更好的选项。

如果条件发生变化（例如，网络连接恶化），一个或两个对等方可能会建议切换到更低带宽的媒体分辨率，或切换到其他编解码器。这会触发新的候选交换，之后可能会发生媒体格式和/或编解码器更改。在 [WebRTC 使用的编解码器] 指南中，你可以了解更多关于 WebRTC 要求浏览器支持的编解码器，哪些额外编解码器得到哪些浏览器的支持，以及如何选择要使用的最佳编解码器。

可选地，如果你希望更深入地了解此过程在 ICE 层中是如何完成的，可以查看 [RFC 8445：互动式连接建立]，第 2.3 节（“协商候选对并完成 ICE”）。你应该注意到，候选在交换，媒体在 ICE 层满意时开始流式传输。这都在幕后处理。我们的角色是通过信令服务器将候选来回传递。

## 客户端应用程序

任何信令过程的核心都是其消息处理。使用 WebSocket 进行信令并不是必需的，但它是一个常见的解决方案。当然，你应该选择适合你应用程序的消息交换机制。

让我们更新聊天客户端以支持视频通话。

### 更新 HTML

我们的客户端的 HTML 需要一个位置来展示视频。这需要视频元素和一个挂断通话的按钮：

```html
<div class="flexChild" id="camera-container">
  <div class="camera-box">
    <video id="received_video" autoplay></video>
    <video id="local_video" autoplay muted></video>
    <button id="hangup-button" onclick="hangUpCall();" disabled>Hang Up</button>
  </div>
</div>
```

这里定义的页面结构使用 `<div>` 元素，通过启用 CSS 的使用，让我们完全控制页面布局。我们将跳过布局细节，但可以在 GitHub 上查看 CSS 以了解我们是如何处理它的。请注意两个 `<video>` 元素，一个用于自我视图，一个用于连接，以及 `<button>` 元素。

具有 `id` `received_video` 的 `<video>` 元素将展示从连接用户接收的视频。我们指定 `autoplay` 属性，确保一旦视频开始到达，它立即播放。这消除了在代码中显式处理播放的需要。`local_video` `<video>` 元素展示用户摄像头的预览；指定 `muted` 属性，因为我们不需要在这个预览面板中听到本地音频。

最后，`hangup-button` `<button>` 元素定义为在没有连接通话时默认禁用（设置为我们默认值），并应用 `hangUpCall()` 函数点击时。此函数的作用是关闭通话，并向另一方发送信令服务器通知，请求它也关闭。

### JavaScript 代码

我们将此代码分为功能区域，以便更容易描述它是如何工作的。此代码的主要部分在 `connect()` 函数中：它在端口 6503 上打开一个 WebSocket 服务器，并建立一个处理程序以接收 JSON 对象格式的消息。此代码通常像以前一样处理文本聊天消息。

#### 向信令服务器发送消息

在代码中，我们调用 `sendToServer()` 以向信令服务器发送消息。此函数使用 WebSocket 连接完成其工作：

```javascript
function sendToServer(msg) {
  const msgJSON = JSON.stringify(msg);

  connection.send(msgJSON);
}
```

传递给此函数的消息对象通过调用 `JSON.stringify()` 转换为 JSON 字符串，然后我们调用 WebSocket 连接的 `send()` 函数将消息传输到服务器。

#### 启动通话的用户界面

处理 `"user-list"` 消息的代码调用 `handleUserListMsg()`。在这里，我们设置每个连接用户的处理程序，显示在聊天面板左侧的用户列表中。此函数接收一个消息对象，其 `users` 属性是一个字符串数组，指定每个连接用户的用户名。

```javascript
function handleUserListMsg(msg) {
  const listElem = document.querySelector(".user-list-box");

  while (listElem.firstChild) {
    listElem.removeChild(listElem.firstChild);
  }

  msg.users.forEach((username) => {
    const item = document.createElement("li");
    item.appendChild(document.createTextNode(username));
    item.addEventListener("click", invite, false);

    listElem.appendChild(item);
  });
}
```

在将 `<ul>` 元素的引用（包含用户名列表）获取到变量 `listElem` 后，我们通过删除其每个子元素来清空列表。

**注意：**
显然，通过添加和删除单个用户来更新列表会更高效，但在这种情况下，每次列表变化时重建整个列表就足够了。

然后我们使用 `forEach()` 遍历用户名数组。对于每个名称，我们创建一个新的 `<li>` 元素，然后使用 `createTextNode()` 创建一个包含用户名的新文本节点。该文本节点被添加为 `<li>` 元素的子元素。接下来，我们为列表项的 `click` 事件设置处理程序，点击用户名时调用我们的 `invite()` 方法，我们将在下一节中看到。

最后，我们将新项目添加到包含所有用户名的 `<ul>` 中。

#### 启动通话

当用户点击他们想要通话的用户名时，调用 `invite()` 函数作为该 `click` 事件的处理程序：

```javascript
const mediaConstraints = {
  audio: true, // 我们想要音频轨
  video: true, // 我们想要视频轨
};

function invite(evt) {
  if (myPeerConnection) {
    alert("你无法开始通话，因为你已经有一个通话在进行！");
  } else {
    const clickedUsername = evt.target.textContent;

    if (clickedUsername === myUsername) {
      alert(
        "恐怕我不能让你和自己通话。这会很奇怪。",
      );
      return;
    }

    targetUsername = clickedUsername;
    createPeerConnection();

    navigator.mediaDevices
      .getUserMedia(mediaConstraints)
      .then((localStream) => {
        document.getElementById("local_video").srcObject = localStream;
        localStream
          .getTracks()
          .forEach((track) => myPeerConnection.addTrack(track, localStream));
      })
      .catch(handleGetUserMediaError);
  }
}
```

这首先进行一个基本的合理性检查：用户是否已经连接？如果已经有一个 `RTCPeerConnection`，他们显然不能发起通话。然后从事件目标的 `textContent` 属性中获取被点击的用户名，并确保尝试发起通话的用户不是他们自己。

然后我们将被叫用户的名称复制到变量 `targetUsername` 中，并调用 `createPeerConnection()`，该函数将创建和基本配置 `RTCPeerConnection`。

一旦 `RTCPeerConnection` 被创建，我们通过调用 `MediaDevices.getUserMedia()` 请求访问用户的摄像头和麦克风。当成功时，返回的承诺的 `then` 处理程序将执行。它接收一个 `MediaStream` 对象，该对象表示来自用户麦克风的音频和来自网络摄像头的视频的流。

**注意：**
我们可以通过调用 `navigator.mediaDevices.enumerateDevices()` 获取设备列表，根据我们的标准过滤结果列表，然后在 `getUserMedia()` 的 `mediaConstraints` 对象的 `deviceId` 字段中使用选定设备的 `deviceId` 值，来限制允许的媒体输入设备。在实际中，`getUserMedia()` 会为你处理大部分工作。

我们将传入的流附加到本地预览 `<video>` 元素，通过设置其 `srcObject` 属性。由于该元素配置为自动播放传入视频，流在我们的本地预览框中开始播放。

然后我们遍历流中的轨道，通过 `addTrack()` 将每个轨道添加到 `RTCPeerConnection`。即使连接尚未完全建立，你也可以在适当的时候开始发送数据。在 ICE 协商完成之前接收到的媒体可能会帮助 ICE 决定采取最佳的连接方式，从而协助协商过程。

对于原生应用（如手机应用），在连接在两端都被接受之前，至少应避免发送，以避免在用户准备好了之前意外发送视频和/或音频数据。

一旦媒体被附加到 `RTCPeerConnection`，就会在连接上触发 `negotiationneeded` 事件，以便开始 ICE 协商。

如果在尝试获取本地媒体流时发生错误，我们的捕获子句将调用 `handleGetUserMediaError()`，根据需要向用户显示适当的错误。

#### 处理 `getUserMedia()` 错误

如果 `getUserMedia()` 返回的承诺以失败告终，我们的 `handleGetUserMediaError()` 函数执行。

```javascript
function handleGetUserMediaError(e) {
  switch (e.name) {
    case "NotFoundError":
      alert(
        "无法打开通话，因为未找到摄像头和/或麦克风。",
      );
      break;
    case "SecurityError":
    case "PermissionDeniedError":
      // 什么都不做；这与用户取消通话相同。
      break;
    default:
      alert(`打开摄像头和/或麦克风时出错：${e.message}`);
      break;
  }

  closeVideoCall();
}
```

在所有情况下都会显示错误消息，但有一个例外。在这个示例中，我们忽略 `"SecurityError"` 和 `"PermissionDeniedError"` 结果，将拒绝授予媒体硬件权限视为用户取消通话。

无论尝试获取流失败的原因是什么，我们都会调用 `closeVideoCall()` 函数来关闭 `RTCPeerConnection`，并释放尝试通话过程中分配的任何资源。此代码旨在安全地处理部分启动的通话。

#### 创建对等连接

`createPeerConnection()` 函数被呼叫方和被叫方用来构造它们的 `RTCPeerConnection` 对象，它们分别是 WebRTC 连接的两端。它由 `invite()` 调用，当呼叫方尝试发起通话时，以及由 `handleVideoOfferMsg()` 调用，当被叫方收到呼叫方的提议消息时。

```javascript
function createPeerConnection() {
  myPeerConnection = new RTCPeerConnection({
    iceServers: [
      // 冰服务器信息 - 使用你自己的！
      {
        urls: "stun:stun.stunprotocol.org",
      },
    ],
  });

  myPeerConnection.onicecandidate = handleICECandidateEvent;
  myPeerConnection.ontrack = handleTrackEvent;
  myPeerConnection.onnegotiationneeded = handleNegotiationNeededEvent;
  myPeerConnection.onremovetrack = handleRemoveTrackEvent;
  myPeerConnection.oniceconnectionstatechange =
    handleICEConnectionStateChangeEvent;
  myPeerConnection.onicegatheringstatechange =
    handleICEGatheringStateChangeEvent;
  myPeerConnection.onsignalingstatechange = handleSignalingStateChangeEvent;
}
```

使用 `RTCPeerConnection()` 构造函数时，我们将指定一个提供连接配置参数的对象。在这个示例中，我们只使用其中一个：`iceServers`。这是一个描述 STUN 和/或 TURN 服务器的对象数组，供 ICE 层在尝试建立呼叫方和被叫方之间的路由时使用。这些服务器用于确定在防火墙或 NAT 后设备之间通信时的最佳路由和协议。

**注意：**
你应该始终使用你自己的 STUN/TURN 服务器，或你有特定授权使用的服务器。此示例使用一个已知的公共 STUN 服务器，但滥用这些是不礼貌的。

`iceServers` 中的每个对象至少包含一个 `urls` 字段，提供可以到达指定服务器的 URL。它还可能提供 `username` 和 `credential` 值，以在需要时进行身份验证。

在创建 `RTCPeerConnection` 后，我们设置了我们关心的事件的处理程序。

前三个事件处理程序是必需的；你必须处理它们才能使用 WebRTC 进行流媒体。其余的不是严格必需的，但可能有用，我们将会看到。在这个示例中还有其他一些事件没有使用。以下是我们将实现的每个事件处理程序的总结：

`onicecandidate`

当本地 ICE 层需要你通过信令服务器向另一方发送 ICE 候选时，它会调用你的 `icecandidate` 事件处理程序。有关更多信息和此示例的代码，请参阅[发送 ICE 候选](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#sending-ice-candidates)。

`ontrack`

此 `track` 事件的处理程序由本地 WebRTC 层在向连接中添加轨道时调用。这可以让你将传入媒体连接到元素以进行显示，例如。有关详细信息，请参阅[接收新流](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#receiving-new-streams)。

`onnegotiationneeded`

每当 WebRTC 基础设施需要你重新开始会话协商过程时，都会调用此函数。它的作用是创建并发送提议，要求另一方与我们连接。有关详细信息，请参阅[开始协商](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#starting-negotiation)。

`onremovetrack`

这是 `ontrack` 的对应方，用于处理 `removetrack` 事件；当远程对等方从发送的媒体中移除轨道时，会将该事件发送到 `RTCPeerConnection`。有关详细信息，请参阅[处理轨道移除](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#handling-the-removal-of-tracks)。

`oniceconnectionstatechange`

ICE 层发送 `iceconnectionstatechange` 事件，以通知你 ICE 连接状态的变化。这可以帮助你知道连接何时失败或丢失。我们将在 [ICE 连接状态](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#ice-connection-state) 中看到此示例的代码。

`onicegatheringstatechange`

当 ICE 代理收集候选的过程从一个状态转移到另一个状态（例如，开始收集候选或完成协商）时，ICE 层会向你发送 `icegatheringstatechange` 事件。有关详细信息，请参阅[ICE 收集状态](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#ice-gathering-state)。

`onsignalingstatechange`

当信令过程的状态发生变化（或信令服务器的连接发生变化）时，WebRTC 基础设施会发送 `signalingstatechange` 消息。有关详细信息，请参阅[信令状态](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#signaling-state)。

#### 开始协商

一旦呼叫方创建了其 `RTCPeerConnection`，创建了媒体流，并如[开始通话](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#starting-a-call)中所示将轨道添加到连接中，浏览器将向 `RTCPeerConnection` 触发 `negotiationneeded` 事件，以指示它已准备好开始与另一方协商。以下是我们的处理 `negotiationneeded` 事件的代码：

```javascript
function handleNegotiationNeededEvent() {
  myPeerConnection
    .createOffer()
    .then((offer) => myPeerConnection.setLocalDescription(offer))
    .then(() => {
      sendToServer({
        name: myUsername,
        target: targetUsername,
        type: "video-offer",
        sdp: myPeerConnection.localDescription,
      });
    })
    .catch(window.reportError);
}
```

为了开始协商过程，我们需要创建并向我们想要连接的对等方发送 SDP 提议。此提议包括我们本地添加到连接的媒体流的受支持配置列表（即我们想要发送到通话另一端的视频），以及 ICE 层已经收集的任何 ICE 候选。我们通过调用 `myPeerConnection.createOffer()` 创建此提议。

当 `createOffer()` 成功（兑现承诺）时，我们将创建的提议信息传递给 `myPeerConnection.setLocalDescription()`，这为呼叫方端的连接配置本地描述和媒体配置状态。

**注意：**
从技术上讲，`createOffer()` 返回的字符串是一个 RFC 3264 提议。

当 `setLocalDescription()` 返回的承诺兑现时，我们知道描述是有效的，并且已经设置。此时，我们通过创建一个包含本地描述（现在与提议相同）的 `"video-offer"` 消息，然后通过信令服务器将其发送给被叫方，来发送我们的提议。提议具有以下成员：

`type`

消息类型：`"video-offer"`。

`name`

呼叫方的用户名。

`target`

我们想要通话的用户名称。

`sdp`

描述提议的 SDP 字符串。

如果在初始 `createOffer()` 或其后的任何兑现处理程序中发生错误，将通过调用我们的 `window.reportError()` 函数报告错误。

一旦 `setLocalDescription()` 的兑现处理程序运行，ICE 代理开始向 `RTCPeerConnection` 发送 `icecandidate` 事件，每个事件都是一个需要传输给远程对等方的候选。我们的 `icecandidate` 事件处理程序负责将候选传输给另一方。

#### 会话协商

现在我们已经开始与另一方协商并发送了提议，让我们看看被叫方在连接的另一侧是如何处理的。被叫方接收提议并调用 `handleVideoOfferMsg()` 函数来处理它。让我们看看被叫方是如何处理 `"video-offer"` 消息的。

##### 处理邀请

当提议到达时，被叫方的 `handleVideoOfferMsg()` 函数被调用，接收收到的 `"video-offer"` 消息。此函数需要做两件事。首先，它需要创建自己的 `RTCPeerConnection` 并将包含其麦克风和网络摄像头音频和视频的轨道添加到该连接中。其次，它需要处理收到的提议，构建并发送其回答。

```javascript
function handleVideoOfferMsg(msg) {
  let localStream = null;

  targetUsername = msg.name;
  createPeerConnection();

  const desc = new RTCSessionDescription(msg.sdp);

  myPeerConnection
    .setRemoteDescription(desc)
    .then(() => navigator.mediaDevices.getUserMedia(mediaConstraints))
    .then((stream) => {
      localStream = stream;
      document.getElementById("local_video").srcObject = localStream;

      localStream
        .getTracks()
        .forEach((track) => myPeerConnection.addTrack(track, localStream));
    })
    .then(() => myPeerConnection.createAnswer())
    .then((answer) => myPeerConnection.setLocalDescription(answer))
    .then(() => {
      const msg = {
        name: myUsername,
        target: targetUsername,
        type: "video-answer",
        sdp: myPeerConnection.localDescription,
      };

      sendToServer(msg);
    })
    .catch(handleGetUserMediaError);
}
```

此代码非常类似于我们在[开始通话](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#starting-a-call)中的 `invite()` 函数中所做的。它首先通过我们的 `createPeerConnection()` 函数创建和配置 `RTCPeerConnection`。然后它从收到的 `"video-offer"` 消息中获取 SDP 提议，并使用它创建一个新的 `RTCSessionDescription` 对象，表示呼叫方的会话描述。

然后将此会话描述传递给 `myPeerConnection.setRemoteDescription()`。这建立了收到的提议作为连接的远程（呼叫方）端的描述。如果成功，兑现处理程序（在 `then()` 子句中）使用 `getUserMedia()` 获取对被叫方的摄像头和麦克风的访问权限，将轨道添加到连接中，依此类推，正如我们在 `invite()` 中之前所做的。

一旦使用 `myPeerConnection.createAnswer()` 创建了回答，通过调用 `myPeerConnection.setLocalDescription()` 设置本地连接描述为回答的 SDP，然后将回答通过信令服务器发送给呼叫方，以告知他们回答是什么。

任何错误都将传递给 `handleGetUserMediaError()`，在[处理 `getUserMedia()` 错误](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#handling-getusermedia-errors)中描述。

**注意：**
与呼叫方一样，一旦 `setLocalDescription()` 的兑现处理程序运行，浏览器开始触发 `icecandidate` 事件，这些事件需要被处理并传输给远程对等方。

最后，呼叫方通过处理收到的回答消息，创建一个新的 `RTCSessionDescription` 对象，表示被叫方的会话描述，并将其传递给 `myPeerConnection.setRemoteDescription()`。

```javascript
function handleVideoAnswerMsg(msg) {
  const desc = new RTCSessionDescription(msg.sdp);
  myPeerConnection.setRemoteDescription(desc).catch(window.reportError);
}
```

##### 发送 ICE 候选

ICE 协商过程涉及每个对等方向另一方发送候选，直到它耗尽了支持 `RTCPeerConnection` 媒体传输需求的潜在配置。由于 ICE 不了解你的信令服务器，你的代码在 `icecandidate` 事件处理程序中处理每个候选的传输。

你的 `onicecandidate` 处理程序接收到一个事件，其 `candidate` 属性是描述候选的 SDP（或为 `null` 以指示 ICE 层已经耗尽了建议的配置）。需要传输的内容在 `candidate` 中。以下是我们的示例实现：

```javascript
function handleICECandidateEvent(event) {
  if (event.candidate) {
    sendToServer({
      type: "new-ice-candidate",
      target: targetUsername,
      candidate: event.candidate,
    });
  }
}
```

这通过 `sendToServer()` 函数（在[向信令服务器发送消息](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#sending-messages-to-the-signaling-server)中描述）构建一个包含候选的对象，并将其发送给另一方。消息的属性是：

`type`

消息类型：`"new-ice-candidate"`。

`target`

需要投递 ICE 候选的用户名。这允许信令服务器路由消息。

`candidate`

代表 ICE 层想要传输给另一方的候选的 SDP。

此消息的格式（以及你在处理信令时所做的一切）完全由你决定，取决于你的需求；你可以根据需要提供其他信息。

**注意：**
需要注意的是，`icecandidate` 事件是在 ICE 候选从通话的另一端到达时触发的。相反，它们是由你自己的通话端触发的，以便你可以承担通过你选择的通道传输数据的任务。这在你刚接触 WebRTC 时可能会令人困惑。

##### 接收 ICE 候选

信令服务器使用它选择的任何方法将 ICE 候选传递给目标对等方；在我们的示例中，这是作为 JSON 对象，`type` 属性包含字符串 `"new-ice-candidate"`。我们的 `handleNewICECandidateMsg()` 函数由主 WebSocket 入站消息代码调用以处理这些消息：

```javascript
function handleNewICECandidateMsg(msg) {
  const candidate = new RTCIceCandidate(msg.candidate);

  myPeerConnection.addIceCandidate(candidate).catch(window.reportError);
}
```

此函数通过将收到的 SDP 传递给其构造函数来构造 `RTCIceCandidate` 对象，然后通过将其传递给 `myPeerConnection.addIceCandidate()` 将候选传递给 ICE 层。这将新的 ICE 候选传递给本地 ICE 层，最后，我们在此过程中处理此候选的角色就完成了。

每个对等方向另一方发送一个可能的传输配置候选，它认为可能适用于交换的媒体。在某个时刻，两个对等方同意某个候选是一个好的选择，他们打开连接并开始共享媒体。需要注意的是，ICE 协商在媒体流式传输后不会停止。相反，候选可能在对话开始后继续交换，要么在尝试找到更好的连接方法，要么因为他们在成功建立连接时已经在传输中。

此外，如果流式传输场景中发生某些变化，协商将再次开始，`negotiationneeded` 事件将被发送到 `RTCPeerConnection`，整个过程将如前所述重新开始。这可能发生在以下情况：

- 网络状态变化，例如带宽变化、从 Wi-Fi 切换到蜂窝连接等。
- 在手机上切换前后摄像头。
- 流的配置发生变化，例如其分辨率或帧率。

##### 接收新流

当新轨道被添加到 `RTCPeerConnection` 时——无论是通过调用其 `addTrack()` 方法还是由于流格式的重新协商——将为连接中添加的每个轨道设置 `track` 事件到 `RTCPeerConnection`。使用新添加的媒体需要实现 `track` 事件的处理程序。一个常见的需求是将传入媒体附加到适当的 HTML 元素。在我们的示例中，我们将轨道的流添加到显示传入视频的 `<video>` 元素：

```javascript
function handleTrackEvent(event) {
  document.getElementById("received_video").srcObject = event.streams[0];
  document.getElementById("hangup-button").disabled = false;
}
```

传入流被附加到 `"received_video"` `<video>` 元素，"挂断" `<button>` 元素被启用，以便用户可以挂断通话。

一旦此代码完成，终于在本地浏览器窗口中显示了另一方发送的视频！

##### 处理轨道移除

当远程对等方通过调用 `RTCPeerConnection.removeTrack()` 从连接中移除轨道时，你的代码将收到 `removetrack` 事件。我们对 `"removetrack"` 的处理程序是：

```javascript
function handleRemoveTrackEvent(event) {
  const stream = document.getElementById("received_video").srcObject;
  const trackList = stream.getTracks();

  if (trackList.length === 0) {
    closeVideoCall();
  }
}
```

此代码从 `"received_video"` `<video>` 元素的 `srcObject` 属性中获取传入视频 `MediaStream`，然后调用流的 `getTracks()` 方法以获取流的轨道数组。

如果数组的长度为零，意味着流中没有轨道，我们通过调用 `closeVideoCall()` 结束通话。这会干净地将我们的应用恢复到可以再次开始或接收通话的状态。有关 `closeVideoCall()` 的工作原理，请参阅[结束通话](https://gjzkeyframe.github.io/posts/webrtc-signaling-video-call/#ending-the-call)。

#### 结束通话

通话结束的原因有很多。通话可能已经完成，双方都已经挂断。或者发生了网络故障，或者一个用户退出了浏览器，或者系统崩溃。无论如何，所有美好的事物都必须结束。

##### 挂断

当用户点击"挂断"按钮结束通话时，调用`hangUpCall()`函数：

```javascript
function hangUpCall() {
  closeVideoCall();
  sendToServer({
    name: myUsername,
    target: targetUsername,
    type: "hang-up",
  });
}
```

`hangUpCall()`执行`closeVideoCall()`以关闭并重置连接，释放资源。然后它构建一个`"hang-up"`消息并发送到通话的另一端，通知另一方整齐地关闭自己。

##### 结束通话

`closeVideoCall()`函数负责停止流，清理，并释放`RTCPeerConnection`对象：

```javascript
function closeVideoCall() {
  const remoteVideo = document.getElementById("received_video");
  const localVideo = document.getElementById("local_video");

  if (myPeerConnection) {
    myPeerConnection.ontrack = null;
    myPeerConnection.onremovetrack = null;
    myPeerConnection.onremovestream = null;
    myPeerConnection.onicecandidate = null;
    myPeerConnection.oniceconnectionstatechange = null;
    myPeerConnection.onsignalingstatechange = null;
    myPeerConnection.onicegatheringstatechange = null;
    myPeerConnection.onnegotiationneeded = null;

    if (remoteVideo.srcObject) {
      remoteVideo.srcObject.getTracks().forEach((track) => track.stop());
    }

    if (localVideo.srcObject) {
      localVideo.srcObject.getTracks().forEach((track) => track.stop());
    }

    myPeerConnection.close();
    myPeerConnection = null;
  }

  remoteVideo.removeAttribute("src");
  remoteVideo.removeAttribute("srcObject");
  localVideo.removeAttribute("src");
  localVideo.removeAttribute("srcObject");

  document.getElementById("hangup-button").disabled = true;
  targetUsername = null;
}
```

在获取对两个`<video>`元素的引用后，我们检查是否存在 WebRTC 连接；如果存在，我们继续断开并关闭通话：

1. 移除所有事件处理程序。这可以防止在连接关闭过程中触发杂散的事件处理程序，从而避免潜在的错误。
2. 对于远程和本地视频流，我们遍历每个轨道，调用`MediaStreamTrack.stop()`方法关闭每个轨道。
3. 通过调用`myPeerConnection.close()`关闭`RTCPeerConnection`。
4. 将`myPeerConnection`设置为`null`，确保我们的代码了解到没有正在进行的通话；这对于用户在用户列表中点击用户名时非常有用。

然后，对于传入和传出的`<video>`元素，我们使用它们的`removeAttribute()`方法移除它们的`src`和`srcObject`属性。这完成了将流与视频元素的分离。

最后，我们将"挂断"按钮的`disabled`属性设置为`true`，使其在没有通话进行时无法点击；然后我们将`targetUsername`设置为`null`，因为我们不再与任何人通话。这允许用户呼叫另一个用户，或接收来电。

#### 处理状态变化

有许多额外的事件，你可以设置监听器以通知你的代码各种状态变化。我们使用其中三个：`iceconnectionstatechange`、`icegatheringstatechange` 和 `signalingstatechange`。

##### ICE 连接状态

当 ICE 层检测到连接状态变化（例如通话从另一端终止）时，会向 `RTCPeerConnection` 发送 `iceconnectionstatechange` 事件。

```javascript
function handleICEConnectionStateChangeEvent(event) {
  switch (myPeerConnection.iceConnectionState) {
    case "closed":
    case "failed":
      closeVideoCall();
      break;
  }
}
```

在此处，当 ICE 连接状态更改为 `"closed"` 或 `"failed"` 时，我们执行 `closeVideoCall()` 函数，以关闭我们的连接端，使我们能够再次开始或接受通话。

**注意：**
我们不在此处监视 `disconnected` 信令状态，因为它可能指示临时问题，并在一段时间后恢复为 `connected` 状态。监视它会在任何临时网络问题时关闭视频通话。

##### ICE 信令状态

同样，我们监视 `signalingstatechange` 事件。如果信令状态更改为 `closed`，我们也关闭通话。

```javascript
function handleSignalingStateChangeEvent(event) {
  switch (myPeerConnection.signalingState) {
    case "closed":
      closeVideoCall();
      break;
  }
}
```

**注意：**
`closed` 信令状态已弃用，取而代之的是 `closed` ICE 连接状态。我们在此处监视它以增加一些向后兼容性。

##### ICE 收集状态

`icegatheringstatechange` 事件用于通知你 ICE 候选收集过程状态的变化。我们的示例没有使用此功能，但为了调试目的，以及检测候选收集何时完成，监视这些事件可能会很有用。

```javascript
function handleICEGatheringStateChangeEvent(event) {
  // 我们的示例在此处只是将信息记录到控制台，
  // 但你可以根据需要进行操作。
}
```

## 后续步骤

现在你可以在 Glitch 上尝试这个示例，看看它是如何工作的。在两个设备上打开 Web 控制台，查看记录的输出——尽管在上面显示的代码中你没有看到它，但服务器（以及 GitHub 上的代码）中有很多控制台输出，以便你可以看到信令和连接过程的工作。

另一个明显的改进是添加一个“响铃”功能，这样在请求使用摄像头和麦克风之前，会先出现一个提示，例如“用户 X 正在呼叫。你想接听吗？”

## 参考资料

![信令流程图](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling/webrtc_-_signaling_diagram.svg)![ICE 候选交换](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling/webrtc_-_ice_candidate_exchange.svg)



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

