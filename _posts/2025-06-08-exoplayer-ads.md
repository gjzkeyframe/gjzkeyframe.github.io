---
title: ExoPlayer 从入门到精通（8）： 插入广告
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-08 18:08:08 +0800
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


ExoPlayer 支持客户端和服务器端的广告插入。

## 客户端广告插入

在客户端广告插入中，播放器在播放内容和广告之间切换时，会从不同的网址加载媒体。广告相关信息（如 XML VAST 或 VMAP 广告代码）与媒体分开加载，可能包括广告在内容开始位置的提示、实际广告媒体 URI 以及元数据（如广告是否可跳过）。

使用 ExoPlayer 的 `AdsMediaSource` 进行客户端广告插入时，播放器会管理要播放的广告信息，带来以下好处：

- 播放器可以通过其 API 提供与广告相关的元数据和功能。
- ExoPlayer 界面组件可以自动显示广告位置的标记，并根据广告是否在播放更改其行为。
- 内部播放器可以在广告和内容之间切换时保持一致的缓冲区。

在此设置中，播放器负责在广告和内容之间切换，应用无需针对广告和内容控制多个单独的后台/前台播放器。

在准备内容视频和广告代码以便与客户端广告插入搭配使用时，广告最好位于内容视频中的同步样本（关键帧）处，以便播放器能够顺畅地继续播放内容。

构建 `MediaItem` 时，可以指定广告代码 URI：

### Kotlin

```kotlin
val mediaItem =
  MediaItem.Builder()
    .setUri(videoUri)
    .setAdsConfiguration(MediaItem.AdsConfiguration.Builder(adTagUri).build())
    .build()
```

### Java

```java
MediaItem mediaItem =
    new MediaItem.Builder()
        .setUri(videoUri)
        .setAdsConfiguration(
            new MediaItem.AdsConfiguration.Builder(adTagUri).build())
        .build();
```

如需为指定广告代码的媒体内容启用播放器支持，必须在创建播放器时构建并注入配置了 `AdsLoader.Provider` 和 `AdViewProvider` 的 `DefaultMediaSourceFactory`：

### Kotlin

```kotlin
val mediaSourceFactory: MediaSource.Factory =
  DefaultMediaSourceFactory(context).setLocalAdInsertionComponents(adsLoaderProvider, playerView)
val player = ExoPlayer.Builder(context).setMediaSourceFactory(mediaSourceFactory).build()
```

### Java

```java
MediaSource.Factory mediaSourceFactory =
    new DefaultMediaSourceFactory(context)
        .setLocalAdInsertionComponents(adsLoaderProvider, /* adViewProvider= */ playerView);
ExoPlayer player =
    new ExoPlayer.Builder(context).setMediaSourceFactory(mediaSourceFactory).build();
```

内部，`DefaultMediaSourceFactory` 会将内容媒体源封装在 `AdsMediaSource` 中。`AdsMediaSource` 将从 `AdsLoader.Provider` 获取 `AdsLoader`，并使用它根据媒体内容的广告代码插入广告。

ExoPlayer 的 `PlayerView` 实现了 `AdViewProvider`。ExoPlayer IMA 库提供了易于使用的 `AdsLoader`，如下所述。

### 含广告的播放列表

播放包含多个媒体项的播放列表时，默认行为是为每个媒体 ID、内容 URI 和广告代码 URI 组合请求广告代码并存储一次广告播放状态。这意味着，对于具有不同媒体 ID 或内容 URI 的广告，用户会看到每个媒体项的广告，即使广告代码 URI 匹配也是如此。如果媒体内容重复播放，用户只会看到相应的广告一次（广告播放状态会存储广告是否已播放，因此广告会在首次出现后被跳过）。

可以通过传递不透明的广告标识符（与给定媒体内容的广告播放状态相关联，基于对象等式）来自定义此行为。下面的示例中，广告播放状态仅与广告代码 URI 相关联（而非媒体 ID 和广告代码 URI 的组合），因为广告代码 URI 被作为广告标识符传递。这样一来，广告只会加载一次，并且当用户从头到尾播放播放列表时，不会在第二个内容中看到广告。

### Kotlin

```kotlin
// 构建媒体项，为两个项传递相同的广告标识符，这意味着它们共享广告播放状态，因此广告只播放一次。
val firstItem =
  MediaItem.Builder()
    .setUri(firstVideoUri)
    .setAdsConfiguration(MediaItem.AdsConfiguration.Builder(adTagUri).setAdsId(adTagUri).build())
    .build()
val secondItem =
  MediaItem.Builder()
    .setUri(secondVideoUri)
    .setAdsConfiguration(MediaItem.AdsConfiguration.Builder(adTagUri).setAdsId(adTagUri).build())
    .build()
player.addMediaItem(firstItem)
player.addMediaItem(secondItem)
```

### Java

```java
// 构建媒体项，为两个项传递相同的广告标识符，这意味着它们共享广告播放状态，因此广告只播放一次。
MediaItem firstItem =
    new MediaItem.Builder()
        .setUri(firstVideoUri)
        .setAdsConfiguration(
            new MediaItem.AdsConfiguration.Builder(adTagUri).setAdsId(adTagUri).build())
        .build();
MediaItem secondItem =
    new MediaItem.Builder()
        .setUri(secondVideoUri)
        .setAdsConfiguration(
            new MediaItem.AdsConfiguration.Builder(adTagUri).setAdsId(adTagUri).build())
        .build();
player.addMediaItem(firstItem);
player.addMediaItem(secondItem);
```

### ExoPlayer IMA 库

ExoPlayer IMA 库提供了 `ImaAdsLoader`，可让您轻松地将客户端广告插入集成到您的应用中。它封装了客户端 IMA SDK 的功能，以支持插入 VAST/VMAP 广告。如需了解如何使用该库（包括如何处理后台播放和恢复播放），请参阅自述文件。

演示版应用使用 IMA 库，并在示例列表中包含多个 VAST/VMAP 广告代码示例。

#### 界面注意事项

默认情况下，`PlayerView` 会在广告播放期间隐藏其传输控件，但应用可以通过调用 `setControllerHideDuringAds` 来切换此行为。在广告播放期间，IMA SDK 会在播放器上方显示其他视图（例如“更多信息”链接和跳过按钮，如果适用）。

IMA SDK 可能会报告广告是否因在播放器上呈现的应用提供的视图被遮挡。如果应用需要叠加对控制播放至关重要的视图，则必须在 IMA SDK 中注册这些视图，以便在计算可见度时将其忽略。使用 `PlayerView` 作为 `AdViewProvider` 时，它会自动注册其控件叠加层。使用自定义播放器界面的应用必须通过从 `AdViewProvider.getAdOverlayInfos` 返回叠加层视图来注册叠加层视图。

如需详细了解叠加视图，请参阅 IMA SDK 中的 Open Measurement。

#### 随播广告

某些广告代码包含可以在应用界面的“广告位”中展示的其他随播广告。这些槽可通过 `ImaAdsLoader.Builder.setCompanionAdSlots(slots)` 传递。如需了解详情，请参阅添加随播广告。

#### 独立广告

IMA SDK 旨在将广告插入媒体内容中，而不是单独播放广告。因此，IMA 库不支持播放独立广告。对于此用例，我们推荐改用 Google 移动广告 SDK。

### 使用第三方广告 SDK

如果您需要通过第三方广告 SDK 加载广告，则有必要检查它是否已提供 ExoPlayer 集成。如果没有，建议实现封装第三方广告 SDK 的自定义 `AdsLoader`，因为它具有上述 `AdsMediaSource` 的优势。`ImaAdsLoader` 用作实现示例。

或者，您也可以使用 ExoPlayer 的播放列表支持功能来构建一系列广告和内容片段：

### Kotlin

```kotlin
// 开头广告。
val preRollAd = MediaItem.fromUri(preRollAdUri)
// 内容的开始部分。
val contentStart =
  MediaItem.Builder()
    .setUri(contentUri)
    .setClippingConfiguration(ClippingConfiguration.Builder().setEndPositionMs(120000).build())
    .build()
// 中间广告。
val midRollAd = MediaItem.fromUri(midRollAdUri)
// 内容的剩余部分。
val contentEnd =
  MediaItem.Builder()
    .setUri(contentUri)
    .setClippingConfiguration(ClippingConfiguration.Builder().setStartPositionMs(120000).build())
    .build()

// 构建播放列表。
player.addMediaItem(preRollAd)
player.addMediaItem(contentStart)
player.addMediaItem(midRollAd)
player.addMediaItem(contentEnd)
```

### Java

```java
// 开头广告。
MediaItem preRollAd = MediaItem.fromUri(preRollAdUri);
// 内容的开始部分。
MediaItem contentStart =
    new MediaItem.Builder()
        .setUri(contentUri)
        .setClippingConfiguration(
            new ClippingConfiguration.Builder().setEndPositionMs(120_000).build())
        .build();
// 中间广告。
MediaItem midRollAd = MediaItem.fromUri(midRollAdUri);
// 内容的剩余部分。
MediaItem contentEnd =
    new MediaItem.Builder()
        .setUri(contentUri)
        .setClippingConfiguration(
            new ClippingConfiguration.Builder().setStartPositionMs(120_000).build())
        .build();

// 构建播放列表。
player.addMediaItem(preRollAd);
player.addMediaItem(contentStart);
player.addMediaItem(midRollAd);
player.addMediaItem(contentEnd);
```

## 服务器端广告插播

在服务器端广告插播（也称为动态广告插播 [DAI]）中，媒体串流同时包含广告和内容。DASH 清单可以同时指向内容和广告片段，可能位于不同的时段。对于 HLS，请参阅有关将广告添加到播放列表的 Apple 文档。

使用服务器端广告插入时，客户端可能需要动态解析媒体网址以获取拼接的直播，可能需要在界面中显示广告叠加层，或者可能需要向广告 SDK 或广告服务器报告事件。

对于使用 `ssai://` 架构的 URI，ExoPlayer 的 `DefaultMediaSourceFactory` 可以将所有这些任务委托给服务器端广告插入 `MediaSource`：

### Kotlin

```kotlin
val player =
  ExoPlayer.Builder(context)
    .setMediaSourceFactory(
      DefaultMediaSourceFactory(context).setServerSideAdInsertionMediaSourceFactory(ssaiFactory)
    )
    .build()
```

### Java

```java
Player player =
    new ExoPlayer.Builder(context)
        .setMediaSourceFactory(
            new DefaultMediaSourceFactory(context)
                .setServerSideAdInsertionMediaSourceFactory(ssaiFactory))
        .build();
```

### ExoPlayer IMA 库

ExoPlayer IMA 库提供 `ImaServerSideAdInsertionMediaSource`，可让您轻松在应用中将 IMA 服务器端插入的广告流集成在一起。它封装了 IMA DAI SDK for Android 的功能，并将提供的广告元数据完全集成到播放器中。例如，这样一来，您就可以使用 `Player.isPlayingAd()` 等方法，监听内容广告转换，并让播放器处理广告播放逻辑（例如跳过已播放的广告）。

如需使用此类，您需要设置 `ImaServerSideAdInsertionMediaSource.AdsLoader` 和 `ImaServerSideAdInsertionMediaSource.Factory`，并将它们连接到玩家：

### Kotlin

```kotlin
// 加载实际媒体流的 MediaSource.Factory。
val defaultMediaSourceFactory = DefaultMediaSourceFactory(context)
// 可以在多次播放中重复使用的 AdsLoader。
val adsLoader =
  ImaServerSideAdInsertionMediaSource.AdsLoader.Builder(context, adViewProvider).build()
// 为当前播放器创建广告源的 MediaSource.Factory。
val adsMediaSourceFactory =
  ImaServerSideAdInsertionMediaSource.Factory(adsLoader, defaultMediaSourceFactory)
// 配置 DefaultMediaSourceFactory 以创建 IMA DAI 源和普通媒体源。如果仅播放 IMA DAI 流，也可以直接使用 adsMediaSourceFactory。
defaultMediaSourceFactory.setServerSideAdInsertionMediaSourceFactory(adsMediaSourceFactory)
// 设置播放器的 MediaSource.Factory。
val player = ExoPlayer.Builder(context).setMediaSourceFactory(defaultMediaSourceFactory).build()
// 设置 AdsLoader 的播放器。
adsLoader.setPlayer(player)
```

### Java

```java
// 加载实际媒体流的 MediaSource.Factory。
DefaultMediaSourceFactory defaultMediaSourceFactory = new DefaultMediaSourceFactory(context);
// 可以在多次播放中重复使用的 AdsLoader。
ImaServerSideAdInsertionMediaSource.AdsLoader adsLoader =
    new ImaServerSideAdInsertionMediaSource.AdsLoader.Builder(context, adViewProvider).build();
// 为当前播放器创建广告源的 MediaSource.Factory。
ImaServerSideAdInsertionMediaSource.Factory adsMediaSourceFactory =
    new ImaServerSideAdInsertionMediaSource.Factory(adsLoader, defaultMediaSourceFactory);
// 配置 DefaultMediaSourceFactory 以创建 IMA DAI 源和普通媒体源。如果仅播放 IMA DAI 流，也可以直接使用 adsMediaSourceFactory。
defaultMediaSourceFactory.setServerSideAdInsertionMediaSourceFactory(adsMediaSourceFactory);
// 设置播放器的 MediaSource.Factory。
Player player =
    new ExoPlayer.Builder(context).setMediaSourceFactory(defaultMediaSourceFactory).build();
// 设置 AdsLoader 的播放器。
adsLoader.setPlayer(player);
```

使用 `ImaServerSideAdInsertionUriBuilder` 构建网址，以加载 IMA 资产键或内容来源 ID 和视频 ID：

### Kotlin

```kotlin
val ssaiUri =
  ImaServerSideAdInsertionUriBuilder()
    .setAssetKey(assetKey)
    .setFormat(C.CONTENT_TYPE_HLS)
    .build()
player.setMediaItem(MediaItem.fromUri(ssaiUri))
```

### Java

```java
Uri ssaiUri =
    new ImaServerSideAdInsertionUriBuilder()
        .setAssetKey(assetKey)
        .setFormat(C.CONTENT_TYPE_HLS)
        .build();
player.setMediaItem(MediaItem.fromUri(ssaiUri));
```

最后，在广告加载器不再使用后释放它：

### Kotlin

```kotlin
adsLoader.release()
```

### Java

```java
adsLoader.release();
```

#### 界面注意事项

与客户端广告插播相关的界面注意事项也适用于服务器端广告插播。

#### 随播广告

某些广告代码包含可以在应用界面的“广告位”中展示的其他随播广告。这些槽可通过 `ImaServerSideAdInsertionMediaSource.AdsLoader.Builder.setCompanionAdSlots(slots)` 传递。如需了解详情，请参阅添加随播广告。

### 使用第三方广告 SDK

如果您需要使用第三方广告 SDK 加载广告，不妨检查该 SDK 是否已提供 ExoPlayer 集成。如果不是，建议您提供一个自定义 `MediaSource`，以接受与 `ImaServerSideAdInsertionMediaSource` 类似的采用 `ssai://` 架构的 URI。

创建广告结构的实际逻辑可以委托给通用 `ServerSideAdInsertionMediaSource`，它会封装数据流 `MediaSource` 并允许用户设置和更新表示广告元数据的 `AdPlaybackState`。

通常，服务器端插入的广告串包含定时事件，用于通知玩家广告元数据。如需了解 ExoPlayer 支持哪些时间戳元数据格式，请参阅支持的格式。自定义广告 SDK `MediaSource` 实现可以使用 `Player.Listener.onMetadata` 监听来自播放器的时间戳元数据事件。

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

