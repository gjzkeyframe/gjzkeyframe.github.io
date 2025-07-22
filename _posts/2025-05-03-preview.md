---
title: 探索 CameraX（3）：实现预览
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-03 18:08:08 +0800
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



## 概述

当在应用中添加预览功能时，可以使用 `PreviewView`，它是一个可以裁剪、缩放和旋转以正确显示的 `View`。当相机激活时，图像预览会流式传输到 `PreviewView` 内部的表面。

## 使用 PreviewView

使用 `PreviewView` 为 CameraX 实现预览涉及以下步骤，后续部分将详细介绍这些步骤：

  1. 可选地配置 `CameraXConfig.Provider`。
  2. 将 `PreviewView` 添加到布局中。
  3. 请求 `ProcessCameraProvider`。
  4. 在视图创建时，检查 `ProcessCameraProvider` 是否可用。
  5. 选择相机并绑定生命周期和用例。

使用 `PreviewView` 存在一些限制。当使用 `PreviewView` 时，无法执行以下操作：

  * 创建 `SurfaceTexture` 并设置到 `TextureView` 和 `Preview.SurfaceProvider`。
  * 从 `TextureView` 获取 `SurfaceTexture` 并设置到 `Preview.SurfaceProvider`。
  * 从 `SurfaceView` 获取 `Surface` 并设置到 `Preview.SurfaceProvider`。

如果发生上述任何情况，则 `Preview` 将停止向 `PreviewView` 流式传输帧。

### 将 PreviewView 添加到布局中

以下示例展示了布局中的 `PreviewView`：

```xml
<FrameLayout
    android:id="@+id/container">
        <androidx.camera.view.PreviewView
            android:id="@+id/previewView" />
</FrameLayout>
```

### 请求 CameraProvider

以下代码展示了如何请求 `CameraProvider`：

### Kotlin

```kotlin
import androidx.camera.lifecycle.ProcessCameraProvider
import com.google.common.util.concurrent.ListenableFuture

class MainActivity : AppCompatActivity() {
    private lateinit var cameraProviderFuture: ListenableFuture<ProcessCameraProvider>
    override fun onCreate(savedInstanceState: Bundle?) {
        cameraProviderFuture = ProcessCameraProvider.getInstance(this)
    }
}
```

### Java

```java
import androidx.camera.lifecycle.ProcessCameraProvider
import com.google.common.util.concurrent.ListenableFuture

public class MainActivity extends AppCompatActivity {
    private ListenableFuture<ProcessCameraProvider> cameraProviderFuture;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        cameraProviderFuture = ProcessCameraProvider.getInstance(this);
    }
}
```

### 检查 CameraProvider 是否可用

在请求 `CameraProvider` 后，需在视图创建时验证其初始化是否成功。以下代码展示了如何执行此操作：

### Kotlin

```kotlin
cameraProviderFuture.addListener(Runnable {
    val cameraProvider = cameraProviderFuture.get()
    bindPreview(cameraProvider)
}, ContextCompat.getMainExecutor(this))
```

### Java

```java
cameraProviderFuture.addListener(() -> {
    try {
        ProcessCameraProvider cameraProvider = cameraProviderFuture.get();
        bindPreview(cameraProvider);
    } catch (ExecutionException | InterruptedException e) {
        // 此 Future 不需要处理错误。
        // 这里永远不应该到达。
    }
}, ContextCompat.getMainExecutor(this));
```

关于此示例中使用的 `bindPreview` 函数的示例，请参见下一节提供的代码。

### 选择相机并绑定生命周期和用例

在创建并确认 `CameraProvider` 后，执行以下操作：

  1. 创建 `Preview`。
  2. 指定所需的相机 `LensFacing` 选项。
  3. 将选定的相机和任何用例绑定到生命周期。
  4. 将 `Preview` 连接到 `PreviewView`。

以下代码展示了示例：

### Kotlin

```kotlin
fun bindPreview(cameraProvider: ProcessCameraProvider) {
    var preview: Preview = Preview.Builder()
        .build()

    var cameraSelector: CameraSelector = CameraSelector.Builder()
        .requireLensFacing(CameraSelector.LENS_FACING_BACK)
        .build()

    preview.setSurfaceProvider(previewView.getSurfaceProvider())

    var camera = cameraProvider.bindToLifecycle(this as LifecycleOwner, cameraSelector, preview)
}
```

### Java

```java
void bindPreview(@NonNull ProcessCameraProvider cameraProvider) {
    Preview preview = new Preview.Builder()
        .build();

    CameraSelector cameraSelector = new CameraSelector.Builder()
        .requireLensFacing(CameraSelector.LENS_FACING_BACK)
        .build();

    preview.setSurfaceProvider(previewView.getSurfaceProvider());

    Camera camera = cameraProvider.bindToLifecycle((LifecycleOwner) this, cameraSelector, preview);
}
```

注意，`bindToLifecycle()` 返回一个 `Camera` 对象。有关控制相机输出的更多信息（例如缩放和曝光），请参见相机输出。

现在你已完成相机预览的实现。构建应用并确认预览在应用中按预期显示和运行。

## PreviewView 的其他控制功能

CameraX 的 `PreviewView` 提供了一些额外的 API，用于配置以下属性：

  * 用于渲染预览流的实现模式。
  * 预览图像的缩放类型。

### 实现模式

`PreviewView` 可以使用以下模式之一将预览流渲染到目标 `View`：

  * `PERFORMANCE` 模式，默认使用 `SurfaceView` 显示视频流，但在某些情况下会回退到 `TextureView`。`SurfaceView` 具有专用的绘图表面，更有可能通过内部硬件合成器实现硬件覆盖层，特别是当预览视频上方没有其他 UI 元素（如按钮）时。通过硬件覆盖层渲染，视频帧可以避免 GPU 路径，从而减少平台功耗和延迟。
  * `COMPATIBLE` 模式，使用 `TextureView`，与 `SurfaceView` 不同，它没有专用的绘图表面。因此，视频以混合方式渲染以便显示。在此额外步骤期间，应用可以执行额外处理，例如无限制地缩放和旋转视频。

使用 `PreviewView.setImplementationMode()` 选择适合应用的实现模式。如果默认的 `PERFORMANCE` 模式不适合你的应用，以下代码示例展示了如何设置 `COMPATIBLE` 模式：

### Kotlin

```kotlin
// viewFinder 是一个 PreviewView 实例
viewFinder.implementationMode = PreviewView.ImplementationMode.COMPATIBLE
```

### 缩放类型

当预览视频分辨率与目标 `PreviewView` 的尺寸不同时，需要通过裁剪或信箱式显示（保持原始纵横比）将视频内容适配到视图。`PreviewView` 提供了以下 `ScaleTypes` 用于此目的：

  * `FIT_CENTER`、`FIT_START` 和 `FIT_END` 用于信箱式显示。视频内容会按比例缩放（放大或缩小）到可以在目标 `PreviewView` 中显示的最大尺寸。但是，虽然整个视频帧可见，但屏幕的某些部分可能会留白。根据选择的三种缩放类型中的哪一种，视频帧会相对于目标视图的中心、起始位置或结束位置对齐。
  * `FILL_CENTER`、`FILL_START` 和 `FILL_END` 用于裁剪。如果视频与 `PreviewView` 的纵横比不匹配，只有部分内容可见，但视频会填满整个 `PreviewView`。

CameraX 默认使用的缩放类型为 `FILL_CENTER`。使用 `PreviewView.setScaleType()` 设置最适合应用的缩放类型。以下代码示例设置了 `FIT_CENTER` 缩放类型：

### Kotlin

```kotlin
// viewFinder 是一个 PreviewView 实例
viewFinder.scaleType = PreviewView.ScaleType.FIT_CENTER
```

显示视频的过程包括以下步骤：

  1. 缩放视频：
     * 对于 `FIT_*` 缩放类型，使用 `min(dst.width/src.width, dst.height/src.height)` 缩放视频。
     * 对于 `FILL_*` 缩放类型，使用 `max(dst.width/src.width, dst.height/src.height)` 缩放视频。

  2. 将缩放后的视频与目标 `PreviewView` 对齐：

     * 对于 `FIT_CENTER` / `FILL_CENTER`，将缩放后的视频与目标 `PreviewView` 居中对齐。
     * 对于 `FIT_START` / `FILL_START`，将缩放后的视频与目标 `PreviewView` 的左上角对齐。
     * 对于 `FIT_END` / `FILL_END`，将缩放后的视频与目标 `PreviewView` 的右下角对齐。

例如，这是一个 640x480 的源视频和一个 1920x1080 的目标 `PreviewView` ：

![img](https://developer.android.com/static/images/training/camera/camerax/camera-preview/camera_preview_view_scale_type_default.png)

下图展示了 `FIT_START` / `FIT_CENTER` / `FIT_END` 的缩放过程：

![img](https://developer.android.com/static/images/training/camera/camerax/camera-preview/camera_preview_view_scale_type_fit.png)

过程如下：

  1. 使用 `min(1920/640, 1080/480) = 2.25` 按比例缩放视频帧（保持原始纵横比），得到一个 1440x1080 的中间视频帧。
  2. 将 1440x1080 的视频帧与 1920x1080 的 `PreviewView` 对齐。

     * 对于 `FIT_CENTER`，将视频帧与 `PreviewView` 窗口的 **中心** 对齐。`PreviewView` 的起始和结束 240 像素列为空白。
     * 对于 `FIT_START`，将视频帧与 `PreviewView` 窗口的 **起始位置**（左上角）对齐。`PreviewView` 的结束 480 像素列为空白。
     * 对于 `FIT_END`，将视频帧与 `PreviewView` 窗口的 **结束位置**（右下角）对齐。`PreviewView` 的起始 480 像素列为空白。

下图展示了 `FILL_START` / `FILL_CENTER` / `FILL_END` 的缩放过程：

![img](https://developer.android.com/static/images/training/camera/camerax/camera-preview/camera_preview_view_scale_type_fill.png)

过程如下：

  1. 使用 `max(1920/640, 1080/480) = 3` 缩放视频帧，得到一个 1920x1440 的中间视频帧（大于 `PreviewView` 的尺寸）。
  2. 将 1920x1440 的视频帧裁剪以适应 1920x1080 的 `PreviewView` 窗口。

     * 对于 `FILL_CENTER`，从 1920x1440 的缩放视频的 **中心** 裁剪 1920x1080。视频的顶部和底部 180 行不可见。
     * 对于 `FILL_START`，从 1920x1440 的缩放视频的 **起始位置** 裁剪 1920x1080。视频的底部 360 行不可见。
     * 对于 `FILL_END`，从 1920x1440 的缩放视频的 **结束位置** 裁剪 1920x1080。视频的顶部 360 行不可见。

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