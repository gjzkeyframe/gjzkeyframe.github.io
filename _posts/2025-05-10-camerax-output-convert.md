---
title: 探索 CameraX（10）：转换输出
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-10 18:08:08 +0800
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



CameraX 用例的输出包括缓冲区和转换信息两部分。缓冲区是一个字节数组，而转换信息是指在将缓冲区显示给最终用户之前应如何进行裁剪和旋转。如何应用转换取决于缓冲区的格式。

## ImageCapture

对于 `ImageCapture` 用例，在保存到磁盘之前会应用裁剪矩形缓冲区，并将旋转保存在 Exif 数据中。应用无需执行任何额外操作。

## Preview

对于 `Preview` 用例，可以通过调用 `SurfaceRequest.setTransformationInfoListener()` 获取转换信息。每次转换更新时，调用者会收到一个新的 `SurfaceRequest.TransformationInfo` 对象。

如何应用转换信息取决于 `Surface` 的来源，通常并不简单。如果目标仅仅是显示预览，可以使用 `PreviewView`。`PreviewView` 是一个自定义视图，自动处理转换。对于高级用途，当需要编辑预览流（例如使用 OpenGL）时，可以参考 CameraX 核心测试应用中的代码示例。

## 转换坐标

另一个常见任务是处理坐标而不是缓冲区，例如在预览中检测到的人脸周围绘制边框。在这种情况下，需要将图像分析中检测到的人脸坐标转换为预览坐标。

以下代码片段创建了一个将图像分析坐标映射到 `PreviewView` 坐标的矩阵。有关如何使用 `Matrix` 转换（x，y）坐标的更多信息，请参阅 `Matrix.mapPoints()`。

### Kotlin

```kotlin
fun getCorrectionMatrix(imageProxy: ImageProxy, previewView: PreviewView): Matrix {
    val cropRect = imageProxy.cropRect
    val rotationDegrees = imageProxy.imageInfo.rotationDegrees
    val matrix = Matrix()

    // 源顶点（裁剪矩形）按顺时针顺序的浮点数组。
    val source = floatArrayOf(
        cropRect.left.toFloat(),
        cropRect.top.toFloat(),
        cropRect.right.toFloat(),
        cropRect.top.toFloat(),
        cropRect.right.toFloat(),
        cropRect.bottom.toFloat(),
        cropRect.left.toFloat(),
        cropRect.bottom.toFloat()
    )

    // 目标顶点按顺时针顺序的浮点数组。
    val destination = floatArrayOf(
        0f,
        0f,
        previewView.width.toFloat(),
        0f,
        previewView.width.toFloat(),
        previewView.height.toFloat(),
        0f,
        previewView.height.toFloat()
    )

    // 根据旋转度数调整目标顶点。旋转度数表示纠正图像所需的顺时针旋转度数。

    // 每个顶点在顶点数组中由 2 个浮点数表示。
    val vertexSize = 2
    // 每旋转 90°，目标需要移动 1 个顶点。
    val shiftOffset = rotationDegrees / 90 * vertexSize
    val tempArray = destination.clone()
    for (toIndex in source.indices) {
        val fromIndex = (toIndex + shiftOffset) % source.size
        destination[toIndex] = tempArray[fromIndex]
    }
    matrix.setPolyToPoly(source, 0, destination, 0, 4)
    return matrix
}
```

### Java

```java
Matrix getMappingMatrix(ImageProxy imageProxy, PreviewView previewView) {
    Rect cropRect = imageProxy.getCropRect();
    int rotationDegrees = imageProxy.getImageInfo().getRotationDegrees();
    Matrix matrix = new Matrix();

    // 源顶点（裁剪矩形）按顺时针顺序的浮点数组。
    float[] source = {
        cropRect.left,
        cropRect.top,
        cropRect.right,
        cropRect.top,
        cropRect.right,
        cropRect.bottom,
        cropRect.left,
        cropRect.bottom
    };

    // 目标顶点按顺时针顺序的浮点数组。
    float[] destination = {
        0f,
        0f,
        previewView.getWidth(),
        0f,
        previewView.getWidth(),
        previewView.getHeight(),
        0f,
        previewView.getHeight()
    };

    // 根据旋转度数调整目标顶点。
    // 旋转度数表示纠正图像所需的顺时针旋转度数。

    // 每个顶点在顶点数组中由 2 个浮点数表示。
    int vertexSize = 2;
    // 每旋转 90°，目标需要移动 1 个顶点。
    int shiftOffset = rotationDegrees / 90 * vertexSize;
    float[] tempArray = destination.clone();
    for (int toIndex = 0; toIndex < source.length; toIndex++) {
        int fromIndex = (toIndex + shiftOffset) % source.length;
        destination[toIndex] = tempArray[fromIndex];
    }
    matrix.setPolyToPoly(source, 0, destination, 0, 4);
    return matrix;
}
```



以上是文章的翻译内容，以下是原文中涉及的少量专业术语的解释：

  * **CameraX** ：是 Android Jetpack 的一部分，是一个相机库，旨在简化相机应用的开发，提供更简单易用的 API 来访问相机功能。
  * **用例** ：在 CameraX 中，用例是指特定的相机使用场景，如预览（Display）、图像捕获（ImageCapture）、图像分析（ImageAnalysis）等。
  * **缓冲区** ：这里指的是一块内存区域，用于存储相机捕获的原始图像数据。
  * **转换信息** ：指的是对捕获的图像数据进行裁剪和旋转等操作所需的信息，以便正确地显示图像。
  * **裁剪矩形** ：定义了从原始图像中裁剪出哪个区域，通常用于调整图像的显示范围或比例。
  * **Exif 数据** ：Exchangeable Image File Format 的缩写，是一种图像文件格式，用于存储数字相机所拍摄的图像的元数据，如拍摄时间、相机设置等信息。
  * **Surface** ：在 Android 中，Surface 是一个用于图形显示的缓冲区，应用程序可以将图形数据绘制到 Surface 上，然后通过 View 展示给用户。
  * **SurfaceRequest** ：用于请求一个 Surface，并提供相关的转换信息。
  * **Matrix** ：在计算机图形学中，矩阵用于表示各种变换操作，如平移、旋转、缩放等。在这里，它用于将图像分析坐标转换为预览视图坐标。



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