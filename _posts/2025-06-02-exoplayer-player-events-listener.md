---
title: ExoPlayer 从入门到精通（2）：播放器事件监听
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-02 18:08:08 +0800
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



## 监听播放事件

诸如播放状态变化和播放错误等事件，会报告给已注册的 `Player.Listener` 实例。若要注册一个监听器以接收此类事件：

### Kotlin

```kotlin
// 添加一个监听器以接收播放器的事件。
player.addListener(listener)
```

### Java

```java
// 添加一个监听器以接收播放器的事件。
player.addListener(listener);
```

`Player.Listener` 的默认方法均为空实现，因此你只需实现自己感兴趣的那些方法。可参阅 Javadoc 以获取方法的完整描述以及它们被调用的时机。下面对一些最重要的方法进行了更详细的说明。

监听器可以选择实现单独的事件回调，或者实现一个通用的 `onEvents` 回调，该回调会在一个或多个事件一起发生后被调用。关于不同使用场景下应优先选择哪种方式，可参阅 [单独回调与 onEvents](https://exoplayer.dev/listening-to-player-events.html#individual-callbacks-vs-onevents) 部分的解释。

### 播放状态变化

播放器状态的变化可以通过在已注册的 `Player.Listener` 中实现 `onPlaybackStateChanged(@State int state)` 来接收。播放器共有四种播放状态：

- `Player.STATE_IDLE`：这是初始状态，也是播放器停止或播放失败时的状态。在此状态下，播放器仅保留有限的资源。
- `Player.STATE_BUFFERING`：播放器无法从当前位置立即播放。这通常是因为需要加载更多数据。
- `Player.STATE_READY`：播放器能够从当前位置立即播放。
- `Player.STATE_ENDED`：播放器已播放完所有媒体。

除了这些状态外，播放器还有一个 `playWhenReady` 标志，用于表示用户希望播放的意图。可以通过实现 `onPlayWhenReadyChanged(playWhenReady, @PlayWhenReadyChangeReason int reason)` 来接收此标志的变化。

当同时满足以下三个条件时，播放器处于播放状态（即其位置在推进且媒体正在呈现给用户）：

- 播放器处于 `Player.STATE_READY` 状态
- `playWhenReady` 为 `true`
- 播放未因 `Player.getPlaybackSuppressionReason` 返回的原因而被抑制

与其逐一检查这些属性，不如直接调用 `Player.isPlaying`。通过实现 `onIsPlayingChanged(boolean isPlaying)` 可以接收此状态的变化：

### Kotlin

```kotlin
player.addListener(
  object : Player.Listener {
    override fun onIsPlayingChanged(isPlaying: Boolean) {
      if (isPlaying) {
        // 正在播放。
      } else {
        // 未播放，因为播放已暂停、结束、被抑制，或者播放器正在缓冲、停止或失败。请检查 player.playWhenReady、player.playbackState、player.playbackSuppressionReason 和 player.playerError 以获取详细信息。
      }
    }
  }
)
```

### Java

```java
player.addListener(
    new Player.Listener() {
      @Override
      public void onIsPlayingChanged(boolean isPlaying) {
        if (isPlaying) {
          // 正在播放。
        } else {
          // 未播放，因为播放已暂停、结束、被抑制，或者播放器正在缓冲、停止或失败。请检查 player.getPlayWhenReady()、player.getPlaybackState()、player.getPlaybackSuppressionReason() 和 player.getPlaybackError() 以获取详细信息。
        }
      }
    });
```

### 播放错误

导致播放失败的错误可以通过在已注册的 `Player.Listener` 中实现 `onPlayerError(PlaybackException error)` 来接收。当发生失败时，此方法将在播放状态转换为 `Player.STATE_IDLE` 之前立即被调用。可以通过调用 `ExoPlayer.prepare` 来重试失败或停止的播放。

需注意，某些 `Player` 实现会将 `PlaybackException` 的子类实例传递给监听器，以提供有关失败的更多详细信息。例如，`ExoPlayer` 会传递 `ExoPlaybackException`，其中包含 `type`、`rendererIndex` 以及其他特定于 ExoPlayer 的字段。

以下示例展示了如何检测由于 HTTP 网络问题导致的播放失败：

### Kotlin

```kotlin
player.addListener(
  object : Player.Listener {
    override fun onPlayerError(error: PlaybackException) {
      val cause = error.cause
      if (cause is HttpDataSourceException) {
        // 发生了 HTTP 错误。
        val httpError = cause
        // 既可以通过强制转换，也可以通过查询 cause 来获取更多有关错误的信息。
        if (httpError is InvalidResponseCodeException) {
          // 强制转换为 InvalidResponseCodeException 并获取响应代码、消息和标题。
        } else {
          // 尝试调用 httpError.getCause() 来获取根本原因，但请注意它可能为 null。
        }
      }
    }
  }
)
```

### Java

```java
player.addListener(
    new Player.Listener() {
      @Override
      public void onPlayerError(PlaybackException error) {
        @Nullable Throwable cause = error.getCause();
        if (cause instanceof HttpDataSourceException) {
          // 发生了 HTTP 错误。
          HttpDataSourceException httpError = (HttpDataSourceException) cause;
          // 既可以通过强制转换，也可以通过查询 cause 来获取更多有关错误的信息。
          if (httpError instanceof HttpDataSource.InvalidResponseCodeException) {
            // 强制转换为 InvalidResponseCodeException 并获取响应代码、消息和标题。
          } else {
            // 尝试调用 httpError.getCause() 来获取根本原因，但请注意它可能为 null。
          }
        }
      }
    });
```

### 播放列表切换

每当播放器在播放列表中切换到新的媒体项时，已注册的 `Player.Listener` 对象的 `onMediaItemTransition(MediaItem mediaItem, @MediaItemTransitionReason int reason)` 方法会被调用。reason 参数表明这是自动切换、寻址（例如调用 player.next() 后）、重复播放同一项，还是由于播放列表更改（例如当前播放项被移除）导致的。

### 元数据

由 `player.getCurrentMediaMetadata()` 返回的元数据可能会因多种原因而变化：播放列表切换、流内元数据更新，或在播放过程中更新当前的 `MediaItem`。

如果您对元数据变化感兴趣，例如为了更新显示当前标题的 UI，可以监听 `onMediaMetadataChanged`。

### 寻址

调用 `Player.seekTo` 方法会导致一系列回调被发送到已注册的 `Player.Listener` 实例：

1. `onPositionDiscontinuity`，其 reason 参数为 DISCONTINUITY_REASON_SEEK。这是调用 `Player.seekTo` 的直接结果。回调包含寻址前后的位置信息。
2. `onPlaybackStateChanged`，其带有与寻址相关的任何即时状态变化。需注意，可能并无此类变化。

### 单独回调与 onEvents

监听器可以选择实现单独的回调（如 `onIsPlayingChanged(boolean isPlaying)`）或通用的 `onEvents(Player player, Events events)` 回调。通用回调提供了对 `Player` 对象的访问，并指定了同时发生的事件集。此回调总是在对应于各个事件的回调之后被调用。

### Kotlin

```kotlin
override fun onEvents(player: Player, events: Player.Events) {
  if (
    events.contains(Player.EVENT_PLAYBACK_STATE_CHANGED) ||
      events.contains(Player.EVENT_PLAY_WHEN_READY_CHANGED)
  ) {
    uiModule.updateUi(player)
  }
}
```

### Java

```java
@Override
public void onEvents(Player player, Events events) {
  if (events.contains(Player.EVENT_PLAYBACK_STATE_CHANGED)
      || events.contains(Player.EVENT_PLAY_WHEN_READY_CHANGED)) {
    uiModule.updateUi(player);
  }
}
```

在以下情况下应优先使用单独的事件：

- 监听器对变化的原因感兴趣。例如，`onPlayWhenReadyChanged` 或 `onMediaItemTransition` 提供的原因。
- 监听器仅根据回调参数中的新值采取行动，或触发不依赖于回调参数的其他操作。
- 监听器实现更倾向于在方法名称中明确指示触发事件的可读性。
- 监听器向分析系统报告需要了解所有单独事件和状态变化的情况。

在以下情况下应优先使用通用的 `onEvents(Player player, Events events)`：

- 监听器希望对多个事件触发相同的逻辑。例如，为 `onPlaybackStateChanged` 和 `onPlayWhenReadyChanged` 更新 UI。
- 监听器需要访问 `Player` 对象以触发进一步的事件，例如在媒体项切换后进行寻址。
- 监听器打算一起使用多个状态值，这些值通过单独的回调报告，或与 `Player` 的 getter 方法结合使用。例如，在 `onTimelineChanged` 中使用 `Player.getCurrentWindowIndex()` 与 `Timeline` 一起使用，这在 `onEvents` 回调内部才是安全的。
- 监听器对事件是否逻辑上同时发生感兴趣。例如，由于媒体项切换导致的 `onPlaybackStateChanged` 到 `STATE_BUFFERING`。

在某些情况下，监听器可能需要将单独的回调与通用的 `onEvents` 回调结合使用，例如记录 `onMediaItemTransition` 的媒体项变化原因，但仅在所有状态变化可以一起在 `onEvents` 中使用时才采取行动。

## 使用 AnalyticsListener

当使用 `ExoPlayer` 时，可以通过调用 `addAnalyticsListener` 向播放器注册 `AnalyticsListener`。`AnalyticsListener` 实现能够监听对于分析和日志记录目的可能有用的详细事件。请参阅分析页面以获取更多详细信息。

### 使用 EventLogger

`EventLogger` 是库直接提供的一个 `AnalyticsListener`，用于日志记录目的。通过一行代码将 `EventLogger` 添加到 `ExoPlayer`，即可启用有用的额外日志记录：

### Kotlin

```kotlin
player.addAnalyticsListener(EventLogger())
```

### Java

```java
player.addAnalyticsListener(new EventLogger());
```

请参阅调试日志页面以获取更多详细信息。

## 在指定播放位置触发事件

某些使用场景需要在指定的播放位置触发事件。这可以通过 `PlayerMessage` 来实现。`PlayerMessage` 可以使用 `ExoPlayer.createMessage` 创建。可以使用 `PlayerMessage.setPosition` 设置其应执行的播放位置。默认情况下，消息在播放线程上执行，但可以使用 `PlayerMessage.setLooper` 自定义。`PlayerMessage.setDeleteAfterDelivery` 可用于控制消息是否在每次遇到指定的播放位置时执行（由于寻址和重复模式，这可能会发生多次），或者仅第一次遇到时执行。一旦 `PlayerMessage` 配置完成，可以使用 `PlayerMessage.send` 进行调度。

### Kotlin

```kotlin
player
  .createMessage { messageType: Int, payload: Any? -> }
  .setLooper(Looper.getMainLooper())
  .setPosition(/* mediaItemIndex= */ 0, /* positionMs= */ 120000)
  .setPayload(customPayloadData)
  .setDeleteAfterDelivery(false)
  .send()
```

### Java

```java
player
    .createMessage(
        (messageType, payload) -> {
          // 在指定的播放位置执行操作。
        })
    .setLooper(Looper.getMainLooper())
    .setPosition(/* mediaItemIndex= */ 0, /* positionMs= */ 120_000)
    .setPayload(customPayloadData)
    .setDeleteAfterDelivery(false)
    .send();
```

---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

