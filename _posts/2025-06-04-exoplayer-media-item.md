---
title: ExoPlayer 从入门到精通（4）：媒体项
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-04 18:08:08 +0800
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


播放列表 API 基于 `MediaItem` 实例，这些实例可以方便地通过 `MediaItem.Builder` 构建。在播放器内部，`MediaItem` 通过 `MediaSource.Factory` 转换为可播放的 `MediaSource`。在没有自定义配置的情况下，此转换由 `DefaultMediaSourceFactory` 执行，它能够根据媒体项的属性构建复杂的媒体源。下面概述了可以在媒体项上设置的一些属性。

仅包含流 URI 的媒体项可以使用 `fromUri` 快捷方法构建：

### Kotlin

```kotlin
val mediaItem = MediaItem.fromUri(videoUri)
```

### Java

```java
MediaItem mediaItem = MediaItem.fromUri(videoUri);
```

对于所有其他情况，可以使用 `MediaItem.Builder`。在以下示例中，使用 ID 和一些附加元数据构建了一个媒体项：

### Kotlin

```kotlin
val mediaItem = MediaItem.Builder().setMediaId(mediaId).setTag(myAppData).setUri(videoUri).build()
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder().setMediaId(mediaId).setTag(myAppData).setUri(videoUri).build();
```

附加元数据对于在播放列表过渡时更新应用的 UI 非常有用。

## 图像

图像的播放需要在媒体项中指定持续时间，以确定图像在播放过程中应显示的时间。有关 Motion Photos 和图像加载库（例如 Glide）的更多信息，请参阅图像指南页面。

### Kotlin

```kotlin
val mediaItem = MediaItem.Builder().setUri(imageUri).setImageDurationMs(3000).build()
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder().setUri(imageUri).setImageDurationMs(3_000).build();
```

## 自适应媒体的非标准文件扩展名

ExoPlayer 为 DASH、HLS 和 SmoothStreaming 提供了自适应媒体源。如果此类自适应媒体项的 URI 以标准文件扩展名结尾，则会自动创建相应的媒体源。如果 URI 具有非标准扩展名或根本没有扩展名，则可以明确设置 MIME 类型以指示媒体项的类型：

### Kotlin

```kotlin
val mediaItem = MediaItem.Builder().setUri(hlsUri).setMimeType(MimeTypes.APPLICATION_M3U8).build()
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder().setUri(hlsUri).setMimeType(MimeTypes.APPLICATION_M3U8).build();
```

对于渐进式媒体流，不需要设置 MIME 类型。

## 受保护的内容

对于受保护的内容，应设置媒体项的 DRM 属性。UUID 是必需的，其他属性是可选的。

以下是一个配置示例，用于播放受 Widevine DRM 保护的项目，其中许可证 URI 不直接在媒体中可用（例如在 DASH 播放列表中），并且需要多个会话（例如由于密钥轮换）：

### Kotlin

```kotlin
val mediaItem =
  MediaItem.Builder()
    .setUri(videoUri)
    .setDrmConfiguration(
      MediaItem.DrmConfiguration.Builder(C.WIDEVINE_UUID)
        .setLicenseUri(licenseUri)
        .setMultiSession(true)
        .setLicenseRequestHeaders(httpRequestHeaders)
        .build()
    )
    .build()
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder()
        .setUri(videoUri)
        .setDrmConfiguration(
            new MediaItem.DrmConfiguration.Builder(C.WIDEVINE_UUID)
                .setLicenseUri(licenseUri)
                .setMultiSession(true)
                .setLicenseRequestHeaders(httpRequestHeaders)
                .build())
        .build();
```

在播放器内部，`DefaultMediaSourceFactory` 将这些属性传递给 `DrmSessionManagerProvider` 以获取 `DrmSessionManager`，然后将其注入到创建的 `MediaSource` 中。DRM 行为可以根据需要进一步自定义。

## 旁加载字幕轨道

要旁加载字幕轨道，可以在构建媒体项时添加 `MediaItem.Subtitle` 实例：

### Kotlin

```kotlin
val subtitle =
  SubtitleConfiguration.Builder(subtitleUri)
    .setMimeType(mimeType) // 正确的 MIME 类型（必需）。
    .setLanguage(language) // 字幕语言（可选）。
    .setSelectionFlags(selectionFlags) // 轨道的选择标志（可选）。
    .build()
val mediaItem =
  MediaItem.Builder().setUri(videoUri).setSubtitleConfigurations(listOf(subtitle)).build()
```

### Java

```java
MediaItem.SubtitleConfiguration subtitle =
    new MediaItem.SubtitleConfiguration.Builder(subtitleUri)
        .setMimeType(mimeType) // 正确的 MIME 类型（必需）。
        .setLanguage(language) // 字幕语言（可选）。
        .setSelectionFlags(selectionFlags) // 轨道的选择标志（可选）。
        .build();
MediaItem mediaItem =
    new MediaItem.Builder()
        .setUri(videoUri)
        .setSubtitleConfigurations(ImmutableList.of(subtitle))
        .build();
```

在内部，`DefaultMediaSourceFactory` 将使用 `MergingMediaSource` 将内容媒体源与每个字幕轨道的 `SingleSampleMediaSource` 合并。`DefaultMediaSourceFactory` 不支持为多周期 DASH 旁加载字幕。

要剪辑媒体项所引用的内容，可以设置自定义的开始和结束位置：

### Kotlin

```kotlin
val mediaItem =
  MediaItem.Builder()
    .setUri(videoUri)
    .setClippingConfiguration(
      MediaItem.ClippingConfiguration.Builder()
        .setStartPositionMs(startPositionMs)
        .setEndPositionMs(endPositionMs)
        .build()
    )
    .build()
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder()
        .setUri(videoUri)
        .setClippingConfiguration(
            new ClippingConfiguration.Builder()
                .setStartPositionMs(startPositionMs)
                .setEndPositionMs(endPositionMs)
                .build())
        .build();
```

在内部，`DefaultMediaSourceFactory` 将使用 `ClippingMediaSource` 包装内容媒体源。还有其他剪辑属性。更多详细信息，请参阅 `MediaItem.Builder` 的 Javadoc。

## 广告插入

要插入广告，需要设置媒体项的广告标签 URI 属性：

### Kotlin

```kotlin
val mediaItem =
  MediaItem.Builder()
    .setUri(videoUri)
    .setAdsConfiguration(MediaItem.AdsConfiguration.Builder(adTagUri).build())
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder()
        .setUri(videoUri)
        .setAdsConfiguration(new MediaItem.AdsConfiguration.Builder(adTagUri).build())
        .build();
```

在内部，`DefaultMediaSourceFactory` 将把内容媒体源包装在 `AdsMediaSource` 中，以根据广告标签插入广告。要使此功能正常工作，播放器还需要相应地配置其 `DefaultMediaSourceFactory`。

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

