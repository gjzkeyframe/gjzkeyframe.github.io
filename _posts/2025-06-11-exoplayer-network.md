---
title: ExoPlayer 从入门到精通（11）：网络栈
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-11 18:08:08 +0800
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



## 配置 ExoPlayer 使用特定网络栈

ExoPlayer 通过 `DataSource` 组件加载数据，这些组件从应用代码注入的 `DataSource.Factory` 实例中获取。

如果您的应用只需要播放 http(s) 内容，选择网络栈只需将应用注入的任何 `DataSource.Factory` 实例更新为使用对应于您希望使用的网络栈的 `HttpDataSource.Factory`。如果您的应用还需要播放非 http(s) 内容，如本地文件，请使用 `DefaultDataSource.Factory`。

以下示例展示了如何构建一个使用 Cronet 网络栈的 `ExoPlayer`，并支持播放非 http(s) 内容。

### Kotlin

```kotlin
// 给定一个 CronetEngine 和 Executor，构建一个 CronetDataSource.Factory。
val cronetDataSourceFactory = CronetDataSource.Factory(cronetEngine, executor)

// 将 CronetDataSource.Factory 包装在 DefaultDataSource.Factory 中，这增加了
// 从其他来源（如文件、资源等）请求数据的支持。
val dataSourceFactory =
  DefaultDataSource.Factory(context, /* baseDataSourceFactory= */ cronetDataSourceFactory)

// 在创建播放器时注入 DefaultDataSource.Factory。
val player =
  ExoPlayer.Builder(context)
    .setMediaSourceFactory(
      DefaultMediaSourceFactory(context).setDataSourceFactory(dataSourceFactory)
    )
    .build()
```

### Java

```java
// 给定一个 CronetEngine 和 Executor，构建一个 CronetDataSource.Factory。
CronetDataSource.Factory cronetDataSourceFactory =
    new CronetDataSource.Factory(cronetEngine, executor);

// 将 CronetDataSource.Factory 包装在 DefaultDataSource.Factory 中，这增加了
// 从其他来源（如文件、资源等）请求数据的支持。
DefaultDataSource.Factory dataSourceFactory =
    new DefaultDataSource.Factory(
        context, /* baseDataSourceFactory= */ cronetDataSourceFactory);

// 在创建播放器时注入 DefaultDataSource.Factory。
ExoPlayer player =
    new ExoPlayer.Builder(context)
        .setMediaSourceFactory(
            new DefaultMediaSourceFactory(context).setDataSourceFactory(dataSourceFactory))
        .build();
```

## 支持的网络栈

ExoPlayer 直接支持 HttpEngine、Cronet、OkHttp 和 Android 内置网络栈。ExoPlayer 还可以扩展以支持任何其他在 Android 上工作的网络栈。

### HttpEngine

HttpEngine 是从 API 34（或 S 扩展 7）开始在 Android 上推荐的默认网络栈。在大多数情况下，它内部使用 Cronet 网络栈，支持 HTTP、HTTP/2 和 HTTP/3 over QUIC 协议。

ExoPlayer 通过其 `HttpEngineDataSource.Factory` 支持 HttpEngine。您可以按照配置 ExoPlayer 使用特定网络栈中的说明注入此数据源工厂。

### Cronet

Cronet 是作为库提供给 Android 应用的 Chromium 网络栈。Cronet 利用多种技术减少应用需要工作的网络请求的延迟并提高吞吐量，包括 ExoPlayer 使用的技术。它原生支持 HTTP、HTTP/2 和 HTTP/3 over QUIC 协议。Cronet 被一些世界上最大的流媒体应用使用，包括 YouTube。

ExoPlayer 通过其 Cronet 库支持 Cronet。请参阅该库的 `README.md` 以获取详细使用说明。请注意，Cronet 库能够使用三种底层 Cronet 实现：

1. **Google Play 服务：** 我们建议在大多数情况下使用此实现，并在 Google Play 服务不可用时回退到 Android 内置网络栈（`DefaultHttpDataSource`）。
2. **Cronet 嵌入式：** 如果您的大部分用户位于 Google Play 服务不可用的市场，或者您想控制使用的 Cronet 实现的确切版本，这可能是一个不错的选择。Cronet 嵌入式的重大缺点是它会增加您的应用大约 8MB。
3. **Cronet 回退：** Cronet 回退实现是围绕 Android 内置网络栈的包装器。不建议与 ExoPlayer 一起使用，因为直接使用 Android 内置网络栈（通过使用 `DefaultHttpDataSource`）更高效。

### OkHttp

OkHttp 是另一个被许多流行 Android 应用广泛使用的现代网络栈。它支持 HTTP 和 HTTP/2，但尚未支持 HTTP/3 over QUIC。

ExoPlayer 通过其 OkHttp 库支持 OkHttp。请参阅该库的 `README.md` 以获取详细使用说明。使用 OkHttp 库时，网络栈嵌入在应用中。这类似于 Cronet 嵌入式，但 OkHttp 显著更小，增加的应用大小不到 1MB。

### Android 内置网络栈

ExoPlayer 支持使用 Android 内置网络栈，通过 `DefaultHttpDataSource` 和 `DefaultHttpDataSource.Factory`，它们是 ExoPlayer 核心库的一部分。

实际的网络栈实现取决于底层设备上运行的软件。在大多数设备上，仅支持 HTTP（即，不支持 HTTP/2 和 HTTP/3 over QUIC）。

### 其他网络栈

应用还可以将其他网络栈与 ExoPlayer 集成。为此，实现一个包装网络栈的 `HttpDataSource` 以及相应的 `HttpDataSource.Factory`。ExoPlayer 的 Cronet 和 OkHttp 库是这样做的良好示例。

当与纯 Java 网络栈集成时，实现一个 `DataSourceContractTest` 来检查您的 `HttpDataSource` 实现是否正确行为是个好主意。OkHttp 库中的 `OkHttpDataSourceContractTest` 是如何做到这一点的良好示例。

## 选择网络栈

下表概述了 ExoPlayer 支持的网络栈的优缺点。

| 网络栈 | 协议 | APK 大小影响 | 备注 |
| --- | --- | --- | --- |
| HttpEngine | HTTP<br>HTTP/2<br>HTTP/3 over QUIC | 无 | 仅在 API 34 或 S 扩展 7 上可用 |
| Cronet (Google Play 服务) | HTTP<br>HTTP/2<br>HTTP/3 over QUIC | 小<br>(<100KB) | 需要 Google Play 服务。Cronet 版本自动更新 |
| Cronet (嵌入式) | HTTP<br>HTTP/2<br>HTTP/3 over QUIC | 大<br>(~8MB) | 应用开发者控制 Cronet 版本 |
| Cronet (回退) | HTTP<br>(因设备而异) | 小<br>(<100KB) | 不推荐用于 ExoPlayer |
| OkHttp | HTTP<br>HTTP/2 | 小<br>(<1MB) |  |
| 内置网络栈 | HTTP<br>(因设备而异) | 无 | 实现因设备而异 |

HTTP/2 和 HTTP/3 over QUIC 协议可以显著提高媒体流性能。特别是当通过内容分发网络（CDN）分发自适应媒体时，使用这些协议可以让 CDN 运行得更高效。因此，与使用 Android 内置网络栈相比，HttpEngine 和 Cronet 对 HTTP/2 和 HTTP/3 over QUIC 的支持（以及 OkHttp 对 HTTP/2 的支持）是一个主要优势，前提是托管内容的服务器也支持这些协议。

仅考虑媒体流时，我们建议使用 HttpEngine 或由 Google Play 服务提供的 Cronet，并在 Google Play 服务不可用时回退到 `DefaultHttpDataSource`。这些建议在大多数设备上启用 HTTP/2 和 HTTP/3 over QUIC 的使用，并避免 APK 大小显著增加之间取得了良好的平衡。存在一些例外情况。如果您的应用将运行的大量设备上 Google Play 服务可能不可用，使用 Cronet 嵌入式或 OkHttp 可能更合适。如果 APK 大小是关键问题，或者媒体流只是应用功能的一小部分，使用内置网络栈可能是可以接受的。

除了媒体之外，通常最好为应用执行的所有网络操作选择一个单一的网络栈。这允许在 ExoPlayer 和其他应用组件之间高效地共享资源（如套接字）。

因为您的应用很可能需要执行与媒体播放无关的网络操作，所以您选择的网络栈应综合考虑上述媒体流的建议、其他执行网络操作的组件的要求以及它们对您的应用的相对重要性。

## 缓存媒体

ExoPlayer 支持将已加载的字节缓存到磁盘，以避免重复从网络加载相同的字节。这在当前媒体中回退或重复播放同一项目时非常有用。

缓存需要一个指向专用缓存目录的 `SimpleCache` 实例和一个 `CacheDataSource.Factory`。

### Kotlin

```kotlin
// 注意：这在您的应用中应该是一个单例。
val databaseProvider = StandaloneDatabaseProvider(context)

// 一个即时缓存应在达到最大磁盘空间限制时驱逐媒体。
val cache =
    SimpleCache(
        downloadDirectory, LeastRecentlyUsedCacheEvictor(maxBytes), databaseProvider)

// 使用缓存和所需 HTTP 栈的工厂配置 DataSource.Factory。
val cacheDataSourceFactory =
    CacheDataSource.Factory()
        .setCache(cache)
        .setUpstreamDataSourceFactory(httpDataSourceFactory)

// 在创建播放器时注入 DefaultDataSource.Factory。
val player =
    ExoPlayer.Builder(context)
        .setMediaSourceFactory(
            DefaultMediaSourceFactory(context).setDataSourceFactory(cacheDataSourceFactory))
        .build()
```

### Java

```java
// 注意：这在您的应用中应该是一个单例。
DatabaseProvider databaseProvider = new StandaloneDatabaseProvider(context);

// 一个即时缓存应在达到最大磁盘空间限制时驱逐媒体。
Cache cache =
    new SimpleCache(
        downloadDirectory, new LeastRecentlyUsedCacheEvictor(maxBytes), databaseProvider);

// 使用缓存和所需 HTTP 栈的工厂配置 DataSource.Factory。
DataSource.Factory cacheDataSourceFactory =
    new CacheDataSource.Factory()
        .setCache(cache)
        .setUpstreamDataSourceFactory(httpDataSourceFactory);

// 在创建播放器时注入 DefaultDataSource.Factory。
ExoPlayer player =
    new ExoPlayer.Builder(context)
        .setMediaSourceFactory(
            new DefaultMediaSourceFactory(context).setDataSourceFactory(cacheDataSourceFactory))
        .build();
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

