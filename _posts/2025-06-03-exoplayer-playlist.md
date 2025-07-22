---
title: ExoPlayer 从入门到精通（3）：播放列表
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-03 18:08:08 +0800
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


播放列表 API 由 `Player` 接口定义，该接口由所有 `ExoPlayer` 实现。播放列表使得多个媒体项可以按顺序播放。以下示例展示了如何开始播放包含两个视频的播放列表：

### Kotlin

```kotlin
// 构建媒体项。
val firstItem = MediaItem.fromUri(firstVideoUri)
val secondItem = MediaItem.fromUri(secondVideoUri)
// 添加要播放的媒体项。
player.addMediaItem(firstItem)
player.addMediaItem(secondItem)
// 准备播放器。
player.prepare()
// 开始播放。
player.play()
```

### Java

```java
// 构建媒体项。
MediaItem firstItem = MediaItem.fromUri(firstVideoUri);
MediaItem secondItem = MediaItem.fromUri(secondVideoUri);
// 添加要播放的媒体项。
player.addMediaItem(firstItem);
player.addMediaItem(secondItem);
// 准备播放器。
player.prepare();
// 开始播放。
player.play();
```

播放列表中各项之间的过渡是无缝的。它们不要求格式相同（例如，播放列表中可以同时包含 H264 和 VP9 视频）。它们甚至可以是不同类型的（即，播放列表中可以包含视频、图像和仅音频流）。你可以在播放列表中多次使用相同的 `MediaItem`。

## 修改播放列表

可以通过添加、移动、删除或替换媒体项来动态修改播放列表。这可以通过调用相应的播放列表 API 方法在播放前后完成：

### Kotlin

```kotlin
// 在播放列表的第 1 个位置添加一个媒体项。
player.addMediaItem(/* index= */ 1, MediaItem.fromUri(thirdUri))
// 将第三个媒体项从第 2 个位置移动到播放列表的开头。
player.moveMediaItem(/* currentIndex= */ 2, /* newIndex= */ 0)
// 从播放列表中移除第一个项。
player.removeMediaItem(/* index= */ 0)
// 替换播放列表中的第二个项。
player.replaceMediaItem(/* index= */ 1, MediaItem.fromUri(newUri))
```

### Java

```java
// 在播放列表的第 1 个位置添加一个媒体项。
player.addMediaItem(/* index= */ 1, MediaItem.fromUri(thirdUri));
// 将第三个媒体项从第 2 个位置移动到播放列表的开头。
player.moveMediaItem(/* currentIndex= */ 2, /* newIndex= */ 0);
// 从播放列表中移除第一个项。
player.removeMediaItem(/* index= */ 0);
// 替换播放列表中的第二个项。
player.replaceMediaItem(/* index= */ 1, MediaItem.fromUri(newUri));
```

还支持替换和清除整个播放列表：

### Kotlin

```kotlin
// 用一个新的播放列表替换当前的播放列表。
val newItems: List<MediaItem> = listOf(MediaItem.fromUri(fourthUri), MediaItem.fromUri(fifthUri))
player.setMediaItems(newItems, /* resetPosition= */ true)
// 清除播放列表。如果已准备好，播放器将转换到结束状态。
player.clearMediaItems()
```

### Java

```java
// 用一个新的播放列表替换当前的播放列表。
ImmutableList<MediaItem> newItems =
    ImmutableList.of(MediaItem.fromUri(fourthUri), MediaItem.fromUri(fifthUri));
player.setMediaItems(newItems, /* resetPosition= */ true);
// 清除播放列表。如果已准备好，播放器将转换到结束状态。
player.clearMediaItems();
```

播放器会自动在播放期间正确处理修改：

- 如果当前播放的 `MediaItem` 被移动，播放不会中断，完成时将播放其新的后继项。
- 如果当前播放的 `MediaItem` 被移除，播放器将自动播放剩余的第一个后继项，或者如果没有这样的后继项，则转换到结束状态。
- 如果当前播放的 `MediaItem` 被替换，如果 `MediaItem` 中与播放相关的属性未更改，则播放不会中断。例如，在大多数情况下，可以更新 `MediaItem.MediaMetadata` 字段而不影响播放。

## 查询播放列表

可以通过 `Player.getMediaItemCount` 和 `Player.getMediaItemAt` 查询播放列表。通过调用 `Player.getCurrentMediaItem` 可以查询当前播放的媒体项。还有其他一些便捷方法，如 `Player.hasNextMediaItem` 或 `Player.getNextMediaItemIndex`，以简化在播放列表中的导航。

## 重复模式

播放器支持 3 种重复模式，可以随时通过 `Player.setRepeatMode` 设置：

- `Player.REPEAT_MODE_OFF`：播放列表不会重复，播放器将在播放完播放列表中的最后一个项后转换到 `Player.STATE_ENDED`。
- `Player.REPEAT_MODE_ONE`：当前项会无限循环播放。像 `Player.seekToNextMediaItem` 这样的方法将忽略这一点并寻址到列表中的下一个项，然后该下一个项将无限循环播放。
- `Player.REPEAT_MODE_ALL`：整个播放列表将无限循环播放。

## 洗牌模式

可以随时通过 `Player.setShuffleModeEnabled` 启用或禁用洗牌模式。在洗牌模式下，播放器将以预计算的随机顺序播放播放列表。所有项都将播放一次，并且可以将洗牌模式与 `Player.REPEAT_MODE_ALL` 结合使用，以无限循环播放相同的随机顺序。当关闭洗牌模式时，播放从当前项在其原始播放列表位置继续进行。

请注意，像 `Player.getCurrentMediaItemIndex` 这样的方法返回的索引始终引用原始的、未洗牌的顺序。同样，`Player.seekToNextMediaItem` 将不会播放 `player.getCurrentMediaItemIndex() + 1` 处的项，而是根据洗牌顺序的下一个项。在播放列表中插入新项或移除项将尽可能保持现有的洗牌顺序不变。

### 设置自定义洗牌顺序

默认情况下，播放器通过使用 `DefaultShuffleOrder` 支持洗牌。可以通过提供自定义洗牌顺序实现，或在 `DefaultShuffleOrder` 构造函数中设置自定义顺序来定制：

### Kotlin

```kotlin
// 为播放列表中的 5 个项设置自定义洗牌顺序：
exoPlayer.setShuffleOrder(DefaultShuffleOrder(intArrayOf(3, 1, 0, 4, 2), randomSeed))
// 启用洗牌模式。
exoPlayer.shuffleModeEnabled = true
```

### Java

```java
// 为播放列表中的 5 个项设置自定义洗牌顺序：
exoPlayer.setShuffleOrder(new DefaultShuffleOrder(new int[] {3, 1, 0, 4, 2}, randomSeed));
// 启用洗牌模式。
exoPlayer.setShuffleModeEnabled(/* shuffleModeEnabled= */ true);
```

## 识别播放列表项

为了识别播放列表项，可以在构建项时设置 `MediaItem.mediaId`：

### Kotlin

```kotlin
// 构建一个带有媒体 ID 的媒体项。
val mediaItem = MediaItem.Builder().setUri(uri).setMediaId(mediaId).build()
```

### Java

```java
// 构建一个带有媒体 ID 的媒体项。
MediaItem mediaItem = new MediaItem.Builder().setUri(uri).setMediaId(mediaId).build();
```

如果应用未明确为媒体项定义媒体 ID，则使用 URI 的字符串表示形式。

## 将应用数据与播放列表项关联

除了 ID 之外，每个媒体项还可以配置自定义标签，可以是任何应用提供的对象。自定义标签的一个用途是将元数据附加到每个媒体项：

### Kotlin

```kotlin
// 构建一个带有自定义标签的媒体项。
val mediaItem = MediaItem.Builder().setUri(uri).setTag(metadata).build()
```

### Java

```java
// 构建一个带有自定义标签的媒体项。
MediaItem mediaItem = new MediaItem.Builder().setUri(uri).setTag(metadata).build();
```

## 检测播放转换到另一个媒体项时

当播放转换到另一个媒体项，或者开始重复同一个媒体项时，将调用 `Listener.onMediaItemTransition(MediaItem, @MediaItemTransitionReason)`。此回调接收新的媒体项，以及一个 `@MediaItemTransitionReason`，指示转换发生的原因。`onMediaItemTransition` 的一个常见用途是更新应用的 UI 以显示新的媒体项：

### Kotlin

```kotlin
override fun onMediaItemTransition(
  mediaItem: MediaItem?,
  @MediaItemTransitionReason reason: Int,
) {
  updateUiForPlayingMediaItem(mediaItem)
}
```

### Java

```java
@Override
public void onMediaItemTransition(
    @Nullable MediaItem mediaItem, @MediaItemTransitionReason int reason) {
  updateUiForPlayingMediaItem(mediaItem);
}
```

如果用于更新 UI 的元数据是使用自定义标签附加到每个媒体项的，那么实现可能如下所示：

### Kotlin

```kotlin
override fun onMediaItemTransition(
  mediaItem: MediaItem?,
  @MediaItemTransitionReason reason: Int,
) {
  var metadata: CustomMetadata? = null
  mediaItem?.localConfiguration?.let { localConfiguration ->
    metadata = localConfiguration.tag as? CustomMetadata
  }
  updateUiForPlayingMediaItem(metadata)
}
```

### Java

```java
@Override
public void onMediaItemTransition(
    @Nullable MediaItem mediaItem, @MediaItemTransitionReason int reason) {
  @Nullable CustomMetadata metadata = null;
  if (mediaItem != null && mediaItem.localConfiguration != null) {
    metadata = (CustomMetadata) mediaItem.localConfiguration.tag;
  }
  updateUiForPlayingMediaItem(metadata);
}
```

## 检测播放列表何时更改

当添加、移除或移动媒体项时，将立即调用 `Listener.onTimelineChanged(Timeline, @TimelineChangeReason)`，并带有 `TIMELINE_CHANGE_REASON_PLAYLIST_CHANGED`。即使播放器尚未准备好，此回调也会被调用。

### Kotlin

```kotlin
override fun onTimelineChanged(timeline: Timeline, @TimelineChangeReason reason: Int) {
  if (reason == Player.TIMELINE_CHANGE_REASON_PLAYLIST_CHANGED) {
    // 根据修改后的播放列表（添加、移动或移除）更新 UI。
    updateUiForPlaylist(timeline)
  }
}
```

### Java

```java
@Override
public void onTimelineChanged(Timeline timeline, @TimelineChangeReason int reason) {
  if (reason == TIMELINE_CHANGE_REASON_PLAYLIST_CHANGED) {
    // 根据修改后的播放列表（添加、移动或移除）更新 UI。
    updateUiForPlaylist(timeline);
  }
}
```

当播放列表中媒体项的持续时间等信息可用时，`Timeline` 将被更新，并且 `onTimelineChanged` 将被调用，带有 `TIMELINE_CHANGE_REASON_SOURCE_UPDATE`。其他可能导致时间线更新的原因包括：

- 在准备自适应媒体项后，清单变得可用。
- 在直播流播放期间，清单定期更新。

---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

