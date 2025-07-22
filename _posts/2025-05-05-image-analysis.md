---
title: 探索 CameraX（5）： 图像分析
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-05 18:08:08 +0800
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



## 操作模式

当应用的分析管道无法跟上 CameraX 的帧率要求时，可以配置 CameraX 以以下述方式之一丢弃帧：

  * **非阻塞模式** （默认模式）：在此模式下，执行器始终将最新图像缓存到图像缓冲区（类似于深度为一的队列），而应用分析上一图像。如果 CameraX 在应用完成处理之前接收到新图像，新图像将保存到同一缓冲区，覆盖上一图像。注意，在这种情况下，`ImageAnalysis.Builder.setImageQueueDepth()` 无效，缓冲区内容始终会被覆盖。可以通过调用 `setBackpressureStrategy()` 并传入 `STRATEGY_KEEP_ONLY_LATEST` 来启用此非阻塞模式。有关执行器影响的更多信息，请参阅 `STRATEGY_KEEP_ONLY_LATEST` 的参考文档。
  * **阻塞模式** ：在此模式下，内部执行器可以将多张图像添加到内部图像队列中，仅在队列满时开始丢弃帧。阻塞发生在整个相机设备范围内：如果相机设备有多个绑定的用例，在 CameraX 处理这些图像时，所有这些用例都会被阻塞。例如，当预览和图像分析同时绑定到相机设备时，在 CameraX 处理图像期间预览也会被阻塞。可以通过将 `STRATEGY_BLOCK_PRODUCER` 传给 `setBackpressureStrategy()` 来启用阻塞模式。还可以使用 `ImageAnalysis.Builder.setImageQueueDepth()` 配置图像队列深度。

如果分析器具有低延迟和高性能（分析图像的总时间小于 CameraX 帧的持续时间，例如 60fps 的 16ms），无论采用哪种操作模式，都能提供流畅的整体体验。即使在处理非常短暂的系统抖动时，阻塞模式仍可能有所帮助。

如果分析器具有高延迟和高性能，需要较长队列的阻塞模式来补偿延迟。但要注意，应用仍然可以处理所有帧。

如果分析器具有高延迟且费时（无法处理所有帧），非阻塞模式可能是更合适的选择，因为必须丢弃分析路径中的帧，但其他并发绑定的用例仍可以看到所有帧。

## 实现

要在应用中使用图像分析，请按照以下步骤操作：

  1. 构建 `ImageAnalysis` 用例。
  2. 创建 `ImageAnalysis.Analyzer`。
  3. 将分析器设置到 `ImageAnalysis` 中。
  4. 将生命周期所有者、相机选择器和 `ImageAnalysis` 用例绑定到生命周期。

绑定后，CameraX 立即将图像发送到注册的分析器。完成分析后，调用 `ImageAnalysis.clearAnalyzer()` 或解绑 `ImageAnalysis` 用例以停止分析。

### 构建 ImageAnalysis 用例

`ImageAnalysis` 将分析器（图像消费者）连接到 CameraX（图像生产者）。应用可以使用 `ImageAnalysis.Builder` 构建 `ImageAnalysis` 对象。通过 `ImageAnalysis.Builder`，应用可以配置以下内容：

  * 图像输出参数：
    * 格式：CameraX 通过 `setOutputImageFormat(int)` 支持 `YUV_420_888` 和 `RGBA_8888` 。默认格式为 `YUV_420_888` 。
    * 分辨率和纵横比：可以设置其中的一个参数，但不能同时设置两者。
    * 旋转。
    * 目标名称：用于调试目的。

  * 图像流程控制：
    * 背景执行器
    * 图像队列（分析器和相机之间）深度
    * 反压策略

应用可以设置分辨率或纵横比，但不能同时设置两者。实际输出分辨率取决于应用请求的尺寸（或纵横比）和硬件能力，可能与请求的尺寸或比例有所不同。有关分辨率匹配算法的信息，请参阅 `setTargetResolution()` 的文档。

应用可以配置输出图像像素为 YUV（默认）或 RGBA 颜色空间。当设置 RGBA 输出格式时，CameraX 在内部将图像从 YUV 转换为 RGBA 颜色空间，并将图像位打包到 ImageProxy 第一平面的 `ByteBuffer` 中（其他两个平面未使用），顺序如下：

```
ImageProxy.getPlanes()[0].buffer[0]: alpha
ImageProxy.getPlanes()[0].buffer[1]: red
ImageProxy.getPlanes()[0].buffer[2]: green
ImageProxy.getPlanes()[0].buffer[3]: blue
...
```

在设备无法跟上帧率时执行复杂的图像分析时，可以按照本文主题中的操作模式部分描述的策略配置 CameraX 以丢弃帧。

### 创建分析器

应用可以通过实现 `ImageAnalysis.Analyzer` 接口并覆写 `analyze(ImageProxy image)` 来创建分析器。在每个分析器中，应用接收一个 `ImageProxy`，它是 Media.Image 的包装器。可以使用 `ImageProxy.getFormat()` 查询图像格式。格式是应用通过 `ImageAnalysis.Builder` 提供的以下值之一：

  * 如果应用请求 `OUTPUT_IMAGE_FORMAT_RGBA_8888`，则为 `ImageFormat.RGBA_8888` 。
  * 如果应用请求 `OUTPUT_IMAGE_FORMAT_YUV_420_888`，则为 `ImageFormat.YUV_420_888` 。

有关颜色空间配置和像素字节检索位置的信息，请参阅构建 ImageAnalysis 用例。

在分析器内部，应用应执行以下操作：

  1. 尽快分析给定帧，最好在给定帧率时间限制内（例如，对于 30fps 的情况，小于 32ms）。如果应用无法足够快地分析帧，请考虑支持的丢帧机制之一。
  2. 通过调用 `ImageProxy.close()` 将 `ImageProxy` 释放回 CameraX 。注意，不应调用包装的 Media.Image 的关闭函数（`Media.Image.close()`）。

应用可以直接使用 ImageProxy 中的包装 `Media.Image` 。只是不要调用包装图像的 `Media.Image.close()`，因为这会破坏 CameraX 内部的图像共享机制；相反，使用 `ImageProxy.close()` 将底层 `Media.Image` 释放回 CameraX 。

### 将分析器配置为 ImageAnalysis

创建分析器后，使用 `ImageAnalysis.setAnalyzer()` 将其注册以开始分析。分析完成后，使用 `ImageAnalysis.clearAnalyzer()` 移除注册的分析器。

图像分析只能配置一个活跃的分析器。调用 `ImageAnalysis.setAnalyzer()` 会替换已注册的分析器（如果已存在）。应用可以在绑定用例之前或之后随时设置新分析器。

### 将 ImageAnalysis 绑定到生命周期

强烈建议使用 `ProcessCameraProvider.bindToLifecycle()` 函数将 `ImageAnalysis` 绑定到现有的 AndroidX 生命周期。注意，`bindToLifecycle()` 函数返回选定的 `Camera` 设备，可用于微调曝光等高级设置。有关控制相机输出的更多信息，请参阅此指南。

以下示例结合了前几个步骤的内容，将 CameraX `ImageAnalysis` 和 `Preview` 用例绑定到 `lifecycle` 所有者：

### Kotlin

```kotlin
val imageAnalysis = ImageAnalysis.Builder()
    // 如果需要 RGBA 输出，请启用下一行。
    // .setOutputImageFormat(ImageAnalysis.OUTPUT_IMAGE_FORMAT_RGBA_8888)
    .setTargetResolution(Size(1280, 720))
    .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
    .build()
imageAnalysis.setAnalyzer(executor, ImageAnalysis.Analyzer { imageProxy ->
    val rotationDegrees = imageProxy.imageInfo.rotationDegrees
    // 在此处插入代码。
    ...
    // 完成后，释放 ImageProxy 对象。
    imageProxy.close()
})

cameraProvider.bindToLifecycle(this as LifecycleOwner, cameraSelector, imageAnalysis, preview)
```


### Java

```java
ImageAnalysis imageAnalysis =
    new ImageAnalysis.Builder()
        // 如果需要 RGBA 输出，请启用下一行。
        //.setOutputImageFormat(ImageAnalysis.OUTPUT_IMAGE_FORMAT_RGBA_8888)
        .setTargetResolution(new Size(1280, 720))
        .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
        .build();

imageAnalysis.setAnalyzer(executor, new ImageAnalysis.Analyzer() {
    @Override
    public void analyze(@NonNull ImageProxy imageProxy) {
        int rotationDegrees = imageProxy.getImageInfo().getRotationDegrees();
            // 在此处插入代码。
            ...
            // 完成后，释放 ImageProxy 对象。
            imageProxy.close();
        }
    });

cameraProvider.bindToLifecycle((LifecycleOwner) this, cameraSelector, imageAnalysis, preview);
```

## 其他资源

要了解更多关于 CameraX 的信息，请参见以下额外资源：

  * Codelab ：[开始使用 CameraX](https://developer.android.com/camerax)
  * 代码示例 ：[CameraX 示例应用](https://github.com/android/CameraX-Samples)


---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }