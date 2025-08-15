---
title: 探索 CameraX（4）： 捕获图像
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-04 18:08:08 +0800
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



## 关键概念

本文主要讨论以下概念：

  * **存储方法** ：可以将图像捕获到内存缓冲区或直接保存到文件中。
  * **执行器** ：`ImageCapture` 使用执行器来处理回调和 I/O 操作。可以自定义这些执行器以提高性能和控制能力。
  * **捕获模式** ：可以配置捕获模式以优化延迟或图像质量。

### 存储方法

使用 `ImageCapture` 捕获图像有两种方式，它们分别使用 `ImageCapture.takePicture()` 的重载方法：

  * **文件** ：使用 `takePicture(OutputFileOptions, Executor, OnImageSavedCallback)` 将捕获的图像直接保存到磁盘上的文件中。

    * 这是最常见的拍照方式。
  * **内存中** ：使用 `takePicture(Executor, OnImageCapturedCallback)` 接收捕获图像的内存缓冲区。

    * 这对于实时图像处理或分析非常有用。

### 执行器

调用 `takePicture` 时，需要传递一个 `Executor` 以及 `OnImageCapturedCallback` 或 `OnImageSavedCallback` 函数。`Executor` 运行回调并处理任何结果 IO。

## 拍照

要拍照，需要先设置相机，然后调用 `takePicture`。

### 设置相机

要设置相机，请创建一个 `CameraProvider`，然后创建一个 `ImageCapture` 对象。使用 `ImageCapture.Builder()`：

### Kotlin

```kotlin
val imageCapture = ImageCapture.Builder()
    .setTargetRotation(view.display.rotation)
    .build()

cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, imageCapture, preview)
```


### Java

```java
ImageCapture imageCapture = new ImageCapture.Builder()
    .setTargetRotation(view.getDisplay().getRotation())
    .build();

cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, imageCapture, preview);
```



### 拍照

配置好相机后，调用 `takePicture()` 捕获图像。以下示例展示了如何使用 `takePicture()` 将图像保存到磁盘：

### Kotlin

```kotlin
fun onClick() {
    val outputFileOptions = ImageCapture.OutputFileOptions.Builder(File(...)).build()
    imageCapture.takePicture(outputFileOptions, cameraExecutor,
        object : ImageCapture.OnImageSavedCallback {
            override fun onError(error: ImageCaptureException) {
                // 在此处插入代码
            }
            override fun onImageSaved(outputFileResults: ImageCapture.OutputFileResults) {
                // 在此处插入代码
            }
        })
}
```



### Java

```java
public void onClick() {
    ImageCapture.OutputFileOptions outputFileOptions =
            new ImageCapture.OutputFileOptions.Builder(new File(...)).build();
    imageCapture.takePicture(outputFileOptions, cameraExecutor,
        new ImageCapture.OnImageSavedCallback() {
            @Override
            public void onImageSaved(ImageCapture.OutputFileResults outputFileResults) {
                // 在此处插入代码
            }
            @Override
            public void onError(ImageCaptureException error) {
                // 在此处插入代码
            }
       }
    );
}
```

以下是此代码片段的关键点：

  * `ImageCapture.OutputFileOptions` 允许你配置保存位置和元数据。

    * 在这里，`OutputFileOptions.Builder()` 使用 `File` 对象来确定保存位置。
  * `takePicture()` 函数使用提供的选项和执行器异步捕获图像。
  * `OnImageSavedCallback` 提供成功和失败的回调。

    * `onImageSaved()` 回调处理成功的图像捕获并提供对保存图像结果的访问。
    * `onError()` 回调处理图像捕获错误。


以下是使用 `ImageCapture` 配置设备相机的其他方法。你可以使用 `ImageCapture.Builder` 方法来实现这些配置。

## 设置捕获模式

使用 `ImageCapture.Builder.setCaptureMode()` 配置拍照时的捕获模式：

  * `CAPTURE_MODE_MINIMIZE_LATENCY` ：优化图像捕获以减少延迟。
  * `CAPTURE_MODE_MAXIMIZE_QUALITY` ：优化图像捕获以提高图像质量。

捕获模式默认为 `CAPTURE_MODE_MINIMIZE_LATENCY` 。更多详细信息，请参阅 `setCaptureMode()` 参考文档。

## 设置闪光灯模式

默认闪光灯模式为 `FLASH_MODE_OFF` 。使用 `ImageCapture.Builder.setFlashMode()` 设置闪光灯模式：

  * `FLASH_MODE_ON` ：闪光灯始终开启。
  * `FLASH_MODE_AUTO` ：在低光环境下自动开启闪光灯。


## 启用零快门延迟

从 CameraX 1.2 开始，零快门延迟（Zero-Shutter Lag）作为一种捕获模式可供使用。启用零快门延迟可以显著减少与默认捕获模式相比的延迟，确保你不会错过任何拍摄机会。

要启用零快门延迟，请将 `CAPTURE_MODE_ZERO_SHOT_LAG` 传递给 `ImageCapture.Builder.setCaptureMode()`。如果启用不成功，`setCaptureMode()` 会回退到 `CAPTURE_MODE_MINIMIZE_LATENCY`。

有关捕获模式的更多信息，请参阅图像捕获指南。

## 工作原理

零快门延迟使用一个环形缓冲区来存储最近的三帧捕获帧。当用户按下捕获按钮时，CameraX 调用 `takePicture()`，然后环形缓冲区检索与按钮按下时间戳最接近的捕获帧。CameraX 随后重新处理捕获会话，从该帧生成图像并以 JPEG 格式保存到磁盘。

## 先决条件

在启用零快门延迟之前，请使用 `isZslSupported()` 确定你的设备是否满足以下要求：

  * 支持 Android 6.0 及以上（API 级别 23 及更高）。
  * 支持 `PRIVATE` 重处理。

对于不满足最低要求的设备，CameraX 会回退到 `CAPTURE_MODE_MINIMIZE_LATENCY`。

零快门延迟仅适用于图像捕获。不能在视频捕获或相机扩展中启用它。

最后，由于使用闪光灯会导致更大的延迟，因此在闪光灯开启或处于自动模式时，零快门延迟无法工作。有关设置闪光灯模式的更多信息，请参阅 `setFlashMode()`。


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