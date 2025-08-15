---
title: 音视频通话延时问题的分析和解决
description: 介绍音视频通话延时问题的分析和解决思路。
author: Keyframe
date: 2025-02-25 14:08:08 +0800
categories: [音视频实战经验]
tags: [音视频实战经验, 音视频, 实时通话, WebRTC, 通话延时]
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






关于 WebRTC 视频通话延迟优化的复杂技术问题，我们来梳理一下思路。

首先，要理解视频通话中的延迟到底是如何产生的。整个流程中涉及采集、编码、传输、解码、渲染等多个环节，每个环节都可能造成延迟。

从采集开始，摄像头采集视频帧、麦克风采集音频采样，这里就会有**硬件延迟**。然后是视频编码，H.264 这类编码器为了达到更好的压缩率，会采用帧间预测，这就引入了**编码延迟**。

网络传输方面，WebRTC 默认使用 UDP 进行传输，虽然避免了 TCP 的三次握手和重传延迟，但 NAT 穿透、丢包重传等机制仍会带来额外延迟。ICE 建立连接的过程也会消耗时间。这些都是**网络延迟**。

接收端又要进行解码、音视频同步、渲染等操作，这些都会增加**接收端处理延迟**。特别是在移动设备上，硬件性能可能会成为瓶颈。

基于这些分析，我们来思考一下优化方案，首先采集环节，可以降低分辨率和帧率来减少数据量。编码方面可以权衡压缩率和延迟，选择更低延迟的配置。

网络传输延迟是个大头，可以优化 ICE 的配置，尽量使用更优的传输路径。实现带宽评估和码率自适应，在网络不佳时及时降低码率。

此外，还有一些细节需要考虑，比如是否启用抖动缓冲区，缓冲区大小如何配置，是否使用硬件加速等。这些都会影响最终的延迟表现。

接下来，我把这些想法组织成一个完整的技术方案，包括详细的优化措施、架构设计和关键代码等等。






## 1、延迟产生的主要原因

1、采集和预处理延迟

- 摄像头硬件采集延迟
- 视频帧预处理(美颜、滤镜等)耗时
- 音频采样和前处理延迟

2、编码延迟

- 视频编码器配置不合理
- 帧间预测带来的延迟
- 编码缓冲区累积

3、网络传输延迟

- ICE建立连接耗时
- NAT穿透延迟
- 网络抖动和丢包重传
- 带宽受限导致数据堆积

4、解码和渲染延迟

- 解码器延迟
- 音视频同步等待
- 渲染队列堆积
- 显示设备刷新延迟

## 2、优化方案设计

**WebRTC 延迟优化系统架构图：**

![WebRTC 延迟优化系统架构图](assets/resource/av-experience/video-call-delay-opt-1.png)
_WebRTC 延迟优化系统架构图_

**延迟优化流程图：**

![延迟优化流程图](assets/resource/av-experience/video-call-delay-opt-2.png)
_延迟优化流程图_


**延迟优化数据流转图：**

![延迟优化数据流转图](assets/resource/av-experience/video-call-delay-opt-3.png)
_延迟优化数据流转图_


## 3、具体优化措施

**3.1 采集端优化参考代码**

```javascript
// 采集配置优化
const constraints = {
  video: {
    width: { ideal: 640 }, // 降低分辨率
    height: { ideal: 480 },
    frameRate: { ideal: 15 }, // 降低帧率
    latency: { ideal: 0 } // 最低延迟模式
  },
  audio: {
    echoCancellation: false, // 关闭不必要的音频处理
    noiseSuppression: false,
    autoGainControl: false
  }
};

// 实现采集数据监控
class CaptureMonitor {
  constructor() {
    this.timestamps = new Map();
  }
  
  onFrameCaptured(frame) {
    const captureTime = performance.now();
    this.timestamps.set(frame.id, captureTime);
    this.checkCaptureLatency(frame);
  }

  checkCaptureLatency(frame) {
    const threshold = 100; // 100ms延迟阈值
    const latency = performance.now() - this.timestamps.get(frame.id);
    if (latency > threshold) {
      this.adjustCaptureParams(); // 动态调整采集参数
    }
  }
}
```

**3.2 编码优化参考代码**

```javascript
// 编码器配置优化
const encoderConfig = {
  codec: 'H264',
  mode: 'realtime',
  profile: 'baseline',
  bitrate: 800000, // 合理的码率
  keyFrameInterval: 2, // 缩短关键帧间隔
  numberOfTemporalLayers: 1, // 禁用时域分层编码
  latencyMode: 'ultralow' // 超低延迟模式
};

// 码率自适应控制
class BitrateController {
  constructor() {
    this.currentBitrate = encoderConfig.bitrate;
    this.minBitrate = 200000;
    this.maxBitrate = 1500000;
  }

  adjustBitrate(stats) {
    const rtt = stats.roundTripTime;
    const packetsLost = stats.packetsLost;
    
    if (rtt > 200 || packetsLost > 0.1) {
      this.decreaseBitrate();
    } else if (rtt < 100 && packetsLost < 0.05) {
      this.increaseBitrate();
    }
  }

  decreaseBitrate() {
    this.currentBitrate = Math.max(
      this.minBitrate,
      this.currentBitrate * 0.8
    );
    this.applyNewBitrate();
  }
}
```

**3.3 网络传输优化参考代码**

```javascript
// ICE配置优化
const iceConfig = {
  iceServers: [
    { urls: 'stun:stun.example.com' },
    { 
      urls: 'turn:turn.example.com',
      username: 'username',
      credential: 'password'
    }
  ],
  iceTransportPolicy: 'relay', // 优先使用TURN中继
  iceCandidatePoolSize: 1, // 减少候选收集时间
};

// 实现带宽估计
class BandwidthEstimator {
  constructor() {
    this.windowSize = 500; // 500ms估计窗口
    this.samples = [];
  }

  addSample(bytes, timestamp) {
    this.samples.push({bytes, timestamp});
    this.cleanOldSamples();
    return this.estimateBandwidth();
  }

  estimateBandwidth() {
    if (this.samples.length < 2) return null;
    const first = this.samples[0];
    const last = this.samples[this.samples.length - 1];
    const duration = last.timestamp - first.timestamp;
    const bytes = last.bytes - first.bytes;
    return (bytes * 8) / (duration / 1000); // bps
  }
}
```

**3.4 接收端优化参考代码**

```javascript
// 解码器配置
const decoderConfig = {
  codec: 'H264',
  latencyMode: 'ultralow',
  renderDelay: 0, // 最小渲染延迟
  outputMode: 'low_delay'
};

// 实现延迟监控
class LatencyMonitor {
  constructor() {
    this.e2eLatency = 0;
    this.jitterBufferDelay = 0;
  }

  updateStats(stats) {
    this.e2eLatency = stats.currentRoundTripTime * 1000;
    this.jitterBufferDelay = stats.jitterBufferDelay;
    this.checkLatency();
  }

  checkLatency() {
    const totalLatency = this.e2eLatency + this.jitterBufferDelay;
    if (totalLatency > 300) { // 300ms阈值
      this.triggerLatencyOptimization();
    }
  }

  triggerLatencyOptimization() {
    // 调整抖动缓冲区
    this.adjustJitterBuffer();
    // 请求降低发送端码率
    this.requestBitrateDecrease();
  }
}
```

## 4、关键优化建议

1、采集端

- 使用较低的分辨率和帧率启动通话，根据网络状况动态调整
- 关闭不必要的视频前处理
- 开启摄像头的低延迟模式

2、编码端

- 使用低延迟编码配置
- 合理设置码率和关键帧间隔
- 实现智能码率自适应

3、网络传输

- 优化 ICE 配置，加快连接建立
- 实现精确的带宽估计算法
- 动态调整发送缓冲区大小

4、接收端

- 使用较小的抖动缓冲区
- 开启硬件解码加速
- 实现端到端延迟监控

5、监控和调优

- 实时监控各环节延迟
- 根据监控数据动态调整参数
- 建立完整的延迟优化反馈环

## 5、预期效果

通过以上优化措施，预计可以将端到端延迟控制在以下范围：

- 理想网络条件：150-200ms
- 一般网络条件：200-300ms
- 较差网络条件：300-500ms

需要注意的是，延迟优化往往需要在质量和延迟之间做平衡，建议根据具体场景需求和网络条件来调整优化策略。




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

