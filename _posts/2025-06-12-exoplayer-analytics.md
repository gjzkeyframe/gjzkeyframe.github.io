---
title: ExoPlayer 从入门到精通（12）：分析
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-12 18:08:08 +0800
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




ExoPlayer 支持广泛的播放分析需求。分析最终是关于收集、解释、聚合和总结播放数据。这些数据可以用于设备上（例如用于日志记录、调试或告知未来的播放决策），或者报告给服务器以监控所有设备上的播放情况。

分析系统通常需要先收集事件，然后进一步处理它们以使其有意义：

- **事件收集**：可以通过在 ExoPlayer 实例上注册 AnalyticsListener 来完成。注册的分析监听器会在播放过程中接收事件。每个事件都与播放列表中的相应媒体项以及播放位置和时间戳元数据相关联。
- **事件处理**：一些分析系统将原始事件上传到服务器，所有事件处理都在服务器端完成。也可以在设备上处理事件，这可能更简单或减少需要上传的信息量。ExoPlayer 提供了 PlaybackStatsListener，允许您执行以下处理步骤：

1. **事件解释**：为了对分析有用，事件需要在单个播放的上下文中进行解释。例如，播放器状态更改为 STATE_BUFFERING 的原始事件可能对应于初始缓冲、重新缓冲或在寻址后发生的缓冲。
2. **状态跟踪**：这一步将事件转换为计数器。例如，状态更改事件可以转换为跟踪在每个播放状态下花费的时间的计数器。结果是一组基本的单个播放分析数据值。
3. **聚合**：这一步将多个播放中的分析数据结合起来，通常通过累加计数器。
4. **总结指标计算**：许多最有用的指标是计算平均值或以其他方式组合基本分析数据值的。总结指标可以为单个或多个播放计算。

## 使用 AnalyticsListener 进行事件收集

来自播放器的原始播放事件会报告给 AnalyticsListener 实现。您可以轻松添加自己的监听器，并仅覆盖您感兴趣的方法：

### Kotlin

```kotlin
exoPlayer.addAnalyticsListener(
  object : AnalyticsListener {
    override fun onPlaybackStateChanged(
      eventTime: EventTime, @Player.State state: Int
    ) {}

    override fun onDroppedVideoFrames(
      eventTime: EventTime,
      droppedFrames: Int,
      elapsedMs: Long,
    ) {}
  }
)
```

### Java

```java
exoPlayer.addAnalyticsListener(
    new AnalyticsListener() {
      @Override
      public void onPlaybackStateChanged(
          EventTime eventTime, @Player.State int state) {}

      @Override
      public void onDroppedVideoFrames(
          EventTime eventTime, int droppedFrames, long elapsedMs) {}
    });
```

传递给每个回调的 EventTime 将事件与播放列表中的媒体项相关联，并包括播放位置和时间戳元数据：

- realtimeMs：事件的时钟时间。
- timeline、windowIndex 和 mediaPeriodId：定义事件所属的播放列表和播放列表中的项目。mediaPeriodId 包含可选的附加信息，例如指示事件是否属于项目内的广告。
- eventPlaybackPositionMs：事件发生时项目中的播放位置。
- currentTimeline、currentWindowIndex、currentMediaPeriodId 和 currentPlaybackPositionMs：如上，但针对当前播放的项目。当前播放的项目可能与事件所属的项目不同，例如如果事件对应于下一个要播放项目的预缓冲。

## 使用 PlaybackStatsListener 进行事件处理

PlaybackStatsListener 是一个在设备上进行事件处理的 AnalyticsListener。它计算 PlaybackStats，包括计数器和派生指标，例如：

- 汇总指标，例如总播放时间。
- 自适应播放质量指标，例如平均视频分辨率。
- 渲染质量指标，例如丢帧率。
- 资源使用指标，例如通过网络读取的字节数。

您可以在 PlaybackStatsListener 中找到播放列表中每个媒体项以及这些项目中插入的每个客户端广告的单独 PlaybackStats。您可以为 PlaybackStatsListener 提供一个回调，以在播放结束时获得通知，并使用回调传递的 EventTime 来识别哪个播放已结束。您可以使用 PlaybackStatsListener.getPlaybackStats() 在任何时间查询当前播放会话的 PlaybackStats。

### Kotlin

```kotlin
exoPlayer.addAnalyticsListener(
  PlaybackStatsListener(/* keepHistory= */ true) {
    eventTime: EventTime?,
    playbackStats: PlaybackStats?,
    -> // 从 `eventTime` 开始的会话的分析数据已准备好。
  }
)
```

### Java

```java
exoPlayer.addAnalyticsListener(
    new PlaybackStatsListener(
        /* keepHistory= */ true,
        (eventTime, playbackStats) -> {
          // 从 `eventTime` 开始的会话的分析数据已准备好。
        }));
```

PlaybackStatsListener 的构造函数提供了保留处理事件完整历史记录的选项。请注意，这可能会根据播放长度和事件数量带来未知的内存开销。因此，您应该只在需要访问处理事件的完整历史记录时才启用它，而不仅仅是为了获取最终的分析数据。

PlaybackStats 使用扩展的状态集来指示媒体的状态以及用户的播放意图和更多详细信息，例如播放中断或结束的原因：

| 播放状态 | 用户意图播放 | 无播放意图 |
| --- | --- | --- |
| 播放前 | `JOINING_FOREGROUND` | `NOT_STARTED`、`JOINING_BACKGROUND` |
| 活动播放 | `PLAYING` |  |
| 中断播放 | `BUFFERING`、`SEEKING` | `PAUSED`、`PAUSED_BUFFERING`、`SUPPRESSED`、`SUPPRESSED_BUFFERING`、`INTERRUPTED_BY_AD` |
| 结束状态 |  | `ENDED`、`STOPPED`、`FAILED`、`ABANDONED` |

用户的播放意图对于区分用户积极等待播放继续的时间与被动等待时间非常重要。例如，PlaybackStats.getTotalWaitTimeMs 返回 `JOINING_FOREGROUND`、`BUFFERING` 和 `SEEKING` 状态的总时间，但不包括播放暂停的时间。类似地，PlaybackStats.getTotalPlayAndWaitTimeMs 将返回用户意图播放的总时间，即活动等待时间与 `PLAYING` 状态的总时间之和。

### 已处理和解释的事件

您可以通过使用 PlaybackStatsListener 并设置 keepHistory=true 来记录已处理和解释的事件。生成的 PlaybackStats 将包含以下事件列表：

- playbackStateHistory：按顺序排列的扩展播放状态列表，以及它们开始适用的 EventTime。您还可以使用 PlaybackStats.getPlaybackStateAtTime 来查找给定时钟时间的状态。
- mediaTimeHistory：时钟时间和媒体时间对的历史记录，允许您重建媒体的哪些部分在哪些时间播放。您还可以使用 PlaybackStats.getMediaTimeMsAtRealtimeMs 来查找给定时钟时间的播放位置。
- videoFormatHistory 和 audioFormatHistory：播放期间使用的视频和音频格式的有序列表，以及它们开始使用的时间。
- fatalErrorHistory 和 nonFatalErrorHistory：致命错误和非致命错误的有序列表，以及它们发生的时间。致命错误是那些导致播放结束的错误，而非致命错误可能是可恢复的。

### 单次播放的分析数据

如果您使用 PlaybackStatsListener，即使 keepHistory=false，也会自动收集这些数据。最终值是 PlaybackStats Javadoc 中的公共字段以及 playbackStateDurationsMs 返回的播放状态持续时间。为了方便，您还会找到 getTotalPlayTimeMs 和 getTotalWaitTimeMs 等方法，它们返回特定播放状态组合的持续时间。

### Kotlin

```kotlin
Log.d(
  "DEBUG",
  "Playback summary: " +
    "play time = " +
    playbackStats.totalPlayTimeMs +
    ", rebuffers = " +
    playbackStats.totalRebufferCount
)
```

### Java

```java
Log.d(
    "DEBUG",
    "Playback summary: "
        + "play time = "
        + playbackStats.getTotalPlayTimeMs()
        + ", rebuffers = "
        + playbackStats.totalRebufferCount);
```

### 多次播放的聚合分析数据

您可以通过调用 PlaybackStats.merge 将多个 PlaybackStats 合并在一起。生成的 PlaybackStats 将包含所有合并播放的聚合数据。请注意，它将不包含单个播放事件的历史记录，因为这些历史记录无法聚合。

PlaybackStatsListener.getCombinedPlaybackStats 可用于获取 PlaybackStatsListener 生命周期内收集的所有分析数据的聚合视图。

### 计算的总结指标

除了基本分析数据外，PlaybackStats 还提供了许多方法来计算总结指标。

### Kotlin

```kotlin
Log.d(
  "DEBUG",
  "Additional calculated summary metrics: " +
    "average video bitrate = " +
    playbackStats.meanVideoFormatBitrate +
    ", mean time between rebuffers = " +
    playbackStats.meanTimeBetweenRebuffers
)
```

### Java

```java
Log.d(
    "DEBUG",
    "Additional calculated summary metrics: "
        + "average video bitrate = "
        + playbackStats.getMeanVideoFormatBitrate()
        + ", mean time between rebuffers = "
        + playbackStats.getMeanTimeBetweenRebuffers());
```

## 高级主题

### 将分析数据与播放元数据关联

在为单个播放收集分析数据时，您可能希望将播放分析数据与正在播放的媒体的元数据关联起来。

建议使用 MediaItem.Builder.setTag 为媒体设置特定的元数据。媒体标签是报告原始事件和 PlaybackStats 结束时的 EventTime 的一部分，因此在处理相应的分析数据时可以轻松检索到：

### Kotlin

```kotlin
PlaybackStatsListener(/* keepHistory= */ false) {
  eventTime: EventTime,
  playbackStats: PlaybackStats ->
  val mediaTag =
    eventTime.timeline
      .getWindow(eventTime.windowIndex, Timeline.Window())
      .mediaItem
      .localConfiguration
      ?.tag
    // 使用 mediaTag 元数据报告 playbackStats。
}
```

### Java

```java
new PlaybackStatsListener(
    /* keepHistory= */ false,
    (eventTime, playbackStats) -> {
      Object mediaTag =
          eventTime.timeline.getWindow(eventTime.windowIndex, new Timeline.Window())
              .mediaItem
              .localConfiguration
              .tag;
      // 使用 mediaTag 元数据报告 playbackStats。
    });
```

### 报告自定义分析事件

如果您需要将自定义事件添加到分析数据中，需要将这些事件保存在您自己的数据结构中，并在之后将它们与报告的 PlaybackStats 结合起来。如果需要，您可以扩展 DefaultAnalyticsCollector 以能够为您的自定义事件生成 EventTime 实例，并将它们发送到已注册的监听器，如以下示例所示。

### Kotlin

```kotlin
private interface ExtendedListener : AnalyticsListener {
  fun onCustomEvent(eventTime: EventTime)
}

private class ExtendedCollector : DefaultAnalyticsCollector(Clock.DEFAULT) {
  fun customEvent() {
    val eventTime = generateCurrentPlayerMediaPeriodEventTime()
    sendEvent(eventTime, CUSTOM_EVENT_ID) { listener: AnalyticsListener ->
      if (listener is ExtendedListener) {
        listener.onCustomEvent(eventTime)
      }
    }
  }
}

// 使用 - 设置和监听器注册。
val player = ExoPlayer.Builder(context).setAnalyticsCollector(ExtendedCollector()).build()
player.addAnalyticsListener(
  object : ExtendedListener {
    override fun onCustomEvent(eventTime: EventTime?) {
      // 保存自定义事件以用于分析数据。
    }
  }
)
// 使用 - 触发自定义事件。
(player.analyticsCollector as ExtendedCollector).customEvent()
```

### Java

```java
private interface ExtendedListener extends AnalyticsListener {
  void onCustomEvent(EventTime eventTime);
}

private static class ExtendedCollector extends DefaultAnalyticsCollector {
  public ExtendedCollector() {
    super(Clock.DEFAULT);
  }

  public void customEvent() {
    AnalyticsListener.EventTime eventTime = generateCurrentPlayerMediaPeriodEventTime();
    sendEvent(
        eventTime,
        CUSTOM_EVENT_ID,
        listener -> {
          if (listener instanceof ExtendedListener) {
            ((ExtendedListener) listener).onCustomEvent(eventTime);
          }
        });
  }
}

// 使用 - 设置和监听器注册。
ExoPlayer player =
    new ExoPlayer.Builder(context).setAnalyticsCollector(new ExtendedCollector()).build();
player.addAnalyticsListener(
    (ExtendedListener) eventTime -> {
      // 保存自定义事件以用于分析数据。
    });
// 使用 - 触发自定义事件。
((ExtendedCollector) player.getAnalyticsCollector()).customEvent();
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

