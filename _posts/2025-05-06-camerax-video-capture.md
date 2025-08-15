---
title: 探索 CameraX（6）：视频捕获架构
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-06 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 音频, 拍摄, CameraX]
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



## 捕获系统概述

捕获系统通常会记录视频和音频流，对其进行压缩，复用这两种流，然后将结果流写入磁盘。

![捕获系统概念图](https://developer.android.com/static/images/training/camera/camerax/conceptual-capturing-system.png)**图 1.** 视频和音频捕获系统的概念图。

在 CameraX 中，视频捕获的解决方案是 `VideoCapture` 用例：

![CameraX VideoCapture 用例概念图](https://developer.android.com/static/images/training/camera/camerax/videocapture-use-case.png)**图 2.** 显示 CameraX 如何处理 `VideoCapture` 用例的概念图。

如图 2 所示，CameraX 视频捕获包括几个高级架构组件：

  * 用于视频源的 `SurfaceProvider` 。
  * 用于音频源的 `AudioSource` 。
  * 两个编码器用于编码和压缩视频 / 音频。
  * 一个媒体复用器用于复用两种流。
  * 一个文件保存器用于写入结果。

`VideoCapture` API 抽象了复杂的捕获引擎，为应用提供了更简单直接的 API。

## VideoCapture API 概览

`VideoCapture` 是一个 CameraX 用例，可以单独使用，也可以与其他用例组合使用。支持的具体组合取决于相机硬件能力，但 `Preview` 和 `VideoCapture` 在所有设备上都是有效的用例组合。

VideoCapture API 包括以下与应用通信的对象：

  * `VideoCapture` 是顶级用例类。`VideoCapture` 与 `LifecycleOwner`、`CameraSelector` 以及其他 CameraX 用例绑定。有关这些概念和用法的更多信息，请参阅 CameraX 架构。
  * `Recorder` 是 `VideoOutput` 的一种实现，与 `VideoCapture` 紧密耦合。`Recorder` 用于执行视频和音频捕获。应用从 `Recorder` 创建录音。
  * `PendingRecording` 配置录音，提供启用音频和设置事件监听器等选项。必须使用 `Recorder` 创建 `PendingRecording`。`PendingRecording` 不会录制任何内容。
  * `Recording` 执行实际的录音。必须使用 `PendingRecording` 创建 `Recording` 。

图 3 显示了这些对象之间的关系：

![VideoCapture 用例中的交互图](https://developer.android.com/static/images/training/camera/camerax/videocapture-roles-interaction.png)**图 3.** 显示 VideoCapture 用例中发生的交互的图。

**图例：**

  1. 使用 `QualitySelector` 创建 `Recorder` 。
  2. 使用其中一种 `OutputOptions` 配置 `Recorder` 。
  3. 如果需要，使用 `withAudioEnabled()` 启用音频。
  4. 调用 `start()` 并带有 `VideoRecordEvent` 监听器以开始录音。
  5. 在 `Recording` 上使用 `pause()`/`resume()`/`stop()` 控制录音。
  6. 在事件监听器内响应 `VideoRecordEvents` 。

详细的 API 列表在源代码的 current.txt 中。

## 使用 VideoCapture API

要将 CameraX `VideoCapture` 用例集成到应用中，请执行以下操作：

  1. 绑定 `VideoCapture` 。
  2. 准备和配置录音。
  3. 开始并控制运行时录音。

以下各节概述了在每个步骤中可以执行的操作，以获得端到端的录音会话。

### 绑定 VideoCapture

要绑定 `VideoCapture` 用例，请执行以下操作：

  1. 创建 `Recorder` 对象。
  2. 创建 `VideoCapture` 对象。
  3. 与生命周期绑定。

CameraX VideoCapture API 遵循构建者设计模式。应用使用 `Recorder.Builder` 创建 `Recorder` 。还可以通过 `QualitySelector` 对象为 `Recorder` 配置视频分辨率。

CameraX `Recorder` 支持以下预定义的视频分辨率 `Qualities` ：

  * `Quality.UHD` 用于 4K 超高清视频尺寸（2160p）。
  * `Quality.FHD` 用于全高清视频尺寸（1080p）。
  * `Quality.HD` 用于高清视频尺寸（720p）。
  * `Quality.SD` 用于标清视频尺寸（480p）。

注意，当应用授权时，CameraX 也可以选择其他分辨率。

每个选择的确切视频尺寸取决于相机和编码器的能力。有关更多信息，请参阅 `CamcorderProfile` 的文档。

应用可以通过创建 `QualitySelector` 来配置分辨率。可以使用以下方法之一创建 `QualitySelector` ：

  * 使用 `fromOrderedList()` 提供一些首选分辨率，并包含在不支持首选分辨率时使用的回退策略。CameraX 可以根据选定相机的能力决定最佳的回退匹配，有关详细信息，请参阅 `QualitySelector` 的 `FallbackStrategy 规范` 。例如，以下代码请求支持的最高分辨率进行录音，如果请求的分辨率都不受支持，授权 CameraX 选择最接近 Quality.SD 分辨率的分辨率：

```kotlin
val qualitySelector = QualitySelector.fromOrderedList(
    listOf(Quality.UHD, Quality.FHD, Quality.HD, Quality.SD),
    FallbackStrategy.lowerQualityOrHigherThan(Quality.SD))
```


  * 首先查询相机能力，然后使用 `QualitySelector::from()` 从支持的分辨率中选择：

```kotlin
val cameraInfo = cameraProvider.availableCameraInfos.filter {
    Camera2CameraInfo
        .from(it)
        .getCameraCharacteristic(CameraCharacteristics.LENS_FACING) == CameraMetadata.LENS_FACING_BACK
}

val supportedQualities = QualitySelector.getSupportedQualities(cameraInfo[0])
val filteredQualities = arrayListOf<Quality>(Quality.UHD, Quality.FHD, Quality.HD, Quality.SD)
    .filter { supportedQualities.contains(it) }

// 使用 id 为 simple_quality_list_view 的简单 ListView
viewBinding.simpleQualityListView.apply {
    adapter = ArrayAdapter(context,
        android.R.layout.simple_list_item_1,
        filteredQualities.map { it.qualityToString() })

    // 设置用户交互以手动显示或隐藏系统 UI。
    setOnItemClickListener { _, _, position, _ ->
        // 在 View.OnClickListener 内部，
        // 将 Quality.* 常量转换为 QualitySelector
        val qualitySelector = QualitySelector.from(filteredQualities[position])

        // 创建新的 Recorder/VideoCapture 用于新质量
        // 并绑定到生命周期
        val recorder = Recorder.Builder()
            .setQualitySelector(qualitySelector).build()

        // ...
    }
}

// 辅助函数将 Quality 转换为字符串
fun Quality.qualityToString(): String {
    return when (this) {
        Quality.UHD -> "UHD"
        Quality.FHD -> "FHD"
        Quality.HD -> "HD"
        Quality.SD -> "SD"
        else -> throw IllegalArgumentException()
    }
}
```


注意，`QualitySelector.getSupportedQualities()` 返回的能力保证可用于 `VideoCapture` 用例或 `VideoCapture` 和 `Preview` 用例的组合。当与 `ImageCapture` 或 `ImageAnalysis` 用例绑定时，如果请求的相机不支持所需的组合，CameraX 可能仍然会绑定失败。

一旦有了 `QualitySelector`，应用就可以创建 `VideoCapture` 对象并执行绑定。注意，此绑定与其他用例的绑定相同：

```kotlin
val recorder = Recorder.Builder()
    .setExecutor(cameraExecutor).setQualitySelector(qualitySelector)
    .build()
val videoCapture = VideoCapture.withOutput(recorder)

try {
    // 将用例绑定到相机
    cameraProvider.bindToLifecycle(
        this, CameraSelector.DEFAULT_BACK_CAMERA, preview, videoCapture)
} catch (exc: Exception) {
    Log.e(TAG, "用例绑定失败", exc)
}
```

注意，`bindToLifecycle()` 返回一个 `Camera` 对象。有关控制相机输出的更多信息（例如缩放和曝光），请参阅此指南。

`Recorder` 为系统选择最适合的格式。最常见的视频编解码器是 H.264 AVC，容器格式为 MPEG-4。

## 配置和创建录音

从 `Recorder` 中，应用可以创建录音对象以执行视频和音频捕获。应用通过执行以下操作创建录音：

  1. 使用 `prepareRecording()` 配置 `OutputOptions` 。
  2. （可选）启用音频录制。
  3. 使用 `start()` 注册 `VideoRecordEvent` 监听器，并开始视频捕获。

调用 `start()` 函数时，`Recorder` 返回一个 `Recording` 对象。应用可以使用此 `Recording` 对象完成捕获或其他操作，例如暂停或继续。

一个 `Recorder` 同时支持一个 `Recording` 对象。可以在对上一个 `Recording` 对象调用 `Recording.stop()` 或 `Recording.close()` 后开始新的录音。

让我们更详细地查看这些步骤。首先，应用使用 `Recorder.prepareRecording()` 为 `Recorder` 配置 `OutputOptions` 。`Recorder` 支持以下类型的 `OutputOptions` ：

  * `FileDescriptorOutputOptions` 用于捕获到 `FileDescriptor` 。
  * `FileOutputOptions` 用于捕获到 `File` 。
  * `MediaStoreOutputOptions` 用于捕获到 `MediaStore` 。

所有 `OutputOptions` 类型都允许使用 `setFileSizeLimit()` 设置最大文件大小。其他选项因输出类型而异，例如 `FileDescriptorOutputOptions` 的 `ParcelFileDescriptor` 。

`prepareRecording()` 返回一个 `PendingRecording` 对象，该对象用于创建相应的 `Recording` 对象。`PendingRecording` 是一个在大多数情况下应不可见的中间类，应用也很少缓存它。

应用可以进一步配置录音：

  * 使用 `withAudioEnabled()` 启用音频。
  * 使用 `start(Executor, Consumer<VideoRecordEvent>)` 注册监听器以接收视频录制事件。
  * 使用 `PendingRecording.asPersistentRecording()` 允许在附加到的 VideoCapture 重新绑定到另一台相机时持续录制。

要开始录制，请调用 `PendingRecording.start()` 。CameraX 将 `PendingRecording` 转换为 `Recording`，排队录音请求，并将新创建的 `Recording` 对象返回给应用。一旦相应的相机设备开始录制，CameraX 就会发送 `VideoRecordEvent.EVENT_TYPE_START` 事件。

以下示例显示如何将视频和音频录制到 `MediaStore` 文件中：

```kotlin
// 为我们的录制器创建 MediaStoreOutputOptions
val name = "CameraX-recording-" +
        SimpleDateFormat(FILENAME_FORMAT, Locale.US)
            .format(System.currentTimeMillis()) + ".mp4"
val contentValues = ContentValues().apply {
    put(MediaStore.Video.Media.DISPLAY_NAME, name)
}
val mediaStoreOutput = MediaStoreOutputOptions.Builder(this.contentResolver,
    MediaStore.Video.Media.EXTERNAL_CONTENT_URI)
    .setContentValues(contentValues)
    .build()

// 2. 配置录制器并开始录制到 mediaStoreOutput。
val recording = videoCapture.output
    .prepareRecording(context, mediaStoreOutput)
    .withAudioEnabled()
    .start(ContextCompat.getMainExecutor(this), captureListener)
```

虽然相机预览默认情况下会镜像前置相机，但 VideoCapture 录制的视频默认情况下不会镜像。从 CameraX 1.3 开始，现在可以镜像视频录制，使前置相机预览和录制的视频相匹配。

有三种镜像模式选项：MIRROR_MODE_OFF、MIRROR_MODE_ON 和 MIRROR_MODE_ON_FRONT_ONLY。为了与相机预览对齐，Google 建议使用 MIRROR_MODE_ON_FRONT_ONLY，这意味着后置相机不启用镜像，但前置相机启用镜像。有关 MirrorMode 的更多信息，请参阅 MirrorMode 常量。

以下代码片段展示了如何使用 MIRROR_MODE_ON_FRONT_ONLY 调用 VideoCapture.Builder.setMirrorMode() 。有关更多信息，请参阅 setMirrorMode()。

### Kotlin

```kotlin
val recorder = Recorder.Builder().build()

val videoCapture = VideoCapture.Builder(recorder)
    .setMirrorMode(MIRROR_MODE_ON_FRONT_ONLY)
    .build()

useCases.add(videoCapture)
```

### Java

```java
Recorder.Builder builder = new Recorder.Builder();
if (mVideoQuality != QUALITY_AUTO) {
    builder.setQualitySelector(
        QualitySelector.from(mVideoQuality));
}
VideoCapture<Recorder> videoCapture = new VideoCapture.Builder<>(builder.build())
    .setMirrorMode(MIRROR_MODE_ON_FRONT_ONLY)
    .build();
useCases.add(videoCapture);
```

## 控制活跃录音

可以通过以下方法暂停、继续和停止正在进行的 `Recording` ：

  * `pause` 暂停当前活跃录音。
  * `resume()` 继续已暂停的活跃录音。
  * `stop()` 完成录音并清除任何相关的录音对象。
  * `mute()` 静音或取消静音当前录音。

注意，无论录音是处于暂停状态还是活跃状态，都可以调用 `stop()` 来终止 `Recording` 。

如果通过 `PendingRecording.start()` 注册了 `EventListener`，`Recording` 将使用 `VideoRecordEvent` 进行通信。

  * `VideoRecordEvent.EVENT_TYPE_STATUS` 用于录音统计信息，如当前文件大小和录制时间跨度。
  * `VideoRecordEvent.EVENT_TYPE_FINALIZE` 用于录音结果，包括最终文件的 URI 以及任何相关错误。

一旦应用收到指示成功录音会话的 `EVENT_TYPE_FINALIZE`，就可以从 `OutputOptions` 指定的位置访问捕获的视频。

## 其他资源

要了解更多关于 CameraX 的信息，请参阅以下额外资源：

  * [开始使用 CameraX Codelab](https://developer.android.com/camerax)


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