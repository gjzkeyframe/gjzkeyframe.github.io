---
title: 探索视频流媒体技术（7）：自适应码率流式传输（ABS）
description: 系列介绍视频流媒体技术相关的基础知识。
author: Keyframe
date: 2025-03-07 08:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 音频, 流媒体]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
>_微信扫码关注我们_
>
>您还可以加入知识星球 `关键帧的音视频开发圈` 来一起交流工作中的**技术难题、职场经验**：
>
>![知识星球](assets/img/keyframe-zsxq.png)
>_微信扫码加入星球_
{: .prompt-tip }


>这个系列文章我们来介绍一位海外工程师如何探索视频流媒体技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，这是第 7 篇：自适应码率流式传输（ABS）。



自适应码率流式传输（ABS）是在线视频传输领域的一项关键技术，通过动态调整视频质量以适应观众的网络条件，确保流畅的播放体验。在本文中，我们将深入探讨 ABS 的工作原理、其优势与挑战，以及如何在你的流媒体服务中实现它。

## 1、自适应码率流式传输的解释

**自适应码率流式传输（ABS）** 是一种在互联网上流式传输视频的方法，无论网络条件如何变化，都能为用户提供尽可能好的体验。与传统的单一质量视频流传输方法不同，ABS 提供多个不同码率和分辨率的视频流。

- **多个视频流**：视频内容以多种码率和分辨率进行编码。
- **动态调整**：流式传输客户端根据实时网络条件和设备能力在这些视频流之间切换。
- **无缝播放**：结果是无缝的观看体验，缓冲最少，视频质量最优。

## 2、ABS 的优势和挑战

### 2.1、优势

- **提升观众体验**：通过适应观众的网络速度，ABS 最小化缓冲并确保连续播放。
- **优化带宽使用**：ABS 提供观众互联网连接所能支持的最高质量流，必要时节省带宽。
- **设备兼容性**：它适应不同设备的能力，确保在高端和低端设备上都能获得最佳观看效果。

### 2.2、挑战

- **复杂性**：实现 ABS 需要在多个码率下对视频进行编码，创建清单文件，并确保流式传输服务器和客户端支持动态切换。
- **增加存储需求**：存储每个视频的多个版本会增加存储需求。
- **延迟**：如果管理不当，分段和在流之间切换可能会引入延迟。

## 3、ABR 的工作原理

### 3.1、播放列表清单

播放列表清单是 ABS 的关键组成部分。它是一个列出所有可用视频流及其码率和段 URL 的文件。

- **HLS（HTTP Live Streaming）**：使用 M3U8 清单文件。
- **DASH（Dynamic Adaptive Streaming over HTTP）**：使用 MPD（Media Presentation Description）文件。

HLS 播放列表清单（`master.m3u8`）示例：

```plaintext
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360
low/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1500000,RESOLUTION=854x480
mid/index.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=3000000,RESOLUTION=1280x720
high/index.m3u8
```

### 3.2、分段

视频被分割成小段（通常为 2-10 秒长）。每一段都在多个码率和分辨率下进行编码。

- **一致性**：每一段都是一个可以独立播放的独立文件。
- **同步性**：所有码率和分辨率必须同步，以确保顺畅切换。

不同码率的示例段文件：

- `low/segment1.ts`
- `mid/segment1.ts`
- `high/segment1.ts`

### 3.3、动态切换

流式传输客户端持续监控观众的网络条件，并切换到合适的流，以提供尽可能好的质量而不中断。

- **缓冲**：客户端保持少量视频数据缓冲区，以便于顺畅切换。
- **监控**：它测量下载速度、缓冲状态和设备性能等参数，以决定何时切换流。

### 3.4、在你的流媒体服务中实现 ABS

#### 第一步：编码视频

使用 FFmpeg 等工具在多个码率和分辨率下对视频进行编码。

生成不同码率的 HLS 段的 FFmpeg 命令示例：

```bash
ffmpeg -i input.mp4 \
  -vf "scale=w=640:h=360" -c:v h264 -b:v 800k -g 48 -keyint_min 48 -profile:v baseline -preset veryfast -c:a aac -b:a 128k -hls_time 4 -hls_playlist_type vod -hls_segment_filename 'low/segment%d.ts' low.m3u8 \
  -vf "scale=w=1280:h=720" -c:v h264 -b:v 2500k -g 48 -keyint_min 48 -profile:v main -preset veryfast -c:a aac -b:a 128k -hls_time 4 -hls_playlist_type vod -hls_segment_filename 'mid/segment%d.ts' mid.m3u8 \
  -vf "scale=w=1920:h=1080" -c:v h264 -b:v 5000k -g 48 -keyint_min 48 -profile:v high -preset veryfast -c:a aac -b:a 128k -hls_time 4 -hls_playlist_type vod -hls_segment_filename 'high/segment%d.ts' high.m3u8
```

#### 第二步：创建清单文件

生成列出可用流的清单文件（HLS 的 M3U8 或 DASH 的 MPD）。

主清单（`output.m3u8`）示例：

```plaintext
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360
output_2/output.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1500000,RESOLUTION=854x480
output_1/output.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=3000000,RESOLUTION=1280x720
output_0/output.m3u8
```

#### 第三步：设置流媒体服务器

配置流媒体服务器以提供分段视频文件和清单文件。流行的服务器包括：

- **带有 RTMP 模块的 NGINX**：适用于 HLS。
- **Apache HTTP Server**：可以配置为提供 HLS/DASH 内容。
- **AWS Elemental Media Services**：提供可扩展的 ABS 解决方案。

#### 第四步：集成流媒体客户端

使用支持 ABS 的播放器，例如：

- **Video.js**：一个流行的 HTML5 视频播放器，支持 HLS 和 DASH。
- **Shaka Player**：一个开源的 JavaScript 库，用于 DASH 和 HLS。
- **JW Player**：一个具有丰富 ABS 功能的商业播放器。

#### 第五步：测试和优化

- **模拟网络条件**：在各种网络条件下测试播放器，以确保顺畅切换。
- **监控性能**：使用分析工具监控观众体验，并根据需要优化设置。
- **微调编码设置**：调整码率阶梯、段持续时间和缓冲区大小，以实现最佳性能。

### 结论

自适应码率流式传输是一种强大的技术，用于通过互联网提供高质量的视频体验。通过了解 ABS 的工作原理、其优势与挑战以及实现步骤，你可以确保你的流媒体服务为观众提供流畅且不间断的播放体验，无论他们的网络条件如何。随着流媒体技术的不断发展，掌握 ABS 将是保持竞争力和满足用户期望的关键。

