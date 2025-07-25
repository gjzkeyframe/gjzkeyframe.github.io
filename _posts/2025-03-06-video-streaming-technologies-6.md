---
title: 探索视频流媒体技术（6）：视频转码的实用示例和最佳实践
description: 系列介绍视频流媒体技术相关的基础知识。
author: Keyframe
date: 2025-03-06 08:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 音频, 流媒体]
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


>这个系列文章我们来介绍一位海外工程师如何探索视频流媒体技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，这是第 6 篇：视频转码的实用示例和最佳实践。

## 1、实用示例

### 1.1、使用 FFmpeg 进行基础编码：

```bash
ffmpeg -i input.mov -c:v libx265 -crf 28 -preset fast -c:a aac -b:a 128k output.mp4
```

- **说明**：该命令将 `input.mov` 转换为 `output.mp4`，使用 H.265 编解码器对视频进行编码，CRF（恒定速率因子）为 28（平衡质量和文件大小），音频使用 AAC 编解码器，码率为 128 kbps。

### 1.2、为兼容性进行转码：

```bash
ffmpeg -i input.mkv -c:v libx264 -c:a copy output.mp4
```

- **说明**：将 `input.mkv` 转换为 `output.mp4`，将视频重新编码为 H.264，同时复制音频流而不重新编码。

## 2、最佳实践

- **选择合适的编解码器**：使用 H.264 以获得广泛的兼容性，使用 H.265 或 VP9 以获得更好的压缩效率，使用 AV1 以适应未来的发展。
- **优化码率**：通过选择合适的码率来平衡质量和带宽。使用 FFmpeg 的 CRF 模式等工具来调整质量。
- **保持宽高比**：确保输出视频保持原始宽高比，以避免失真。
- **批量处理**：使用脚本自动化编码任务，提高效率，尤其是在处理大量文件时。
- **跨设备测试**：验证编码后的视频能否在所有目标设备和平台上正确播放。

## 3、编码/转码在自适应码率流式传输中的作用

自适应码率流式传输（ABS）是一种通过互联网传输视频的技术，它可以根据观众的网络条件动态调整视频流的质量。即使在网络连接不稳定的情况下，也能确保流畅的观看体验。

- **编码多个码率**：视频以多个码率和分辨率进行编码。例如，一个视频可能会被编码为 240p、480p、720p 和 1080p。
- **清单文件**：清单文件（如 HLS 的 M3U8 或 DASH 的 MPD）列出了可用的视频流及其码率。
- **动态切换**：流式传输客户端会监控网络条件，并在不同码率的流之间切换，以保持最佳播放效果。例如，如果网络速度下降，客户端可能会从 1080p 切换到 720p。
- **提升用户体验**：通过以多种码率对视频进行编码和转码，ABS 确保观众获得尽可能高的质量，同时避免中断。

### 3.1、使用 FFmpeg 为 ABS 实现的实用示例

要为 HLS 生成多个码率的流，可以使用如下命令：

```bash
ffmpeg -i input.mp4 -filter_complex "[0:v]split=3[v1][v2][v3];[v1]scale=w=1280:h=720[v1out];[v2]scale=w=854:h=480[v2out];[v3]scale=w=640:h=360[v3out]" \
  -map "[v1out]" -c:v:0 libx264 -b:v:0 3000k -map "[v2out]" -c:v:1 libx264 -b:v:1 1500k -map "[v3out]" -c:v:2 libx264 -b:v:2 800k \
  -map a -c:a aac -b:a 128k -f hls -hls_time 10 -hls_playlist_type vod \
  -hls_segment_filename "output_%v/segment_%03d.ts" -master_pl_name "output.m3u8" \
  "output_%v/output.m3u8"
```

- **说明**：该命令将输入视频分为三个分辨率（720p、480p、360p），以不同的码率（3000k、1500k、800k）对它们进行编码，并生成 HLS 段和播放列表。

## 4、结论

视频编码和转码是视频流媒体工作流程中的关键环节，确保内容被高效压缩，并且与各种设备和网络条件兼容。像 FFmpeg 和 HandBrake 这样的工具为编码和转码提供了强大的功能，而最佳实践有助于优化视频质量和性能。在自适应码率流式传输中，对多个码率进行编码发挥着至关重要的作用，以提供无缝的观看体验。了解这些流程，并利用合适的工具和技术，可以极大地提升你的视频流媒体解决方案。




---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

