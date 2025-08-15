---
title: 探索 CameraX（8）：MLKit 分析器
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-08 18:08:08 +0800
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



谷歌的 ML Kit 提供了设备端机器学习视觉 API，用于检测人脸、扫描二维码、标记图像等。ML Kit Analyzer 使将 ML Kit 与你的 CameraX 应用集成变得更容易。

ML Kit Analyzer 是 `ImageAnalysis.Analyzer` 接口的实现。它根据需要覆盖默认的目标分辨率以优化 ML Kit 使用，处理坐标变换，并将帧传递给 ML Kit，后者返回聚合的分析结果。

## 实现 ML Kit Analyzer

为了实现 ML Kit Analyzer，我们建议使用 `CameraController` 类，它与 `PreviewView` 配合显示 UI 元素。当使用 `CameraController` 实现时，ML Kit Analyzer 会为你处理原始 `ImageAnalysis` 流和 `PreviewView` 之间的坐标变换。它从 CameraX 接收目标坐标系，计算坐标变换，并将其传递给 ML Kit 的 `Detector` 类进行分析。

要使用 ML Kit Analyzer 与 `CameraController`，调用 `setImageAnalysisAnalyzer()` 并传入一个 ML Kit Analyzer 对象，并在其构造函数中包含以下内容：

  * 一个 ML Kit `Detector` 列表，CameraX 将按顺序调用它们。
  * 确定 ML Kit 输出坐标的[目标坐标系](https://developers.google.com/ml-kit/vision/barcode-scanning/android)：
    * `COORDINATE_SYSTEM_VIEW_REFERENCED` ：变换后的 `PreviewView` 坐标。
    * `COORDINATE_SYSTEM_ORIGINAL` ：原始 `ImageAnalysis` 流的坐标。

  * 一个 `Executor` ，用于调用 Consumer 回调并将 `MlKitAnalyzer.Result`（或相机帧的聚合 ML Kit 结果）传递给应用。
  * 一个 `Consumer` ，CameraX 在有新的 ML Kit 输出时调用它。

以下代码使用 `CameraController` 实现 ML Kit Analyzer，设置 `BarcodeScanner` 以检测二维码：

### Kotlin

```kotlin
// 创建 BarcodeScanner 对象
val options = BarcodeScannerOptions.Builder()
  .setBarcodeFormats(Barcode.FORMAT_QR_CODE)
  .build()
val barcodeScanner = BarcodeScanning.getClient(options)

cameraController.setImageAnalysisAnalyzer(
            ContextCompat.getMainExecutor(this),
            MlKitAnalyzer(
                listOf(barcodeScanner),
                COORDINATE_SYSTEM_VIEW_REFERENCED,
                ContextCompat.getMainExecutor(this)
            ) { result: MlKitAnalyzer.Result? ->
    // result.getResult(barcodeScanner) 的值可以直接用于绘制 UI 覆盖层。
    }
)
```


### Java

```java
// 创建 BarcodeScanner 对象
BarcodeScannerOptions options = new BarcodeScannerOptions.Builder()
   .setBarcodeFormats(Barcode.FORMAT_QR_CODE)
   .build();
BarcodeScanner barcodeScanner = BarcodeScanning.getClient(options);

cameraController.setImageAnalysisAnalyzer(executor,
    new MlKitAnalyzer(List.of(barcodeScanner), COORDINATE_SYSTEM_VIEW_REFERENCED,
    executor, result -> {
   // result.getResult(barcodeScanner) 的值可以直接用于绘制 UI 覆盖层。
 });
```

在上述代码示例中，ML Kit Analyzer 将以下内容传递给 `BarcodeScanner` 的 `Detector` 类：

  * 基于 `COORDINATE_SYSTEM_VIEW_REFERENCED` 的变换矩阵，它表示目标坐标系。
  * 相机帧。

如果 `BarcodeScanner` 遇到任何问题，其 `Detector` 将抛出错误，ML Kit Analyzer 将其传播到你的应用。如果成功，ML Kit Analyzer 返回 `MLKitAnalyzer.Result#getValue()`，在这种情况下是 `Barcode` 对象。

还可以使用 `camera-core` 中的 `ImageAnalysis` 类来实现 ML Kit Analyzer。但是，由于 `ImageAnalysis` 未与 `PreviewView` 集成，因此你必须手动处理坐标变换。有关更多信息，请参阅 ML Kit Analyzer 参考文档。

## 其他资源

有关带有 ML Kit Analyzer 功能的示例相机应用，请参阅 CameraX-MLKit 示例。


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