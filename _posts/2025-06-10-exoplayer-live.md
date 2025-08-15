---
title: ExoPlayer 从入门到精通（10）：直播流
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-10 18:08:08 +0800
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


ExoPlayer 在无需特殊配置的情况下，能够开箱即用地播放大多数自适应直播流。如需了解更多详情，请参阅支持格式页面。

自适应直播流提供了一个定期更新的可用媒体窗口，以跟随当前实时时间移动。这意味着播放位置将始终位于此窗口内，通常靠近流被生成的实时时间。当前实时时间与播放位置之间的差异称为直播偏移。

## 检测和监控直播播放

每当直播窗口更新时，注册的 `Player.Listener` 实例将收到 `onTimelineChanged` 事件。您可以通过查询 `Player` 和 `Timeline.Window` 的各种方法来获取当前直播播放的详细信息，如下图所示。

![img](https://developer.android.com/static/guide/topics/media/exoplayer/images/live-window.png)

- `Player.isCurrentWindowLive` 表示当前播放的媒体项是否为直播流。即使直播流已结束，此值仍为 true。
- `Player.isCurrentWindowDynamic` 表示当前播放的媒体项是否仍在更新。对于尚未结束的直播流，此值通常为 true。请注意，对于某些非直播流，此标志也可能为 true。
- `Player.getCurrentLiveOffset` 返回当前实时时间与播放位置之间的偏移量（如果可用）。
- `Player.getDuration` 返回当前直播窗口的时长。
- `Player.getCurrentPosition` 返回相对于直播窗口起始位置的播放位置。
- `Player.getCurrentMediaItem` 返回当前媒体项，其中 `MediaItem.liveConfiguration` 包含应用提供的目标直播偏移和直播偏移调整参数。
- `Player.getCurrentTimeline` 返回当前媒体结构的 `Timeline`。可以通过 `Player.getCurrentMediaItemIndex` 和 `Timeline.getWindow` 从 `Timeline` 中检索当前 `Timeline.Window`。在 `Window` 中：

  - `Window.liveConfiguration` 包含目标直播偏移和直播偏移调整参数。这些值基于媒体中的信息以及在 `MediaItem.liveConfiguration` 中设置的任何应用提供的覆盖值。
  - `Window.windowStartTimeMs` 是直播窗口开始时间自 Unix 纪元以来的时间。
  - `Window.getCurrentUnixTimeMs` 是当前实时时间自 Unix 纪元以来的时间。此值可能会根据服务器和客户端之间已知的时钟差异进行修正。
  - `Window.getDefaultPositionMs` 是播放器默认开始播放的直播窗口中的位置。

## 在直播流中寻址

可以使用 `Player.seekTo` 在直播窗口内任意位置进行寻址。传递的寻址位置相对于直播窗口的起始位置。例如，`seekTo(0)` 将寻址到直播窗口的起始位置。寻址后，播放器将尝试保持与寻址到的位置相同的直播偏移。

直播窗口还有一个默认位置，播放从此位置开始。此位置通常靠近直播边缘。可以通过调用 `Player.seekToDefaultPosition` 寻址到默认位置。

## 直播播放 UI

ExoPlayer 的默认 UI 组件显示直播窗口的时长以及当前播放位置在窗口内的位置。这意味着每次直播窗口更新时，位置将向后跳跃。如果需要不同的行为，例如显示 Unix 时间或当前直播偏移，可以 fork `PlayerControlView` 并根据需要进行修改。

## 配置直播播放参数

ExoPlayer 使用一些参数来控制播放位置与直播边缘的偏移，以及可用于调整此偏移的播放速度范围。

ExoPlayer 从以下三个地方获取这些参数的值，优先级从高到低（找到的第一个值将被使用）：

- 通过 `MediaItem.Builder.setLiveConfiguration` 传递的每个 `MediaItem` 的值。
- 在 `DefaultMediaSourceFactory` 上设置的全局默认值。
- 直接从媒体中读取的值。

### Kotlin

```kotlin
// 全局设置。
val player =
  ExoPlayer.Builder(context)
    .setMediaSourceFactory(DefaultMediaSourceFactory(context).setLiveTargetOffsetMs(5000))
    .build()

// 每个 MediaItem 的设置。
val mediaItem =
  MediaItem.Builder()
    .setUri(mediaUri)
    .setLiveConfiguration(
      MediaItem.LiveConfiguration.Builder().setMaxPlaybackSpeed(1.02f).build()
    )
    .build()
player.setMediaItem(mediaItem)
```

### Java

```java
// 全局设置。
ExoPlayer player =
    new ExoPlayer.Builder(context)
        .setMediaSourceFactory(
            new DefaultMediaSourceFactory(context).setLiveTargetOffsetMs(5000))
        .build();

// 每个 MediaItem 的设置。
MediaItem mediaItem =
    new MediaItem.Builder()
        .setUri(mediaUri)
        .setLiveConfiguration(
            new MediaItem.LiveConfiguration.Builder().setMaxPlaybackSpeed(1.02f).build())
        .build();
player.setMediaItem(mediaItem);
```

可用的配置值包括：

- `targetOffsetMs`：目标直播偏移。如果可能，播放器将尝试在播放过程中接近此直播偏移。
- `minOffsetMs`：允许的最小直播偏移。即使在根据当前网络条件调整偏移时，播放器也不会尝试在播放过程中低于此偏移。
- `maxOffsetMs`：允许的最大直播偏移。即使在根据当前网络条件调整偏移时，播放器也不会尝试在播放过程中超过此偏移。
- `minPlaybackSpeed`：播放器可用于回退以达到目标直播偏移的最小播放速度。
- `maxPlaybackSpeed`：播放器可用于赶上目标直播偏移的最大播放速度。

## 播放速度调整

在播放低延迟直播流时，ExoPlayer 通过轻微改变播放速度来调整直播偏移。播放器将尝试匹配媒体或应用提供的目标直播偏移，但也会对不断变化的网络条件做出反应。例如，如果在播放过程中出现重新缓冲，播放器将稍微降低播放速度，以远离直播边缘。如果网络随后变得足够稳定，可以再次支持靠近直播边缘播放，播放器将加快播放速度，以回到目标直播偏移。

如果不需要自动播放速度调整，可以通过将 `minPlaybackSpeed` 和 `maxPlaybackSpeed` 属性设置为 `1.0f` 来禁用。同样，对于非低延迟直播流，可以通过将这些属性显式设置为非 `1.0f` 的值来启用。有关如何设置这些属性的更多详细信息，请参阅上面的配置部分。

### 自定义播放速度调整算法

如果启用了速度调整，`LivePlaybackSpeedControl` 将定义进行哪些调整。可以实现自定义 `LivePlaybackSpeedControl`，或者自定义默认实现 `DefaultLivePlaybackSpeedControl`。在这两种情况下，可以在构建播放器时设置实例。

### Kotlin

```kotlin
val player =
  ExoPlayer.Builder(context)
    .setLivePlaybackSpeedControl(
      DefaultLivePlaybackSpeedControl.Builder().setFallbackMaxPlaybackSpeed(1.04f).build()
    )
    .build()
```

### Java

```java
ExoPlayer player =
    new ExoPlayer.Builder(context)
        .setLivePlaybackSpeedControl(
            new DefaultLivePlaybackSpeedControl.Builder()
                .setFallbackMaxPlaybackSpeed(1.04f)
                .build())
        .build();
```

`DefaultLivePlaybackSpeedControl` 的相关自定义参数包括：

- `fallbackMinPlaybackSpeed` 和 `fallbackMaxPlaybackSpeed`：如果媒体和应用提供的 `MediaItem` 均未定义限制，则可用于调整的最小和最大播放速度。
- `proportionalControlFactor`：控制速度调整的平滑程度。较高值使调整更突然、更反应迅速，但也更可能被察觉。较低值则使速度之间的过渡更平滑，但调整速度较慢。
- `targetLiveOffsetIncrementOnRebufferMs`：每次发生重新缓冲时，此值将被加到目标直播偏移上，以便更谨慎地进行调整。通过将此值设置为 0 可以禁用此功能。
- `minPossibleLiveOffsetSmoothingFactor`：用于根据当前缓冲的媒体跟踪最小可能直播偏移的指数平滑因子。非常接近 1 的值意味着估计更谨慎，可能需要更长时间才能适应改善的网络条件，而较低值则意味着估计将更快调整，但存在更高遇到重新缓冲的风险。

## BehindLiveWindowException 和 ERROR_CODE_BEHIND_LIVE_WINDOW

如果播放位置落后于直播窗口，例如播放器暂停或缓冲的时间足够长，就会发生这种情况。如果发生，播放将失败，并通过 `Player.Listener.onPlayerError` 报告一个错误代码为 `ERROR_CODE_BEHIND_LIVE_WINDOW` 的异常。应用程序代码可能希望处理此类错误，通过在默认位置恢复播放。示例应用的 PlayerActivity 就采用了这种方法。

### Kotlin

```kotlin
override fun onPlayerError(error: PlaybackException) {
  if (error.errorCode == PlaybackException.ERROR_CODE_BEHIND_LIVE_WINDOW) {
    // 在直播边缘重新初始化播放器。
    player.seekToDefaultPosition()
    player.prepare()
  } else {
    // 处理其他错误
  }
}
```

### Java

```java
@Override
public void onPlayerError(PlaybackException error) {
  if (error.errorCode == PlaybackException.ERROR_CODE_BEHIND_LIVE_WINDOW) {
    // 在直播边缘重新初始化播放器。
    player.seekToDefaultPosition();
    player.prepare();
  } else {
    // 处理其他错误
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

