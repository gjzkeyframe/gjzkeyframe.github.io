---
title: ExoPlayer 从入门到精通（9）：获取元数据
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-09 18:08:08 +0800
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


## 在播放期间

在播放期间，可以通过多种方式获取媒体的元数据。最直接的方式是监听 `Player.Listener#onMediaMetadataChanged` 事件；这将提供一个 `MediaMetadata` 对象供使用，该对象包含 `title` 和 `albumArtist` 等字段。或者，调用 `Player#getMediaMetadata` 也会返回相同的对象。

### Kotlin

```kotlin
override fun onMediaMetadataChanged(mediaMetadata: MediaMetadata) {
  mediaMetadata.title?.let(::handleTitle)
}
```

### Java

```java
@Override
public void onMediaMetadataChanged(MediaMetadata mediaMetadata) {
  if (mediaMetadata.title != null) {
    handleTitle(mediaMetadata.title);
  }
}
```

如果您的应用需要访问特定的 `Metadata.Entry` 对象，那么它应该监听 `Player.Listener#onMetadata`（用于在播放过程中传递的动态元数据）。或者，如果需要查看静态元数据，可以通过 `TrackSelections#getFormat` 访问。`Player#getMediaMetadata` 从这两个来源填充数据。

## 在不播放的情况下

如果不需要播放，使用 `MetadataRetriever` 提取元数据更为高效，因为它无需创建和准备播放器。

### Kotlin

```kotlin
val trackGroupsFuture = MetadataRetriever.retrieveMetadata(context, mediaItem)
Futures.addCallback(
  trackGroupsFuture,
  object : FutureCallback<TrackGroupArray?> {
    override fun onSuccess(trackGroups: TrackGroupArray?) {
      if (trackGroups != null) handleMetadata(trackGroups)
    }

    override fun onFailure(t: Throwable) {
      handleFailure(t)
    }
  },
  executor
)
```

### Java

```java
ListenableFuture<TrackGroupArray> trackGroupsFuture =
    MetadataRetriever.retrieveMetadata(context, mediaItem);
Futures.addCallback(
    trackGroupsFuture,
    new FutureCallback<TrackGroupArray>() {
      @Override
      public void onSuccess(TrackGroupArray trackGroups) {
        handleMetadata(trackGroups);
      }

      @Override
      public void onFailure(Throwable t) {
        handleFailure(t);
      }
    },
    executor);
```

## 动态照片

还可以提取动态照片元数据，包括文件中图像和视频部分的偏移量和长度。

对于动态照片，使用 `MetadataRetriever` 获取的 `TrackGroupArray` 包含一个 `TrackGroup`，其中包含一个包含 `MotionPhotoMetadata` 元数据条目的 `Format`。

### Kotlin

```kotlin
0.until(trackGroups.length)
  .asSequence()
  .mapNotNull { trackGroups[it].getFormat(0).metadata }
  .filter { metadata -> metadata.length() == 1 }
  .map { metadata -> metadata[0] }
  .filterIsInstance<MotionPhotoMetadata>()
  .forEach(::handleMotionPhotoMetadata)
```

### Java

```java
for (int i = 0; i < trackGroups.length; i++) {
  TrackGroup trackGroup = trackGroups.get(i);
  Metadata metadata = trackGroup.getFormat(0).metadata;
  if (metadata != null && metadata.length() == 1) {
    Metadata.Entry metadataEntry = metadata.get(0);
    if (metadataEntry instanceof MotionPhotoMetadata) {
      MotionPhotoMetadata motionPhotoMetadata = (MotionPhotoMetadata) metadataEntry;
      handleMotionPhotoMetadata(motionPhotoMetadata);
    }
  }
}
```

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

