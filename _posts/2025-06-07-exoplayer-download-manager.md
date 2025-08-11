---
title: ExoPlayer 从入门到精通（7）： 下载媒体
description: 系统介绍 Android ExoPlayer 相关的基础技术。
author: Keyframe
date: 2025-06-07 18:08:08 +0800
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


ExoPlayer 提供了用于离线播放的媒体下载功能。在大多数使用场景中，即使应用处于后台，下载也应继续进行。对于这些场景，您的应用应继承 `DownloadService`，并通过命令将下载添加、移除和控制。以下图示展示了涉及的主要类。

![img](https://developer.android.com/static/guide/topics/media/exoplayer/images/downloading.svg)

- `DownloadService`：包装 `DownloadManager` 并向其转发命令。该服务允许 `DownloadManager` 即使在应用处于后台时也能继续运行。
- `DownloadManager`：管理多个下载任务，从（和到）`DownloadIndex` 加载（和存储）它们的状态，根据网络连接等要求启动和停止下载。为了下载内容，管理器通常会从 `HttpDataSource` 读取正在下载的数据，并将其写入 `Cache`。
- `DownloadIndex`：持久化下载任务的状态。

## 创建 DownloadService

要创建 `DownloadService`，需继承它并实现其抽象方法：

- `getDownloadManager()`：返回要使用的 `DownloadManager`。
- `getScheduler()`：返回可选的 `Scheduler`，当满足挂起下载的条件时，它可以重新启动服务。ExoPlayer 提供了以下实现：

  - `PlatformScheduler`，使用 JobScheduler（最低 API 级别为 21）。请参阅 PlatformScheduler 的 Javadoc 以了解应用权限要求。
  - `WorkManagerScheduler`，使用 WorkManager。
- `getForegroundNotification()`：返回当服务在前台运行时要显示的通知。可以使用 `DownloadNotificationHelper.buildProgressNotification` 创建默认样式的通知。

最后，在 `AndroidManifest.xml` 文件中定义服务：

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC"/>
<application>
  <service android:name="com.myapp.MyDownloadService"
      android:exported="false"
      android:foregroundServiceType="dataSync">
    <!-- 这是 Scheduler 所需的 -->
    <intent-filter>
      <action android:name="androidx.media3.exoplayer.downloadService.action.RESTART"/>
      <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
  </service>
</application>
```

可在 ExoPlayer 示例应用中查看 `DemoDownloadService` 和 `AndroidManifest.xml` 的具体示例。

## 创建 DownloadManager

以下代码片段演示了如何实例化 `DownloadManager`，该实例可由 `DownloadService` 中的 `getDownloadManager()` 返回：

### Kotlin

```kotlin
// 注意：这在您的应用中应为单例。
val databaseProvider = StandaloneDatabaseProvider(context)

// 下载缓存不应驱逐媒体，因此应使用 NoopCacheEvictor。
val downloadCache = SimpleCache(downloadDirectory, NoOpCacheEvictor(), databaseProvider)

// 创建用于从网络读取数据的工厂。
val dataSourceFactory = DefaultHttpDataSource.Factory()

// 选择用于下载数据的执行器。使用 Runnable::run 将导致每个下载任务在其自身线程上下载数据。传递使用多个线程的执行器将加速可以拆分为更小部分以并行执行的下载任务。已经为后台下载设有执行器的应用可能希望复用现有的执行器。
val downloadExecutor = Executor(Runnable::run)

// 创建下载管理器。
val downloadManager =
  DownloadManager(context, databaseProvider, downloadCache, dataSourceFactory, downloadExecutor)

// 可选地，可以赋值属性以配置下载管理器。
downloadManager.requirements = requirements
downloadManager.maxParallelDownloads = 3
```

### Java

```java
// 注意：这在您的应用中应为单例。
databaseProvider = new StandaloneDatabaseProvider(context);

// 下载缓存不应驱逐媒体，因此应使用 NoopCacheEvictor。
downloadCache = new SimpleCache(downloadDirectory, new NoOpCacheEvictor(), databaseProvider);

// 创建用于从网络读取数据的工厂。
dataSourceFactory = new DefaultHttpDataSource.Factory();

// 选择用于下载数据的执行器。使用 Runnable::run 将导致每个下载任务在其自身线程上下载数据。传递使用多个线程的执行器将加速可以拆分为更小部分以并行执行的下载任务。已经为后台下载设有执行器的应用可能希望复用现有的执行器。
Executor downloadExecutor = Runnable::run;

// 创建下载管理器。
downloadManager =
    new DownloadManager(
        context, databaseProvider, downloadCache, dataSourceFactory, downloadExecutor);

// 可选地，可以调用设置方法以配置下载管理器。
downloadManager.setRequirements(requirements);
downloadManager.setMaxParallelDownloads(3);
```

可在示例应用的 `DemoUtil` 中查看具体示例。

## 添加下载任务

要添加下载任务，创建 `DownloadRequest` 并将其发送到 `DownloadService`。对于自适应流，使用 `DownloadHelper` 来帮助构建 `DownloadRequest`。以下示例展示了如何创建下载请求：

### Kotlin

```kotlin
val downloadRequest = DownloadRequest.Builder(contentId, contentUri).build()
```

### Java

```java
DownloadRequest downloadRequest = new DownloadRequest.Builder(contentId, contentUri).build();
```

在此示例中，`contentId` 是内容的唯一标识符。在简单情况下，通常可以将 `contentUri` 用作 `contentId`，但应用可以自由使用最适合其使用场景的 ID 方案。`DownloadRequest.Builder` 还有一些可选的设置方法。例如，`setKeySetId` 和 `setData` 可用于设置 DRM 和应用希望与下载关联的自定义数据。还可以使用 `setMimeType` 指定内容的 MIME 类型，作为在无法从 `contentUri` 推断内容类型时的提示。

创建后，请求可以发送到 `DownloadService` 以添加下载任务：

### Kotlin

```kotlin
DownloadService.sendAddDownload(
  context,
  MyDownloadService::class.java,
  downloadRequest,
  /* foreground= */ false
)
```

### Java

```java
DownloadService.sendAddDownload(
    context, MyDownloadService.class, downloadRequest, /* foreground= */ false);
```

在此示例中，`MyDownloadService` 是应用的 `DownloadService` 子类，`foreground` 参数控制服务是否以前台方式启动。如果应用已经处于前台，则通常应将 `foreground` 参数设置为 `false`，因为 `DownloadService` 会在确定有工作要做时自行进入前台。

## 移除下载任务

可以通过向 `DownloadService` 发送移除命令来移除下载任务，其中 `contentId` 用于标识要移除的下载任务：

### Kotlin

```kotlin
DownloadService.sendRemoveDownload(
  context,
  MyDownloadService::class.java,
  contentId,
  /* foreground= */ false
)
```

### Java

```java
DownloadService.sendRemoveDownload(
    context, MyDownloadService.class, contentId, /* foreground= */ false);
```

您也可以使用 `DownloadService.sendRemoveAllDownloads` 移除所有已下载的数据。

## 启动和停止下载任务

只有在满足以下四个条件时，下载任务才会进行：

- 下载任务没有停止原因。
- 下载任务未暂停。
- 满足下载任务进行的要求。要求可以指定允许的网络类型，以及设备是否应处于空闲状态或连接到充电器。
- 未超过最大并行下载数量。

所有这些条件都可以通过向 `DownloadService` 发送命令来控制。

### 设置和清除下载停止原因

可以为一个或所有下载任务设置停止原因：

### Kotlin

```kotlin
// 为单个下载设置停止原因。
DownloadService.sendSetStopReason(
  context,
  MyDownloadService::class.java,
  contentId,
  stopReason,
  /* foreground= */ false
)

// 为单个下载清除停止原因。
DownloadService.sendSetStopReason(
  context,
  MyDownloadService::class.java,
  contentId,
  Download.STOP_REASON_NONE,
  /* foreground= */ false
)
```

### Java

```java
// 为单个下载设置停止原因。
DownloadService.sendSetStopReason(
    context, MyDownloadService.class, contentId, stopReason, /* foreground= */ false);

// 为单个下载清除停止原因。
DownloadService.sendSetStopReason(
    context,
    MyDownloadService.class,
    contentId,
    Download.STOP_REASON_NONE,
    /* foreground= */ false);
```

`stopReason` 可以是任何非零值（`Download.STOP_REASON_NONE = 0` 是一个特殊值，表示下载未停止）。有多个停止下载原因的应用可以使用不同的值来跟踪每个下载停止的原因。为所有下载任务设置和清除停止原因的方式与单个下载任务相同，只是 `contentId` 应设置为 `null`。

当下载任务有非零的停止原因时，它将处于 `Download.STATE_STOPPED` 状态。停止原因会持久化在 `DownloadIndex` 中，如果应用进程被杀死并重新启动，这些原因将被保留。

### 暂停和恢复所有下载任务

可以如下暂停和恢复所有下载任务：

### Kotlin

```kotlin
// 暂停所有下载。
DownloadService.sendPauseDownloads(
  context,
  MyDownloadService::class.java,
  /* foreground= */ false
)

// 恢复所有下载。
DownloadService.sendResumeDownloads(
  context,
  MyDownloadService::class.java,
  /* foreground= */ false
)
```

### Java

```java
// 暂停所有下载。
DownloadService.sendPauseDownloads(context, MyDownloadService.class, /* foreground= */ false);

// 恢复所有下载。
DownloadService.sendResumeDownloads(context, MyDownloadService.class, /* foreground= */ false);
```

当下载任务被暂停时，它们将处于 `Download.STATE_QUEUED` 状态。与设置停止原因不同，这种方法不会持久化任何状态更改。它仅影响 `DownloadManager` 的运行时状态。

### 设置下载任务进行的要求

`Requirements` 可用于指定下载任务进行必须满足的约束条件。要求可以在创建 `DownloadManager` 时通过调用 `DownloadManager.setRequirements()` 设置，如上面的示例所示。也可以通过向 `DownloadService` 发送命令动态更改要求：

### Kotlin

```kotlin
// 设置下载要求。
DownloadService.sendSetRequirements(
  context, MyDownloadService::class.java, requirements, /* foreground= */ false)
```

### Java

```java
// 设置下载要求。
DownloadService.sendSetRequirements(
  context,
  MyDownloadService.class,
  requirements,
  /* foreground= */ false);
```

当下载任务因不满足要求而无法进行时，它将处于 `Download.STATE_QUEUED` 状态。可以使用 `DownloadManager.getNotMetRequirements()` 查询不满足的要求。

### 设置最大并行下载数量

可以通过调用 `DownloadManager.setMaxParallelDownloads()` 设置最大并行下载数量。通常在创建 `DownloadManager` 时完成此设置，如上面的示例所示。

当下载任务因达到最大并行下载数量限制而无法进行时，它将处于 `Download.STATE_QUEUED` 状态。

## 查询下载任务

可以通过 `DownloadManager` 的 `DownloadIndex` 查询所有下载任务的状态，包括已完成或失败的下载任务。可以通过调用 `DownloadManager.getDownloadIndex()` 获取 `DownloadIndex`，然后通过调用 `DownloadIndex.getDownloads()` 获取遍历所有下载任务的游标。或者，可以通过调用 `DownloadIndex.getDownload()` 查询单个下载任务的状态。

`DownloadManager` 还提供了 `DownloadManager.getCurrentDownloads()`，该方法仅返回当前（即未完成或失败）下载任务的状态。此方法对于更新显示当前下载任务进度和状态的通知和 UI 组件非常有用。

## 监听下载任务

可以向 `DownloadManager` 添加监听器，以便在当前下载任务状态更改时获得通知：

### Kotlin

```kotlin
downloadManager.addListener(
  object : DownloadManager.Listener { // 在此处覆盖感兴趣的方法。
  }
)
```

### Java

```java
downloadManager.addListener(
    new DownloadManager.Listener() {
      // 在此处覆盖感兴趣的方法。
    });
```

可在示例应用的 `DownloadTracker` 类中查看 `DownloadManagerListener` 的具体示例。

## 播放已下载的内容

播放已下载的内容与播放在线内容类似，只是数据是从下载 `Cache` 读取而不是通过网络获取。

要播放已下载的内容，使用与下载相同的 `Cache` 实例创建 `CacheDataSource.Factory`，并在构建播放器时将其注入 `DefaultMediaSourceFactory`：

### Kotlin

```kotlin
// 使用下载缓存创建只读缓存数据源工厂。
val cacheDataSourceFactory: DataSource.Factory =
  CacheDataSource.Factory()
    .setCache(downloadCache)
    .setUpstreamDataSourceFactory(httpDataSourceFactory)
    .setCacheWriteDataSinkFactory(null) // 禁用写入。

val player =
  ExoPlayer.Builder(context)
    .setMediaSourceFactory(
      DefaultMediaSourceFactory(context).setDataSourceFactory(cacheDataSourceFactory)
    )
    .build()
```

### Java

```java
// 使用下载缓存创建只读缓存数据源工厂。
DataSource.Factory cacheDataSourceFactory =
    new CacheDataSource.Factory()
        .setCache(downloadCache)
        .setUpstreamDataSourceFactory(httpDataSourceFactory)
        .setCacheWriteDataSinkFactory(null); // 禁用写入。

ExoPlayer player =
    new ExoPlayer.Builder(context)
        .setMediaSourceFactory(
            new DefaultMediaSourceFactory(context).setDataSourceFactory(cacheDataSourceFactory))
        .build();
```

如果同一播放器实例也将用于播放未下载的内容，则应将 `CacheDataSource.Factory` 配置为只读，以避免在播放期间下载该内容。

配置好播放器后，它将能够访问已下载的内容以供播放。播放下载内容只需将相应的 `MediaItem` 传递给播放器。可以从 `Download` 使用 `Download.request.toMediaItem` 获取 `MediaItem`，也可以从 `DownloadRequest` 使用 `DownloadRequest.toMediaItem` 直接获取。

### MediaSource 配置

前述示例使下载缓存可用于播放所有 `MediaItem`。您还可以使下载缓存可用于单个 `MediaSource` 实例，这些实例可以直接传递给播放器：

### Kotlin

```kotlin
val mediaSource =
  ProgressiveMediaSource.Factory(cacheDataSourceFactory)
    .createMediaSource(MediaItem.fromUri(contentUri))
player.setMediaSource(mediaSource)
player.prepare()
```

### Java

```java
ProgressiveMediaSource mediaSource =
    new ProgressiveMediaSource.Factory(cacheDataSourceFactory)
        .createMediaSource(MediaItem.fromUri(contentUri));
player.setMediaSource(mediaSource);
player.prepare();
```

## 下载和播放自适应流

自适应流（例如 DASH、SmoothStreaming 和 HLS）通常包含多个媒体轨道。通常有多个轨道包含相同内容但质量不同（例如 SD、HD 和 4K 视频轨道）。也可能有同一类型的多个轨道包含不同内容（例如多种语言的音频轨道）。

对于流式播放，可以使用轨道选择器选择播放的轨道。同样，对于下载，可以使用 `DownloadHelper` 选择下载的轨道。`DownloadHelper` 的典型用法如下：

1. 使用其中一个 `DownloadHelper.forMediaItem` 方法构建 `DownloadHelper`。准备助手并等待回调。

### Kotlin

```kotlin
val downloadHelper =
    DownloadHelper.forMediaItem(
      context,
      MediaItem.fromUri(contentUri),
      DefaultRenderersFactory(context),
      dataSourceFactory
    )
downloadHelper.prepare(callback)
```

### Java

```java
DownloadHelper downloadHelper =
      DownloadHelper.forMediaItem(
          context,
          MediaItem.fromUri(contentUri),
          new DefaultRenderersFactory(context),
          dataSourceFactory);
downloadHelper.prepare(callback);
```

2. 可选地，使用 `getMappedTrackInfo` 和 `getTrackSelections` 检查默认选定的轨道，并使用 `clearTrackSelections`、`replaceTrackSelections` 和 `addTrackSelection` 进行调整。
3. 通过调用 `getDownloadRequest` 创建所选轨道的 `DownloadRequest`。可以将请求传递给 `DownloadService` 以添加下载任务，如上所述。
4. 使用 `release()` 释放助手。

播放已下载的自适应内容需要配置播放器并传递相应的 `MediaItem`，如上所述。

在构建 `MediaItem` 时，`MediaItem.localConfiguration.streamKeys` 必须设置为与 `DownloadRequest` 中的匹配，以便播放器仅尝试播放已下载的轨道子集。使用 `Download.request.toMediaItem` 和 `DownloadRequest.toMediaItem` 构建 `MediaItem` 将为您处理此问题。

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

