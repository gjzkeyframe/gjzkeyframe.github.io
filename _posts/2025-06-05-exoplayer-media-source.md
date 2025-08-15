---
title: ExoPlayer 从入门到精通（5）：媒体源
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-05 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 音频, 拍摄, ExoPlayer]
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



## 媒体源流程

在 ExoPlayer 中，每个媒体文件都由一个 `MediaItem` 表示。但在内部，播放器需要 `MediaSource` 实例来播放内容。播放器通过 `MediaSource.Factory` 从媒体项创建这些实例。

默认情况下，播放器使用 `DefaultMediaSourceFactory`，它可以创建以下内容 `MediaSource` 实现的实例：

- `DashMediaSource` 用于 DASH。
- `SsMediaSource` 用于 SmoothStreaming。
- `HlsMediaSource` 用于 HLS。
- `ProgressiveMediaSource` 用于普通媒体文件。
- `RtspMediaSource` 用于 RTSP。

`DefaultMediaSourceFactory` 还可以根据相应媒体项的属性创建更复杂的媒体源。这在 [媒体项页面](https://developer.android.com/media/media3/exoplayer/media-items) 中有更详细的描述。

对于需要默认配置无法支持的媒体源设置的应用，有几种自定义选项。

在构建播放器时，可以注入 `MediaSource.Factory`。例如，如果应用需要插入广告并使用 `CacheDataSource.Factory` 来支持缓存，可以配置 `DefaultMediaSourceFactory` 以满足这些需求，并在构建播放器时注入：

### Kotlin

```kotlin
val mediaSourceFactory: MediaSource.Factory =
    DefaultMediaSourceFactory(context)
        .setDataSourceFactory(cacheDataSourceFactory)
        .setLocalAdInsertionComponents(adsLoaderProvider, playerView)
val player = ExoPlayer.Builder(context).setMediaSourceFactory(mediaSourceFactory).build()
```

### Java

```java
MediaSource.Factory mediaSourceFactory =
    new DefaultMediaSourceFactory(context)
        .setDataSourceFactory(cacheDataSourceFactory)
        .setLocalAdInsertionComponents(adsLoaderProvider, /* adViewProvider= */ playerView);
ExoPlayer player =
    new ExoPlayer.Builder(context).setMediaSourceFactory(mediaSourceFactory).build();
```

`DefaultMediaSourceFactory` 的 JavaDoc 详细描述了可用选项。

还可以注入自定义的 `MediaSource.Factory` 实现，例如支持创建自定义媒体源类型。工厂的 `createMediaSource(MediaItem)` 将被调用来为添加到播放列表的每个媒体项创建媒体源。

`ExoPlayer` 接口定义了额外的播放列表方法，这些方法接受媒体源而不是媒体项。这使得可以绕过播放器的内部 `MediaSource.Factory`，直接将媒体源实例传递给播放器：

### Kotlin

```kotlin
// 设置媒体源列表作为初始播放列表。
exoPlayer.setMediaSources(listOfMediaSources)
// 添加单个媒体源。
exoPlayer.addMediaSource(anotherMediaSource)

// 可以与媒体项 API 结合使用。
exoPlayer.addMediaItem(/* index= */ 3, MediaItem.fromUri(videoUri))

exoPlayer.prepare()
exoPlayer.play()
```

### Java

```java
// 设置媒体源列表作为初始播放列表。
exoPlayer.setMediaSources(listOfMediaSources);
// 添加单个媒体源。
exoPlayer.addMediaSource(anotherMediaSource);

// 可以与媒体项 API 结合使用。
exoPlayer.addMediaItem(/* index= */ 3, MediaItem.fromUri(videoUri));

exoPlayer.prepare();
exoPlayer.play();
```

ExoPlayer 提供了多种 `MediaSource` 实现，用于修改和组合其他 `MediaSource` 实例。在需要组合多种自定义设置且简单的设置路径不足以满足需求时，这些实现最为有用。

- `ClippingMediaSource`：允许将媒体剪辑到指定的时间戳范围。如果这是唯一的修改，最好使用 `MediaItem.ClippingConfiguration`。
- `FilteringMediaSource`：过滤可用轨道到指定类型，例如，仅从包含音频和视频的文件中公开视频轨道。如果这是唯一的修改，最好使用轨道选择参数。
- `MergingMediaSource`：合并多个媒体源以并行播放。在几乎所有情况下，建议在构造函数中将 `adjustPeriodTimeOffsets` 和 `clipDurations` 设置为 true，以确保所有源同时开始和结束。如果此修改是为了添加旁加载的字幕，最好使用 `MediaItem.SubtitleConfiguration`。
- `ConcatenatingMediaSource2`：合并多个媒体源以连续播放。用户可见的媒体结构公开一个单一的 `Timeline.Window`，这意味着它看起来像一个单独的项目。如果此修改是为了播放不应看起来像一个单独项目的多个项目，最好使用播放列表 API 方法，如 `Player.addMediaItem`。
- `SilenceMediaSource`：为指定时长生成静音，用于填充间隙。
- `AdsMediaSource`：扩展媒体源，使其具有客户端广告插入功能。详情请参阅广告插入指南。
- `ServerSideAdInsertionMediaSource`：扩展媒体源，使其具有服务器端广告插入功能。详情请参阅广告插入指南。

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

