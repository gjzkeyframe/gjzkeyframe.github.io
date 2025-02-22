---
title: HLS 协议：直播回放首选的传输协议
description: 介绍 HLS 协议的基础知识。
author: Keyframe
date: 2025-02-22 07:28:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 传输协议, HLS]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }


（本文基本逻辑：HLS 协议概览）


## 1、HLS 协议概览


**HLS（HTTP Live Streaming）**是由苹果公司提出的一种流媒体传输协议，可支持流媒体的直播和点播。对于 HLS 点播，基本上就是常见的分段 HTTP 点播，不同在于，它的分段非常小。要实现 HLS 点播，重点在于对媒体文件分段。对于 HLS 直播，相对于常见的流媒体直播协议，例如 RTMP 协议、RTSP 协议等，HLS 最大的不同在于直播客户端获取到的并不是一个完整的数据流，而是连续的、短时长的媒体文件（如 MPEG-TS 格式），客户端不断的下载并播放这些小文件。由于数据通过 HTTP 协议传输，所以完全不用考虑防火墙或者代理的问题，而且分段文件的时长很短，客户端可以很快的选择和切换码率，以适应不同带宽条件下的播放。不过 HLS 的这种技术特点，决定了它的延迟一般总是会高于普通的流媒体直播协议。

**HLS 作为苹果公司提出的协议，在 iOS 客户端上得到了很好的支持，比如 AVPlayer 和 Safari 都支持对 HLS 流媒体的播放；再加上 M3U8/TS 封装格式可以在直播中持续处理和存储流媒体数据，所以直播回放通常都会选择 HLS 协议来实现。**

HLS 协议的实现是和 M3U8 文件的定义密切相关的，这部分的知识在[《M3U8 格式》](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)中已经做了详细介绍。在这里只简单介绍一下 HLS 协议的整体框架。

HLS 协议的整体框架如下图所示：

![](assets/resource/av-basic-knowledge/av-protocol-hls-1.png)

HLS 协议涉及到 3 个部分：

**1）服务端组件**

服务端组件主要的职责是处理上传链路的媒体流，并对齐进行编码和格式封装，为资源分发做好准备。


**2）分发组件**


分发组件一般是一组 Web 服务器，主要负责接受客户端的请求并将媒体数据和相关的资源分发给客户端去消费。一般需要通过 CDN 网络来进行资源分发。


**3）客户端软件**

客户端软件一般是指播放器，主要负责请求媒体资源，并对其进行解封装、解码、渲染等一系列的处理从而展现给用户。

在典型的 HLS 协议实现中，一般是采集音频和视频输入，将它们编码为 H.264 和 AAC 格式，最终经过服务端组件处理成 MPEG-2 的传输流。在这个过程中，MPEG-2 的数据流会被处理为一系列连续的小切片文件（.ts）存储在 Web 服务器上，同时服务器会生成一个索引文件对这些切片文件进行索引，并将这个索引文件发布出去。客户端请求和读取该索引文件，并请求和获取其中的切片文件从而获得对应的媒体资源数据来进行处理和展示。




## 2、实战解析

### 2.1、工具介绍

这里介绍两款可以在 Chrome 浏览器里播放 HLS 视频的插件：

- [Play HLS M3U8](https://www.extfans.com/productivity/ckblfoghkjhaclegefojbgllenffajdc/)
- [Native HLS Playback](https://www.extfans.com/productivity/emnphkkblegpebimobpbekeedfgemhof)

有了这个插件，我们就可以配合 Chrome 的 Inspect/Network 能力来抓取 HLS 加载 M3U8 和 ts 切片的请求的细节信息。

![](assets/resource/av-basic-knowledge/av-protocol-hls-2.png)


## 3、问题集锦

1、如何提升 HLS 的播放秒开？

HLS 的播放秒开分为「直接开播」和「开播 seek」的情况。

1）直接开播

对于「直接开播」的场景，播放起播速度跟播放器的策略有很大的关系。比如 iOS 的 AVPlayer 可能需要下载 3 个 ts 切片才会开始播放。IJKPlayer 则使用水位线策略下载到一定量的数据就能开播，这样相对起播会更快一些。

2）开播 seek

通常我们会用 HLS 来保存直播的回放文件，由于一场直播的时间通常较长，在观看回放时，通常需要 seek 到某一个位置来定位到用户感兴趣的内容。

对于一般的播放器，可能需要先初始化播放器并从头加载播放内容，再由业务做 seek 操作，这样会比较慢。IJKPlayer 有 seek-at-start 能力，直接去下载目前位置的数据，不用从头加载再 seek，优化开播就 seek 的速度。

一般 HLS 的 ts 切片是按照直播的 GOP 来切片的，如果 seek 到某个 ts 切片的中间位置，会需要从这个 ts 切片的开始位置下载数据并解码，再计算 seek 到的位置来展示画面，这样的 seek 过程会比较慢。对于这个问题，服务端可以根据直播内容打点将一场直播切成多个 m3u8，不再是整场内容只使用一个 m3u8，而是一个内容段对应一个 m3u8。这样可以尽量保证用户点击内容锚点时，是直接从头播放一个 m3u8，不用 seek，从而优化响应速度。

如果用户自己拖动进度来 seek，这时候就是实实在在的需要对 seek 进行优化了，这里我们在使用 IJKPlayer 时有一个优化点：有时候会遇到 video packet duration 会有为空的情况，而 IJKPlayer 是以 video 缓冲区水位线来驱动起播的，这样由于有点 video packet 的 duration 为空，会导致为了累积足够的水位下载了实际时长超过水位线的视频数据才开播，这就导致 seek 较慢，对于这个问题，可以改为：以 audio 缓冲区水位线驱动起播，因为 audio packet 的 duration 通常都是正常的，这样可以优化 seek 速度。


我们还遇到过 HLS seek 黑屏的问题。主要是启动就 seek 时，`seek_position = stream.start_time + seek_duration`，stream 的 first_timestamp 未正确初始化，导致无法找到 seek 的位置，seek 失败。

此外，IJKPlayer 支持 EXT-X-DISCONTINUITY 标签有问题，需要解决跨断层 ts seek 的问题。seek 位置解析到的 pts 如果有断层，要加上断层前的所有 ts 的时长。


2、HLS 使用 IJKPlayer 在 seek(0) 时为什么会偶现失败的问题？

IJKPlayer 在播放 HLS 时 seek(0) 时，播放器的实现逻辑是 seek 到 `0 + start_time`，即加上了时间基础。同时，在处理这个时间时，会向上向下取整，这样就有可能由于取整误差导致 seek(0) 对应 timestamp 可能小于视频最小 pts，这时候就会出错。对于这个问题，可以在 seek(0) 处理时间戳时 +1，而不是向下取整。


## 小结


通过上文的介绍，我们了解了 HLS 协议的基础知识，HLS 协议配合 M3U8 和 TS 封装格式可以应用在直播和点播场景。




## 本文参考

- [HLS 协议](https://developer.apple.com/streaming/)
- [HLS](https://datatracker.ietf.org/doc/html/draft-pantos-http-live-streaming-23)




