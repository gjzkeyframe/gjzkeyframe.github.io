---
title: ExoPlayer 从入门到精通（1）：ExoPlayer 入门指南
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-01 18:08:08 +0800
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





使用集合保持条理井然根据您的喜好保存和分类内容。对于简单的使用场景，使用 `ExoPlayer` 入门包括实现以下步骤：

1. 将 ExoPlayer 添加到您的项目中作为依赖项。
2. 创建一个 `ExoPlayer` 实例。
3. 将播放器附加到一个视图（用于视频输出和用户输入）。
4. 使用 `MediaItem` 准备播放器进行播放。
5. 播放完成后释放播放器。

下面更详细地描述了这些步骤。有关完整示例，请参阅主演示应用中的 `PlayerActivity`。

## 将 ExoPlayer 添加为依赖项

### 添加 ExoPlayer 模块

使用 AndroidX Media3 入门的最简单方法是在应用模块的 `build.gradle` 文件中添加对所需库的 Gradle 依赖项。

例如，要依赖支持 DASH 播放的 ExoPlayer 和 UI 组件，可以添加对以下模块的依赖项：

### Kotlin

```
implementation("androidx.media3:media3-exoplayer:1.5.1")
implementation("androidx.media3:media3-exoplayer-dash:1.5.1")
implementation("androidx.media3:media3-ui:1.5.1")
```

### Groovy

```
implementation "androidx.media3:media3-exoplayer:1.5.1"
implementation "androidx.media3:media3-exoplayer-dash:1.5.1"
implementation "androidx.media3:media3-ui:1.5.1"
```

其中 1.5.1 是您首选的版本（最新版本可以通过查看发行说明来找到）。所有模块必须是同一版本。

AndroidX Media3 有依赖于外部库以提供额外功能的库模块。有些可以从 Maven 存储库中获取，而有些则必须手动构建。浏览库目录并查看各个 README 文件以了解详细信息。

有关可用库模块的更多信息，请访问 Google Maven AndroidX Media 页面。

### 启用 Java 8 支持

如果尚未启用，您需要在所有依赖于 ExoPlayer 的 `build.gradle` 文件中启用 Java 8 支持，通过在 `android` 部分添加以下内容：

```
compileOptions {
  targetCompatibility JavaVersion.VERSION_1_8
}

```

## 创建播放器

您可以使用 `ExoPlayer.Builder` 创建一个 `ExoPlayer` 实例，它提供了一系列自定义选项。以下代码是最简单的创建实例的示例。

### Kotlin

```
val player = ExoPlayer.Builder(context).build()
```

### Java

```
ExoPlayer player = new ExoPlayer.Builder(context).build();
```

### 关于线程的说明

ExoPlayer 实例必须从单个应用程序线程访问。在大多数情况下，这应该是应用程序的主线程。在使用 ExoPlayer 的 UI 组件或 IMA 扩展时，使用应用程序的主线程是必需的。

可以在线程上明确指定 ExoPlayer 实例必须访问的 `Looper`，通过在创建播放器时传递一个 `Looper`。如果没有指定 `Looper`，则使用创建播放器的线程的 `Looper`，或者如果该线程没有 `Looper`，则使用应用程序主线程的 `Looper`。在所有情况下，可以通过 `Player.getApplicationLooper` 查询播放器必须访问的线程的 `Looper`。

有关 ExoPlayer 线程模型的更多信息，请参阅 ExoPlayer Javadoc 中的"线程模型"部分。

## 将播放器附加到视图

ExoPlayer 库提供了多种预构建的 UI 组件用于媒体播放。这些组件包括 `PlayerView`，它封装了一个 `PlayerControlView`、一个 `SubtitleView` 和一个用于渲染视频的 `Surface`。`PlayerView` 可以包含在应用的布局 XML 中。例如，要将播放器绑定到视图：

### Kotlin

```
// 将播放器绑定到视图。
playerView.player = player
```

### Java

```
// 将播放器绑定到视图。
playerView.setPlayer(player);
```

您也可以将 `PlayerControlView` 作为独立组件使用，这对于仅音频的场景非常有用。

使用 ExoPlayer 的预构建 UI 组件是可选的。对于实现自定义 UI 的视频应用，可以分别使用 ExoPlayer 的 `setVideoSurfaceView`、`setVideoTextureView`、`setVideoSurfaceHolder` 和 `setVideoSurface` 方法设置目标 `SurfaceView`、`TextureView`、`SurfaceHolder` 或 `Surface`。ExoPlayer 的 `addTextOutput` 方法可用于接收在播放期间应渲染的字幕。

## 填充播放列表并准备播放器

在 ExoPlayer 中，每个媒体文件由一个 `MediaItem` 表示。要播放媒体文件，需要构建相应的 `MediaItem`，将其添加到播放器，准备播放器，然后调用 `play` 开始播放：

### Kotlin

```
// 构建媒体项。
val mediaItem = MediaItem.fromUri(videoUri)
// 设置要播放的媒体项。
player.setMediaItem(mediaItem)
// 准备播放器。
player.prepare()
// 开始播放。
player.play()
```

### Java

```
// 构建媒体项。
MediaItem mediaItem = MediaItem.fromUri(videoUri);
// 设置要播放的媒体项。
player.setMediaItem(mediaItem);
// 准备播放器。
player.prepare();
// 开始播放。
player.play();
```

ExoPlayer 直接支持播放列表，因此可以准备播放器以播放多个媒体项，这些媒体项将依次播放：

### Kotlin

```
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

```
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

可以在播放过程中更新播放列表，而无需再次准备播放器。有关填充和操作播放列表的更多信息，请参阅播放列表页面。有关构建媒体项时可用的不同选项的更多信息，例如剪辑和附加字幕文件，请参阅媒体项页面。

## 控制播放器

一旦播放器准备就绪，就可以通过调用播放器上的方法来控制播放。以下是一些最常用的方法：

- `play` 和 `pause` 分别开始和暂停播放。
- `seekTo` 允许在媒体中进行寻址。
- `hasPrevious`、`hasNext`、`previous` 和 `next` 允许在播放列表中导航。
- `setRepeatMode` 控制媒体是否以及如何循环播放。
- `setShuffleModeEnabled` 控制播放列表的洗牌模式。
- `setPlaybackParameters` 调整播放速度和音频音高。

如果播放器绑定到 `PlayerView` 或 `PlayerControlView`，则用户与这些组件的交互将导致播放器上相应方法被调用。


## 释放播放器


当不再需要播放器时，释放它很重要，这样可以释放有限的资源（如视频解码器）供其他应用程序使用。这可以通过调用 来完成 `ExoPlayer.release`。

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

