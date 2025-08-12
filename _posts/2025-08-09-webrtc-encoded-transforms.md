---
title: 探索 WebRTC 开发（9）：WebRTC 编码转换
description: 本文将介绍 WebRTC 编码转换。
author: Keyframe
date: 2025-08-09 18:08:08 +0800
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


## 有限的可用性

WebRTC 编码转换提供了一种注入高性能流 API 的机制，用于修改编码后的视频和音频帧，将其注入到 WebRTC 的传入和传出管道中。这使得第三方代码能够实现诸如对编码后的帧进行端到端加密等使用场景。

该 API 定义了主线程和工作线程中的对象。主线程接口是一个 `RTCRtpScriptTransform` 实例，其构造函数指定了实现转换器代码的 `Worker`。通过将 `RTCRtpScriptTransform` 添加到 `RTCRtpReceiver.transform` 或 `RTCRtpSender.transform`，可以将运行在工作线程中的转换插入到传入或传出的 WebRTC 管道中。

在工作线程中创建了一个对应的 `RTCRtpScriptTransformer` 对象，它具有一个可读流（`readable` 属性）、一个可写流（`writable` 属性）以及一个从关联的 `RTCRtpScriptTransform` 构造函数传递过来的选项对象（`options`）。来自 WebRTC 管道的编码视频帧（`RTCEncodedVideoFrame`）或音频帧（`RTCEncodedAudioFrame`）会被排队到 `readable` 上以供处理。

`RTCRtpScriptTransformer` 通过 `rtctransform` 事件的 `transformer` 属性提供给代码，该事件在工作线程的全局作用域中触发，每当编码帧被排队以供处理时（以及在对应的 `RTCRtpScriptTransform` 构造时最初触发）。

工作线程代码必须实现该事件的处理程序，从 `transformer.readable` 读取编码帧，按需修改它们，并将它们按相同的顺序且不重复地写回到 `transformer.writable`。

虽然接口对实现没有其他限制，但一种自然的帧转换方式是创建一个管道链，将 `event.transformer.readable` 流中的帧通过 `TransformStream` 发送到 `event.transformer.writable` 流。我们可以使用 `event.transformer.options` 属性来配置依赖于是否将传入帧从分片器排队或从编码器输出帧的转换代码。

`RTCRtpScriptTransformer` 接口还提供了在发送编码视频时可以让编码器生成“关键”帧的方法，以及在接收视频时请求发送新的关键帧的方法。如果接收者加入会议通话时正在发送差异帧，这可能允许接收者更快地开始查看视频。

以下示例提供了如何使用基于 `TransformStream` 的实现来使用该框架的更具体示例。

## 测试是否支持编码转换

通过检查 `RTCRtpSender.transform`（或 `RTCRtpReceiver.transform`）是否存在来测试是否支持编码转换：

```javascript
const supportsEncodedTransforms =
  window.RTCRtpSender && "transform" in RTCRtpSender.prototype;
```

## 为传出帧添加转换

通过将 `RTCRtpScriptTransform` 实例赋值给传出轨道的 `RTCRtpSender.transform` 属性，可以将运行在工作线程中的转换插入到传出的 WebRTC 管道中。

以下示例展示了如何通过 WebRTC 流式传输用户网络摄像头的视频，并添加 WebRTC 编码转换以修改传出流。代码假设已经存在一个名为 `peerConnection` 的 RTCPeerConnection，并且已经连接到远程对等方。

首先，我们使用 `getUserMedia()` 获取一个视频 MediaStream，然后通过 `MediaStream.getTracks()` 方法获取流中的第一个 MediaStreamTrack。

通过 `addTrack()` 将轨道添加到对等连接中，这将开始将轨道流式传输到远程对等方。`addTrack()` 方法返回正在用于发送轨道的 RTCRtpSender。

```javascript
// 获取视频流和媒体轨道
const stream = await navigator.mediaDevices.getUserMedia({ video: true });
const [track] = stream.getTracks();
const videoSender = peerConnection.addTrack(track, stream);
```

然后，我们构造一个 `RTCRtpScriptTransform`，它接受一个工作线程脚本，该脚本定义了转换，并且可以传递一个可选对象，用于向工作线程传递任意消息（在此示例中，我们使用了一个 `name` 属性，其值为 "senderTransform"，以告诉工作线程此转换将被添加到传出流中）。我们通过将其赋值给 `RTCRtpSender.transform` 属性，将转换添加到传出管道中。

```javascript
// 创建一个包含 TransformStream 的工作线程
const worker = new Worker("worker.js");
videoSender.transform = new RTCRtpScriptTransform(worker, {
  name: "senderTransform",
});
```

## 为传入帧添加转换

通过将 `RTCRtpScriptTransform` 实例赋值给传入轨道的 RTCRtpReceiver.transform 属性，可以将运行在工作线程中的转换插入到传入的 WebRTC 管道中。

以下示例展示了如何添加转换以修改传入流。代码假设已经存在一个名为 `peerConnection` 的 RTCPeerConnection，并且已经连接到远程对等方。

首先，我们添加一个 RTCPeerConnection 的 `track` 事件处理程序，以捕获对等方开始接收新轨道时的事件。在处理程序中，我们构造一个 RTCRtpScriptTransform，并将其添加到 event.receiver.transform（event.receiver 是一个 RTCRtpReceiver）。与上一节类似，构造函数接受一个具有 `name` 属性的对象，但在此我们使用 "receiverTransform" 作为值，以告诉工作线程帧是传入的。

```javascript
peerConnection.ontrack = (event) => {
  const worker = new Worker("worker.js");
  event.receiver.transform = new RTCRtpScriptTransform(worker, {
    name: "receiverTransform",
  });
  received_video.srcObject = event.streams[0];
};
```

同样，你可以在任何时间添加转换流，但在 `track` 事件处理程序中添加它，以确保转换流将获得该轨道的第一个编码帧。

## 工作线程实现

工作线程脚本必须实现一个 `rtctransform` 事件的处理程序，创建一个管道链，将 `event.transformer.readable`（ReadableStream）流通过 TransformStream 管道传输到 event.transformer.writable（WritableStream）流。

工作线程可能支持转换传入或传出的编码帧，或者两者都转换，并且转换可能是硬编码的，或者使用从 Web 应用程序传递的信息在运行时配置。

### 基本的 WebRTC 编码转换

以下示例展示了一个基本的 WebRTC 编码转换，它对所有排队帧中的位取反。它不需要从主线程传递选项，因为可以在发送方管道中使用相同的算法对位取反，并在接收方管道中恢复它们。

代码实现了 `rtctransform` 事件的处理程序。这构造了一个 TransformStream，然后使用 ReadableStream.pipeThrough() 通过它进行管道传输，最后使用 ReadableStream.pipeTo() 管道传输到 event.transformer.writable。

```javascript
addEventListener("rtctransform", (event) => {
  const transform = new TransformStream({
    start() {}, // 启动时调用。
    flush() {}, // 在流即将关闭时调用。
    async transform(encodedFrame, controller) {
      // 重构原始帧。
      const view = new DataView(encodedFrame.data);

      // 构造一个新缓冲区
      const newData = new ArrayBuffer(encodedFrame.data.byteLength);
      const newView = new DataView(newData);

      // 对传入帧中的所有位取反
      for (let i = 0; i < encodedFrame.data.byteLength; ++i) {
        newView.setInt8(i, ~view.getInt8(i));
      }

      encodedFrame.data = newData;
      controller.enqueue(encodedFrame);
    },
  });
  event.transformer.readable
    .pipeThrough(transform)
    .pipeTo(event.transformer.writable);
});
```

该实现与“通用”的 TransformStream 类似，但也有一些重要区别。与通用流一样，其构造函数接受一个对象，该对象定义了一个可选的 start() 方法（构造时调用），flush() 方法（在流即将关闭时调用），以及 transform() 方法（每次有块需要处理时调用）。

该 transform() 方法的不同之处在于，它接收的是 RTCEncodedVideoFrame 或 RTCEncodedAudioFrame，而不是通用的“块”。此处显示的方法代码并不显眼，除了它演示了如何将帧转换为可以修改的形式，并在之后将其重新排队到流中。

### 使用独立的发送方和接收方转换

如果发送和接收时转换函数相同，上述示例可以正常工作，但在许多情况下，算法会有所不同。你可以为发送方和接收方使用独立的工作线程脚本，或者像下面这样在一个工作线程中处理两种情况。

如果工作线程同时用于发送方和接收方，它需要知道当前编码帧是来自编解码器的传出帧，还是来自分片器的传入帧。此信息可以通过 RTCRtpScriptTransform 构造函数的第二个选项指定。例如，我们可以为发送方和接收方定义独立的 RTCRtpScriptTransform，传递相同的工作线程，并传递一个具有名称属性的选项对象，以指示转换是用于发送方还是接收方（如上文所述）。
该信息在工作线程中可通过 event.transformer.options 获得。

在此示例中，我们在全局专用工作线程作用域对象上实现了 onrtctransform 事件处理程序。名称属性的值用于确定要构造哪种 TransformStream（实际的构造方法未显示）。

```javascript
// 实例化转换并将它们附加到发送方/接收方管道的代码。
onrtctransform = (event) => {
  let transform;
  if (event.transformer.options.name == "senderTransform")
    transform = createSenderTransform(); // 返回 TransformStream
  else if (event.transformer.options.name == "receiverTransform")
    transform = createReceiverTransform(); // 返回 TransformStream
  else return;
  event.transformer.readable
    .pipeThrough(transform)
    .pipeTo(event.transformer.writable);
};
```

请注意，创建管道链的代码与上一示例相同。

### 与转换的运行时通信

RTCRtpScriptTransform 构造函数允许你传递选项和转移对象到工作线程。在上一示例中，我们传递了静态信息，但有时你可能需要在运行时修改工作线程中的转换算法，或者从工作线程获取信息。例如，支持加密的 WebRTC 会议通话可能需要将新密钥添加到转换使用的算法中。

虽然可以使用 Worker.postMessage() 在运行转换代码的工作线程和主线程之间共享信息，但通常更简单的方法是将 MessageChannel 作为 RTCRtpScriptTransform 构造函数的选项传递，因为这样通道上下文在处理新编码帧时直接在 event.transformer.options 中可用。

以下代码创建了一个 MessageChannel，并将第二个端口转移到工作线程。主线程和转换可以使用第一个和第二个端口进行通信。

```javascript
// 创建一个包含 TransformStream 的工作线程
const worker = new Worker("worker.js");

// 创建一个通道
// 将 channel.port2 作为构造函数选项传递给转换器
// 并将其转移到工作线程
const channel = new MessageChannel();
const transform = new RTCRtpScriptTransform(
  worker,
  { purpose: "encrypt", port: channel.port2 },
  [channel.port2],
);

// 使用 port1 发送字符串。
// （我们可以发送和转移基本类型/对象）。
channel.port1.postMessage("给工作线程的消息");
channel.port1.start();
```

在工作线程中，端口作为 event.transformer.options.port 可用。以下代码显示了如何监听端口的 message 事件以获取主线程的消息。你也可以使用端口向主线程发送消息。

```javascript
event.transformer.options.port.onmessage = (event) => {
  // 消息载荷在 'event.data' 中；
  console.log(event.data);
};
```

### 触发关键帧

由于原始视频可以表示为完整图像，因此很少被发送或存储，因为它会占用大量空间和带宽，因此需要高效的存储方式。相反，编解码器会定期生成一个“关键帧”，其中包含构建完整图像所需的信息，并在关键帧之间发送“差异帧”，这些帧仅包含自上一个差异帧以来的变化。虽然这种效率远高于发送原始视频，但意味着为了显示与特定差异帧相关的图像，你需要最后一个关键帧和所有后续差异帧。

这可能会导致新的用户在加入 WebRTC 会议应用程序时出现延迟，因为他们需要接收到第一个关键帧才能显示视频。同样，如果使用编码转换对帧进行加密，接收者在接收到第一个使用其密钥加密的关键帧之前将无法显示视频。

为了确保在需要时可以尽快发送新的关键帧，event.transformer 中的 RTCRtpScriptTransformer 对象提供了两种方法：RTCRtpScriptTransformer.generateKeyFrame()，它会使编解码器生成关键帧，以及 RTCRtpScriptTransformer.sendKeyFrameRequest()，接收者可以使用它来请求发送方发送关键帧。

以下示例展示了主线程如何将加密密钥传递给发送方转换，并触发编解码器生成关键帧。主线程无法直接访问 RTCRtpScriptTransformer 对象，因此需要通过工作线程传递密钥和限制标识符（“rid”）（“rid” 是一个流标识符，它指示必须生成关键帧的编码器）。
这里假设已经存在对等连接，并且 videoSender 是 RTCRtpSender。

```javascript
const worker = new Worker("worker.js");
const channel = new MessageChannel();

videoSender.transform = new RTCRtpScriptTransform(
  worker,
  { name: "senderTransform", port: channel.port2 },
  [channel.port2],
);

// 向发送方发送 rid 和新密钥
channel.port1.start();
channel.port1.postMessage({
  rid: "1",
  key: "93ae0927a4f8e527f1gce6d10bc6ab6c",
});
```

工作线程中的 rtctransform 事件处理程序获取端口并使用它来监听主线程的 message 事件。如果接收到事件，它将获取 rid 和密钥，然后调用 generateKeyFrame()。

```javascript
event.transformer.options.port.onmessage = (event) => {
  const { rid, key } = event.data;
  // 密钥用于转换器加密帧（未显示）

  // 使用 rid 获取编解码器生成新的关键帧
  // 这里的 rcEvent 是 rtctransform 事件。
  rcEvent.transformer.generateKeyFrame(rid);
};
```

接收方请求新关键帧的代码几乎完全相同，只是不指定 “rid”。以下是仅端口消息处理程序的代码：

```javascript
event.transformer.options.port.onmessage = (event) => {
  const { key } = event.data;
  // 密钥用于转换器解密帧（未显示）

  // 请求发送方发送关键帧。
  transformer.sendKeyFrameRequest();
};
```




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

