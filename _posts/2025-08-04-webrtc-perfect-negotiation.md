---
title: 探索 WebRTC 开发（4）：WebRTC 完美协商模式
description: 本文将介绍 WebRTC 完美协商，解释其工作原理及为何是推荐的对等连接协商方式，并提供示例代码。
author: Keyframe
date: 2025-08-04 18:08:08 +0800
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

本文介绍 WebRTC **完美协商**，解释其工作原理及为何是推荐的对等连接协商方式，并提供示例代码。

由于 WebRTC 不强制信令传输机制，因此在协商时非常灵活。然而，尽管信令消息的传输和通信方式灵活，但仍建议遵循完美协商模式。

早期的 WebRTC 浏览器部署后，发现协商过程的部分环节对于典型用例来说过于复杂。这源于 API 的一些问题和需要预防的潜在竞态条件。这些问题已得到解决，让我们得以显著简化 WebRTC 协商。完美协商模式是 WebRTC 早期发展以来协商改进的一个例子。

## 完美协商概念

完美协商使得协商过程与应用的其余逻辑完全分离成为可能。协商本质上是不对称的操作：一方需要作为 “呼叫者”，另一方作为 “被呼叫者”。完美协商模式通过将这种差异分离到独立的协商逻辑中，使应用无需关心连接的哪一端。对于应用来说，无论是主动呼叫还是接收呼叫都没有区别。

完美协商的最大优点是，呼叫者和被呼叫者使用相同的代码，无需重复编写额外的协商代码。

完美协商通过为两个对等方分配角色来实现，这些角色与 WebRTC 连接状态完全分离：

- **礼貌的对等方**：使用 ICE 回滚来防止与传入提议的冲突。礼貌的对等方可能会发送提议，但如果收到另一个对等方的提议，会回应 “好吧，算了吧，撤回我的提议，我来考虑你的提议。”
- **不礼貌的对等方**：总是忽略与其自身提议冲突的传入提议。它从不道歉或向礼貌的对等方让步。每次发生冲突时，不礼貌的对等方获胜。

这样，两个对等方在发送的提议发生冲突时，确切地知道应该发生什么。对错误条件的响应变得更加可预测。

如何确定哪个对等方是礼貌的，哪个是不礼貌的，通常由你决定。可以将礼貌的角色分配给首先连接到信令服务器的对等方，或者更复杂的方式，比如让对等方交换随机数并分配给胜者。无论你如何确定，一旦分配了这些角色，两个对等方就可以协同管理信令，避免死锁，无需大量额外代码。

需要记住的重要一点是：在完美协商过程中，呼叫者和被呼叫者的角色可以切换。如果礼貌的对等方是呼叫者并发送提议，但与不礼貌的对等方发生冲突，礼貌的对等方会撤回其提议，并回复收到的提议。通过这样做，礼貌的对等方已从呼叫者切换为被呼叫者！

## 实现完美协商

让我们看看实现完美协商模式的示例。代码假设有一个定义好的 `SignalingChannel` 类，用于与信令服务器通信。当然，你的代码可以使用任何你喜欢的信令技术。

请注意，此代码对于连接中的两个对等方是相同的。

### 创建信令和对等连接

首先，需要打开信令通道并创建 `RTCPeerConnection`。这里列出的 STUN 服务器显然不是真实的；你需要将 `stun.my-server.tld` 替换为真实 STUN 服务器的地址。

```javascript
const config = {
  iceServers: [{ urls: "stun:stun.my-stun-server.tld" }],
};

const signaler = new SignalingChannel();
const pc = new RTCPeerConnection(config);
```

这段代码还使用类 "self-view" 和 "remote-view" 获取 `<video>` 元素；这些将分别包含本地用户的自我视图和来自远程对等方的传入流视图。

### 连接到远程对等方

```javascript
const constraints = { audio: true, video: true };
const selfVideo = document.querySelector("video.self-view");
const remoteVideo = document.querySelector("video.remote-view");

async function start() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia(constraints);

    for (const track of stream.getTracks()) {
      pc.addTrack(track, stream);
    }
    selfVideo.srcObject = stream;
  } catch (err) {
    console.error(err);
  }
}
```

上面的 `start()` 函数可以由想要通信的两个端点中的任何一个调用。谁先调用并不重要；协商会正常进行。

这与旧的 WebRTC 连接建立代码没有显著区别。通过调用 `getUserMedia()` 获取用户的摄像头和麦克风。然后将得到的媒体轨道通过 `addTrack()` 添加到 `RTCPeerConnection`。最后，将自我视图 `<video>` 元素的媒体源设置为摄像头和麦克风流，让本地用户看到其他对等方看到的内容。

### 处理传入轨道

接下来，我们需要设置一个 `track` 事件处理程序来处理通过协商接收的视频和音频轨道。为此，我们实现了 `RTCPeerConnection` 的 `ontrack` 事件处理程序。

```javascript
pc.ontrack = ({ track, streams }) => {
  track.onunmute = () => {
    if (remoteVideo.srcObject) {
      return;
    }
    remoteVideo.srcObject = streams[0];
  };
};
```

当 `track` 事件发生时，执行此处理程序。通过解构，提取 `RTCTrackEvent` 的 `track` 和 `streams` 属性。前者是接收到的视频轨道或音频轨道。后者是一个 `MediaStream` 对象数组，每个对象表示包含此轨道的流（在某些情况下，一个轨道可能同时属于多个流）。在我们的情况下，这将始终在索引 0 处包含一个流，因为我们之前将一个流传递给了 `addTrack()`。

我们为轨道添加了一个静音解除事件处理程序，因为轨道在开始接收数据包时会解除静音。我们将接收代码的其余部分放在这里。

如果已经从远程对等方接收视频（可以通过检查远程视图的 `<video>` 元素的 `srcObject` 属性是否有值来判断），我们什么也不做。否则，将 `srcObject` 设置为 `streams` 数组索引 0 处的流。

### 完美协商逻辑

现在我们来看真正的完美协商逻辑，它完全独立于应用的其余部分。

#### 处理协商需要事件

首先，我们实现 `RTCPeerConnection` 事件处理程序 `onnegotiationneeded`，以获取本地描述并通过信令通道发送给远程对等方。

```javascript
let makingOffer = false;

pc.onnegotiationneeded = async () => {
  try {
    makingOffer = true;
    await pc.setLocalDescription();
    signaler.send({ description: pc.localDescription });
  } catch (err) {
    console.error(err);
  } finally {
    makingOffer = false;
  }
};
```

请注意，`setLocalDescription()` 不带参数会根据当前 `signalingState` 自动创建并设置适当的描述。设置的描述是远程对等方最新提议的响应，或者是如果没有任何正在进行的协商，则创建一个新的提议。在这里，它将始终是一个 `offer`，因为 `negotiationneeded` 事件仅在 `stable` 状态下触发。

我们设置一个布尔变量 `makingOffer` 为 `true` 以标记我们正在准备提议。为了避免竞态条件，我们将使用此值而不是 `signalingState` 来确定是否正在处理提议，因为 `signalingState` 的值是异步更改的，会引入竞态条件。

一旦提议被创建、设置并发送（或发生错误），`makingOffer` 将被设置回 `false`。

#### 处理传入 ICE 候选

接下来，我们需要处理 `RTCPeerConnection` 事件 `icecandidate`，这是本地 ICE 层将候选传递给我们的方式，以便通过信令通道发送给远程对等方。

```javascript
pc.onicecandidate = ({ candidate }) => signaler.send({ candidate });
```

此代码获取此 ICE 事件的 `candidate` 成员，并将其传递给信令通道的 `send()` 方法，以通过信令服务器发送给远程对等方。

#### 处理信令通道上的传入消息

最后一部分是处理信令服务器发来的消息的代码。这里实现为信令通道对象的 `onmessage` 事件处理程序。每次从信令服务器收到消息时都会调用此方法。

```javascript
let ignoreOffer = false;

signaler.onmessage = async ({ data: { description, candidate } }) => {
  try {
    if (description) {
      const offerCollision =
        description.type === "offer" &&
        (makingOffer || pc.signalingState !== "stable");

      ignoreOffer = !polite && offerCollision;
      if (ignoreOffer) {
        return;
      }

      await pc.setRemoteDescription(description);
      if (description.type === "offer") {
        await pc.setLocalDescription();
        signaler.send({ description: pc.localDescription });
      }
    } else if (candidate) {
      try {
        await pc.addIceCandidate(candidate);
      } catch (err) {
        if (!ignoreOffer) {
          throw err;
        }
      }
    }
  } catch (err) {
    console.error(err);
  }
};
```

当通过 `SignalingChannel` 的 `onmessage` 事件处理程序收到传入消息时，收到的 JSON 对象被解构以获取其中的 `description` 或 `candidate`。如果传入消息有 `description`，则它是远程对等方发送的提议或响应。

另一方面，如果消息有 `candidate`，则它是远程对等方作为 trickle ICE 的一部分发送的 ICE 候选。候选的最终目的是通过将其传递给 `addIceCandidate()` 传递给本地 ICE 层。

##### 收到描述

如果收到 `description`，我们准备响应传入的提议或响应。首先，我们检查是否处于可以接受提议的状态。如果连接的信令状态不是 `stable`，或者我们的连接端已经开始发送自己的提议，则可能会发生提议冲突。

如果收到冲突的提议，我们返回，不设置描述，并将 `ignoreOffer` 设置为 `true`，以确保我们还忽略信令通道上属于此提议的任何候选。这样做可以避免错误，因为我们没有告知我们的端关于这个提议。

如果收到冲突的提议，我们不需要做任何特殊处理，因为我们的现有提议将在下一步自动回滚。

确保我们想要接受提议后，我们通过调用 `setRemoteDescription()` 将远程描述设置为传入的提议。这让 WebRTC 知道远程对等方的提议配置。如果我们是礼貌的对等方，我们将撤回我们的提议并接受新的提议。

如果新设置的远程描述是提议，我们通过调用 `setLocalDescription()` 无参数让 WebRTC 选择适当的本地配置。这将自动生成一个对收到提议的适当响应。然后我们通过信令通道将响应发送回第一个对等方。

##### 收到 ICE 候选

另一方面，如果收到的消息包含 ICE 候选，我们通过调用 `RTCPeerConnection` 方法 `addIceCandidate()` 将其传递给本地 ICE 层。如果发生错误且我们没有因为冲突的提议而忽略，则抛出错误，让调用者处理。否则，我们忽略错误，因为它在此上下文中不重要。

## 使协商完美

如果你好奇是什么让完美协商如此完美，这一节为你解答。这里，我们将查看 WebRTC API 的每个更改以及最佳实践建议，以实现完美协商。

### 无眩光的 setLocalDescription()

过去，`negotiationneeded` 事件的处理方式容易受到眩光影响，即两个对等方可能同时尝试发送提议，导致一个或两个对等方出错并中止连接尝试。

#### 旧方法

考虑以下 `onnegotiationneeded` 事件处理程序：

```javascript
pc.onnegotiationneeded = async () => {
  try {
    await pc.setLocalDescription(await pc.createOffer());
    signaler.send({ description: pc.localDescription });
  } catch (err) {
    console.error(err);
  }
};
```

由于 `createOffer()` 方法是异步的，需要一些时间完成，在此期间，远程对等方可能会尝试发送自己的提议，导致我们离开 `stable` 状态并进入 `have-remote-offer` 状态，这意味着我们在等待对提议的响应。但一旦它收到我们刚发送的提议，远程对等方也是如此。这使得两个对等方都处于无法完成连接尝试的状态。

#### 使用更新 API 的完美协商

如实现完美协商一节所示，我们可以通过引入一个变量（这里称为 `makingOffer`）来消除此问题，该变量用于指示我们正在发送提议，并利用更新的 `setLocalDescription()` 方法：

```javascript
let makingOffer = false;

pc.onnegotiationneeded = async () => {
  try {
    makingOffer = true;
    await pc.setLocalDescription();
    signaler.send({ description: pc.localDescription });
  } catch (err) {
    console.error(err);
  } finally {
    makingOffer = false;
  }
};
```

我们在调用 `setLocalDescription()` 之前立即设置 `makingOffer` 以防止干扰发送此提议，并且在提议被发送到信令服务器之前（或发生错误阻止提议发送），我们不会将其清除为 `false`。这样，我们避免了提议冲突的风险。

### setRemoteDescription() 中的自动回滚

完美协商的一个关键组件是礼貌对等方的概念，它在发送提议时收到提议会自动回滚。以前，触发回滚需要手动检查回滚条件并手动触发回滚，方法是将本地描述设置为类型为 `rollback` 的描述，如下所示：

```javascript
await pc.setLocalDescription({ type: "rollback" });
```

这样做会使本地对等方从之前的任何状态返回到 `stable` `signalingState`。由于对等方只有在 `stable` 状态下才能接受提议，因此对等方已撤销其提议，并准备好接收远程（不礼貌）对等方的提议。然而，这种方法存在一些问题。

#### 使用旧 API 的完美协商

使用之前的 API 实现完美协商中的传入协商消息处理如下所示：

```javascript
signaler.onmessage = async ({ data: { description, candidate } }) => {
  try {
    if (description) {
      if (description.type === "offer" && pc.signalingState !== "stable") {
        if (!polite) {
          return;
        }

        await Promise.all([
          pc.setLocalDescription({ type: "rollback" }),
          pc.setRemoteDescription(description),
        ]);
      } else {
        await pc.setRemoteDescription(description);
      }

      if (description.type === "offer") {
        await pc.setLocalDescription(await pc.createAnswer());
        signaler.send({ description: pc.localDescription });
      }
    } else if (candidate) {
      try {
        await pc.addIceCandidate(candidate);
      } catch (err) {
        if (!ignoreOffer) {
          throw err;
        }
      }
    }
  } catch (err) {
    console.error(err);
  }
};
```

由于回滚通过推迟更改直到下一次协商（这将在当前协商完成后立即开始）来工作，因此礼貌的对等方需要知道在发送的提议等待响应时是否需要丢弃收到的提议。此代码检查收到的消息是否为提议，并且如果收到提议时本地信令状态不是 `stable`，则会发生冲突。如果收到冲突的提议，并且本地对等方是礼貌的，则需要触发回滚。这需要在 `Promise.all()` 的上下文中执行两个步骤，以确保两个语句在处理收到的提议之前完全执行。第一个语句触发回滚，第二个将远程描述设置为收到的提议，从而完成用新收到的提议替换之前发送的提议的过程。礼貌的对等方现在已从呼叫者切换为被呼叫者。

所有其他从不礼貌对等方收到的描述都通过调用 `setRemoteDescription()` 正常处理。

最后，我们通过调用 `setLocalDescription()` 并将其设置为 `createAnswer()` 返回的描述来处理收到的提议。然后通过信令通道将其发送回礼貌的对等方。

如果收到的消息是 ICE 候选而不是 SDP 描述，则通过调用 `RTCPeerConnection` 方法 `addIceCandidate()` 将其传递给本地 ICE 层。如果在此处发生错误，并且我们没有因为冲突的提议而忽略，则抛出错误，让调用者处理。否则，我们忽略错误，因为它在此上下文中不重要。

#### 使用更新 API 的完美协商

更新的代码利用了现在可以无参数调用 `setLocalDescription()` 的事实，让它为你做正确的事情，以及 `setRemoteDescription()` 会在需要时自动回滚。这让我们可以去掉用于保持时间顺序的 `Promise`，因为回滚已成为 `setRemoteDescription()` 调用的一个基本原子部分。

```javascript
let ignoreOffer = false;

signaler.onmessage = async ({ data: { description, candidate } }) => {
  try {
    if (description) {
      const offerCollision =
        description.type === "offer" &&
        (makingOffer || pc.signalingState !== "stable");

      ignoreOffer = !polite && offerCollision;
      if (ignoreOffer) {
        return;
      }

      await pc.setRemoteDescription(description);
      if (description.type === "offer") {
        await pc.setLocalDescription();
        signaler.send({ description: pc.localDescription });
      }
    } else if (candidate) {
      try {
        await pc.addIceCandidate(candidate);
      } catch (err) {
        if (!ignoreOffer) {
          throw err;
        }
      }
    }
  } catch (err) {
    console.error(err);
  }
};
```

虽然代码大小差异不大，复杂度也未显著降低，但代码的可靠性大大提高。让我们深入探讨代码以了解它现在是如何工作的。

##### 收到描述

在修订后的代码中，如果收到的消息是 SDP `description`，我们检查它是否在我们尝试发送提议时到达。如果收到的消息是 `offer` 并且本地对等方是不礼貌的，并且发生冲突，则忽略该提议，因为我们希望继续尝试使用正在发送的提议。这是不礼貌的对等方在行动。

在任何其他情况下，我们将尝试处理收到的消息。这首先通过将收到的 `description` 传递给 `setRemoteDescription()` 来设置远程描述。这适用于处理提议或响应，因为需要时会自动回滚。

此时，如果收到的消息是 `offer`，我们使用 `setLocalDescription()` 来创建并设置适当的本地描述，然后通过信令服务器将其发送回远程对等方。

##### 收到 ICE 候选

另一方面，如果收到的消息是 ICE 候选——由 JSON 对象中包含 `candidate` 成员指示——我们通过调用 `RTCPeerConnection` 方法 `addIceCandidate()` 将其传递给本地 ICE 层。与之前一样，如果因为冲突的提议而忽略，则忽略此处的任何错误。

### 添加显式的 restartIce() 方法

过去在处理 `negotiationneeded` 事件时触发 ICE 重启的技术存在显著缺陷。这些缺陷使得在协商期间安全可靠地触发重启变得困难。完美协商的改进通过向 `RTCPeerConnection` 添加新的 `restartIce()` 方法解决了这个问题。

#### 旧方法

过去，如果遇到 ICE 错误并需要重新启动协商，你可能会这样做：

```javascript
pc.onnegotiationneeded = async (options) => {
  await pc.setLocalDescription(await pc.createOffer(options));
  signaler.send({ description: pc.localDescription });
};
pc.oniceconnectionstatechange = () => {
  if (pc.iceConnectionState === "failed") {
    pc.onnegotiationneeded({ iceRestart: true });
  }
};
```

这存在许多可靠性问题和明显的错误（例如，如果 `iceconnectionstatechange` 事件在信令状态不是 `stable` 时触发，则会失败），但你实际上无法通过其他方式请求 ICE 重启，除了创建并发送带有 `iceRestart` 选项设置为 `true` 的提议。发送重启请求因此需要直接调用 `negotiationneeded` 事件处理程序。正确实现这一点非常棘手，而且很容易出错，导致错误很常见。

#### 使用 restartIce()

现在，你可以使用 `restartIce()` 更清晰地实现这一点：

```javascript
let makingOffer = false;

pc.onnegotiationneeded = async () => {
  try {
    makingOffer = true;
    await pc.setLocalDescription();
    signaler.send({ description: pc.localDescription });
  } catch (err) {
    console.error(err);
  } finally {
    makingOffer = false;
  }
};
pc.oniceconnectionstatechange = () => {
  if (pc.iceConnectionState === "failed") {
    pc.restartIce();
  }
};
```

通过这种改进的技术，而不是直接带有选项调用 `onnegotiationneeded` 来触发 ICE 重启，`failed` ICE 连接状态会调用 `restartIce()`。`restartIce()` 告诉 ICE 层自动在下一个发送的 ICE 消息中添加 `iceRestart` 标志。问题解决了！

### pranswer 状态不再支持回滚

最后一个突出的 API 变更是，你不能再在 `have-remote-pranswer` 或 `have-local-pranswer` 状态下回滚。幸运的是，当使用完美协商时，反正不需要这样做，因为导致这种情况的情况会在需要回滚之前被捕捉和预防。

因此，尝试在两个 `pranswer` 状态之一时触发回滚，现在将抛出 `InvalidStateError`。


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

