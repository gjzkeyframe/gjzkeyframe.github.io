---
title: ExoPlayer 从入门到精通（6）：轨道选择
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-06 18:08:08 +0800
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




当媒体项包含多个轨道时，轨道选择是确定其中哪些轨道被选中进行播放的过程。轨道选择过程由 `TrackSelectionParameters` 配置，该参数允许指定许多不同的约束和覆盖，从而影响轨道选择。

## 查询可用轨道

您可以监听 `Player.Listener.onTracksChanged` 以获取轨道更改的通知，包括：

- 当播放的媒体项准备完成时，可用轨道变得已知。请注意，播放器需要准备媒体项才能知道它包含哪些轨道。
- 由于播放从一个媒体项过渡到另一个媒体项，可用轨道发生变化。
- 选定轨道的更改。

### Kotlin

```kotlin
player.addListener(
  object : Player.Listener {
    override fun onTracksChanged(tracks: Tracks) {
      // 使用当前轨道更新 UI。
    }
  }
)
```

### Java

```java
player.addListener(
    new Player.Listener() {
      @Override
      public void onTracksChanged(Tracks tracks) {
        // 使用当前轨道更新 UI。
      }
    });
```

您还可以通过调用 `player.getCurrentTracks()` 查询当前轨道。返回的 `Tracks` 包含一个 `Track.Group` 对象列表，其中单个 `Group` 中的轨道呈现相同内容但格式不同。

例如，在自适应播放中，主视频流以五种比特率提供，另一种视频流（例如体育比赛中的不同摄像机角度）以两种比特率提供。这种情况下，将有两个视频轨道组，一个对应主视频流，包含五个轨道，另一个对应替代视频流，包含两个轨道。

不同语言的音频轨道不会被分组，因为不同语言的内容不被视为相同。相反，同一语言的音频轨道如果在比特率、采样率、声道数等属性上有所不同，可以被分组。文本轨道也是如此。

每个 `Group` 可以查询以确定哪些轨道支持播放，哪些当前被选中，以及每个轨道使用什么 `Format`：

### Kotlin

```kotlin
for (trackGroup in tracks.groups) {
  // 组级别信息。
  val trackType = trackGroup.type
  val trackInGroupIsSelected = trackGroup.isSelected
  val trackInGroupIsSupported = trackGroup.isSupported
  for (i in 0 until trackGroup.length) {
    // 单个轨道信息。
    val isSupported = trackGroup.isTrackSupported(i)
    val isSelected = trackGroup.isTrackSelected(i)
    val trackFormat = trackGroup.getTrackFormat(i)
  }
}
```

### Java

```java
for (Tracks.Group trackGroup : tracks.getGroups()) {
  // 组级别信息。
  @C.TrackType int trackType = trackGroup.getType();
  boolean trackInGroupIsSelected = trackGroup.isSelected();
  boolean trackInGroupIsSupported = trackGroup.isSupported();
  for (int i = 0; i < trackGroup.length; i++) {
    // 单个轨道信息。
    boolean isSupported = trackGroup.isTrackSupported(i);
    boolean isSelected = trackGroup.isTrackSelected(i);
    Format trackFormat = trackGroup.getTrackFormat(i);
  }
}
```

- 如果 `Player` 能够解码并呈现轨道的样本，则该轨道是受支持的。请注意，即使多个相同类型的轨道组（例如多个音频轨道组）受支持，这也仅表示它们可以单独受支持，播放器不一定能够同时播放它们。
- 如果根据当前 `TrackSelectionParameters` 选择了轨道进行播放，则该轨道是被选中的。如果一个轨道组中有多个轨道被选中，播放器会使用这些轨道进行自适应播放（例如，具有不同比特率的多个视频轨道）。请注意，任何时候只有一个轨道会播放。

## 修改轨道选择参数

可以使用 `Player.setTrackSelectionParameters` 配置轨道选择过程，这可以在播放前后进行。以下示例演示了如何从播放器获取当前 `TrackSelectionParameters`，修改它们，然后用修改后的结果更新 `Player`：

### Kotlin

```kotlin
player.trackSelectionParameters =
  player.trackSelectionParameters
    .buildUpon()
    .setMaxVideoSizeSd()
    .setPreferredAudioLanguage("hu")
    .build()
```

### Java

```java
player.setTrackSelectionParameters(
    player
        .getTrackSelectionParameters()
        .buildUpon()
        .setMaxVideoSizeSd()
        .setPreferredAudioLanguage("hu")
        .build());
```

### 基于约束的轨道选择

`TrackSelectionParameters` 中的大多数选项允许您指定与实际可用轨道无关的约束。可用约束包括：

- 视频的最大和最小宽度、高度、帧率和比特率。
- 音频的最大声道数和比特率。
- 视频和音频的首选 MIME 类型。
- 音频的首选语言和角色标志。
- 文本的首选语言和角色标志。

ExoPlayer 为这些约束提供了合理的默认值，例如将视频分辨率限制为显示尺寸，并优先选择与用户系统区域设置匹配的音频语言。

与从可用轨道中选择特定轨道相比，使用基于约束的轨道选择有以下几项优势：

- 您可以在不知道媒体项提供哪些轨道的情况下指定约束。这意味着可以在播放器准备媒体项之前指定约束，而选择特定轨道需要应用程序代码等待直到可用轨道已知。
- 约束适用于播放列表中的所有媒体项，即使这些项具有不同的可用轨道。例如，首选音频语言约束将自动应用于所有媒体项，即使该语言的轨道格式在不同媒体项之间有所不同。而选择特定轨道则不适用，如下所述。

### 选择特定轨道

可以使用 `TrackSelectionParameters` 选择特定轨道。首先，使用 `Player.getCurrentTracks` 查询播放器当前可用的轨道。其次，在确定要选择的轨道后，可以使用 `TrackSelectionOverride` 将其设置到 `TrackSelectionParameters` 上。例如，选择特定 `audioTrackGroup` 中的第一个轨道：

### Kotlin

```kotlin
player.trackSelectionParameters =
  player.trackSelectionParameters
    .buildUpon()
    .setOverrideForType(
      TrackSelectionOverride(audioTrackGroup.mediaTrackGroup, /* trackIndex= */ 0)
    )
    .build()
```

### Java

```java
player.setTrackSelectionParameters(
    player
        .getTrackSelectionParameters()
        .buildUpon()
        .setOverrideForType(
            new TrackSelectionOverride(
                audioTrackGroup.getMediaTrackGroup(), /* trackIndex= */ 0))
        .build());
```

只有在媒体项包含与覆盖中指定的 `TrackGroup` 完全匹配的轨道时，`TrackSelectionOverride` 才会应用。因此，如果后续媒体项包含不同的轨道，则覆盖可能不适用。

### 禁用轨道类型或组

可以使用 `TrackSelectionParameters.Builder.setTrackTypeDisabled` 完全禁用视频、音频或文本等轨道类型。禁用的轨道类型将适用于所有媒体项：

### Kotlin

```kotlin
player.trackSelectionParameters =
  player.trackSelectionParameters
    .buildUpon()
    .setTrackTypeDisabled(C.TRACK_TYPE_VIDEO, /* disabled= */ true)
    .build()
```

### Java

```java
player.setTrackSelectionParameters(
    player
        .getTrackSelectionParameters()
        .buildUpon()
        .setTrackTypeDisabled(C.TRACK_TYPE_VIDEO, /* disabled= */ true)
        .build());
```

或者，可以通过为特定 `TrackGroup` 指定空覆盖来阻止从该组中选择轨道：

### Kotlin

```kotlin
player.trackSelectionParameters =
  player.trackSelectionParameters
    .buildUpon()
    .addOverride(
      TrackSelectionOverride(disabledTrackGroup.mediaTrackGroup, /* trackIndices= */ listOf())
    )
    .build()
```

### Java

```java
player.setTrackSelectionParameters(
    player
        .getTrackSelectionParameters()
        .buildUpon()
        .addOverride(
            new TrackSelectionOverride(
                disabledTrackGroup.getMediaTrackGroup(),
                /* trackIndices= */ ImmutableList.of()))
        .build());
```

## 自定义轨道选择器

轨道选择是 `TrackSelector` 的责任，可以在构建 `ExoPlayer` 时提供其实例，之后可以通过 `ExoPlayer.getTrackSelector()` 获取。

### Kotlin

```kotlin
val trackSelector = DefaultTrackSelector(context)
val player = ExoPlayer.Builder(context).setTrackSelector(trackSelector).build()
```

### Java

```java
DefaultTrackSelector trackSelector = new DefaultTrackSelector(context);
ExoPlayer player = new ExoPlayer.Builder(context).setTrackSelector(trackSelector).build();
```

`DefaultTrackSelector` 是一个灵活的 `TrackSelector`，适用于大多数使用场景。它使用 `Player` 中设置的 `TrackSelectionParameters`，还提供了一些高级自定义选项，这些选项可以在 `DefaultTrackSelector.ParametersBuilder` 中指定：

### Kotlin

```kotlin
trackSelector.setParameters(
  trackSelector.buildUponParameters().setAllowVideoMixedMimeTypeAdaptiveness(true))
)
```

### Java

```java
trackSelector.setParameters(
    trackSelector.buildUponParameters().setAllowVideoMixedMimeTypeAdaptiveness(true));
```

### 隧道传输

在渲染器和选定轨道的组合支持的情况下，可以使用 `DefaultTrackSelector.ParametersBuilder.setTunnelingEnabled(true)` 启用隧道播放。

### 音频卸载

在渲染器和选定轨道的组合支持的情况下，可以通过在 `TrackSelectionParameters` 中指定 `AudioOffloadModePreferences` 来启用卸载音频播放。

### Kotlin

```kotlin
val audioOffloadPreferences =
  AudioOffloadPreferences.Builder()
      .setAudioOffloadMode(AudioOffloadPreferences.AUDIO_OFFLOAD_MODE_ENABLED)
      // 根据需要添加其他选项
      .setIsGaplessSupportRequired(true)
      .build()
player.trackSelectionParameters =
  player.trackSelectionParameters
    .buildUpon()
    .setAudioOffloadPreferences(audioOffloadPreferences)
    .build()
```

### Java

```java
AudioOffloadPreferences audioOffloadPreferences =
  new AudioOffloadPreferences.Builder()
      .setAudioOffloadMode(AudioOffloadPreferences.AUDIO_OFFLOAD_MODE_ENABLED)
      // 根据需要添加其他选项
      .setIsGaplessSupportRequired(true)
      .build();
player.setTrackSelectionParameters(
  player.getTrackSelectionParameters()
    .buildUpon()
    .setAudioOffloadPreferences(audioOffloadPreferences)
    .build());
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

