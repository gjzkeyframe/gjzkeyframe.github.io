---
title: 探索 CameraX（7）：扩展 API
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-07 18:08:08 +0800
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



CameraX 提供了一个扩展 API，用于访问设备制造商在各种 Android 设备上实现的扩展功能。有关支持的扩展模式列表，请参阅相机扩展。

有关支持扩展的设备列表，请参阅支持的设备。

## 扩展架构

下图显示了相机扩展架构。

![Camera Extensions 架构](https://developer.android.com/static/images/training/camera/camerax/camerax-camera-extensions-architecture.png)**图 1.** Camera Extensions 架构

CameraX 应用可以通过 CameraX 扩展 API 使用扩展功能。CameraX 扩展 API 管理查询可用扩展、配置扩展相机会话以及与 Camera Extensions OEM 库通信。这允许你的应用使用诸如夜景、HDR、自动、虚化或面部修饰等功能。

## 为图像捕获和预览启用扩展

在使用扩展 API 之前，请使用 ExtensionsManager.getInstanceAsync(Context, CameraProvider) 方法获取 ExtensionsManager 实例。这将允许你查询扩展可用性信息。然后获取启用了扩展的 CameraSelector。当使用启用了扩展的 CameraSelector 调用 bindToLifecycle() 方法时，扩展模式将应用于图像捕获和预览用例。

有关为图像捕获和预览用例实现扩展的代码示例，请参见下文：

### Kotlin

```kotlin
import androidx.camera.extensions.ExtensionMode
import androidx.camera.extensions.ExtensionsManager

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val lifecycleOwner = this

    val cameraProviderFuture = ProcessCameraProvider.getInstance(applicationContext)
    cameraProviderFuture.addListener({
        // 获取进程相机提供程序实例
        // 相机提供程序提供对设备相关相机集的访问。
        // 从提供程序获取的相机将绑定到活动生命周期。
        val cameraProvider = cameraProviderFuture.get()

        val extensionsManagerFuture =
            ExtensionsManager.getInstanceAsync(applicationContext, cameraProvider)
        extensionsManagerFuture.addListener({
            // 获取扩展管理器实例
            // 扩展管理器使相机能够使用设备上可用的扩展功能。
            val extensionsManager = extensionsManagerFuture.get()

            // 选择相机
            val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA

            // 查询扩展是否可用。
            // 并非所有设备都支持扩展，或者可能只支持部分扩展。
            if (extensionsManager.isExtensionAvailable(cameraSelector, ExtensionMode.NIGHT)) {
                // 在启用不同扩展模式之前，先解除绑定所有用例。
                try {
                    cameraProvider.unbindAll()

                    // 获取启用了夜景扩展的相机选择器
                    val nightCameraSelector =
                        extensionsManager.getExtensionEnabledCameraSelector(
                            cameraSelector,
                            ExtensionMode.NIGHT
                        )

                    // 使用启用了扩展的相机选择器绑定图像捕获和预览用例。
                    val imageCapture = ImageCapture.Builder().build()
                    val preview = Preview.Builder().build()
                    // 将预览连接到接收相机输出帧的表面。
                    // 这将允许在 TextureView 或 SurfaceView 中显示相机帧。
                    // SurfaceProvider 可以从 PreviewView 获得。
                    preview.setSurfaceProvider(surfaceProvider)

                    // 返回绑定到生命周期的相机实例
                    // 使用此相机对象控制相机的各种操作
                    // 示例：闪光灯、缩放、对焦测光等。
                    val camera = cameraProvider.bindToLifecycle(
                        lifecycleOwner,
                        nightCameraSelector,
                        imageCapture,
                        preview
                    )
                } catch (e: Exception) {
                    Log.e(TAG, "用例绑定失败", e)
                }
            }
        }, ContextCompat.getMainExecutor(this))
    }, ContextCompat.getMainExecutor(this))
}
```


### Java

```java
import androidx.camera.extensions.ExtensionMode;
import androidx.camera.extensions.ExtensionsManager;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    final LifecycleOwner lifecycleOwner = this;

    final ListenableFuture<ProcessCameraProvider> cameraProviderFuture =
            ProcessCameraProvider.getInstance(getApplicationContext());

    cameraProviderFuture.addListener(() -> {
        try {
            // 获取进程相机提供程序实例
            // 相机提供程序提供对设备相关相机集的访问。
            // 从提供程序获取的相机将绑定到活动生命周期。
            final ProcessCameraProvider cameraProvider = cameraProviderFuture.get();

            final ListenableFuture<ExtensionsManager> extensionsManagerFuture =
                    ExtensionsManager.getInstanceAsync(getApplicationContext(), cameraProvider);
            extensionsManagerFuture.addListener(() -> {
                // 获取扩展管理器实例
                // 扩展管理器使相机能够使用设备上可用的扩展功能。
                try {
                    final ExtensionsManager extensionsManager = extensionsManagerFuture.get();

                    // 选择相机
                    final CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;

                    // 查询扩展是否可用。
                    // 并非所有设备都支持扩展，或者可能只支持部分扩展。
                    if (extensionsManager.isExtensionAvailable(
                            cameraSelector,
                            ExtensionMode.NIGHT
                    )) {
                        // 在启用不同扩展模式之前，先解除绑定所有用例。
                        cameraProvider.unbindAll();

                        // 获取启用了扩展的相机选择器
                        final CameraSelector nightCameraSelector = extensionsManager
                                .getExtensionEnabledCameraSelector(cameraSelector, ExtensionMode.NIGHT);

                        // 使用启用了扩展的相机选择器绑定图像捕获和预览用例。
                        final ImageCapture imageCapture = new ImageCapture.Builder().build();
                        final Preview preview = new Preview.Builder().build();
                        // 将预览连接到接收相机输出帧的表面。
                        // 这将允许在 TextureView 或 SurfaceView 中显示相机帧。
                        // SurfaceProvider 可以从 PreviewView 获得。
                        preview.setSurfaceProvider(surfaceProvider);

                        cameraProvider.bindToLifecycle(
                                lifecycleOwner,
                                nightCameraSelector,
                                imageCapture,
                                preview
                        );
                    }
                } catch (ExecutionException | InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }, ContextCompat.getMainExecutor(this));

        } catch (ExecutionException | InterruptedException e) {
            throw new RuntimeException(e);
        }

    }, ContextCompat.getMainExecutor(this));
}
```


## 禁用扩展

要禁用供应商扩展，请解除绑定所有用例，然后使用普通相机选择器重新绑定图像捕获和预览用例。例如，使用 `CameraSelector.DEFAULT_BACK_CAMERA` 重新绑定到后置相机。

## 依赖项

CameraX 扩展 API 实现在 `camera-extensions` 库中。扩展功能依赖于 CameraX 核心模块（`core`、`camera2`、`lifecycle`）。

### Groovy

```groovy
dependencies {
  def camerax_version = "1.2.0-rc01"
  implementation "androidx.camera:camera-core:${camerax_version}"
  implementation "androidx.camera:camera-camera2:${camerax_version}"
  implementation "androidx.camera:camera-lifecycle:${camerax_version}"
  // CameraX 扩展库
  implementation "androidx.camera:camera-extensions:${camerax_version}"
    ...
}
```

### Kotlin

```kotlin
dependencies {
  val camerax_version = "1.2.0-rc01"
  implementation("androidx.camera:camera-core:${camerax_version}")
  implementation("androidx.camera:camera-camera2:${camerax_version}")
  implementation("androidx.camera:camera-lifecycle:${camerax_version}")
  // CameraX 扩展库
  implementation("androidx.camera:camera-extensions:${camerax_version}")
    ...
}
```

## 旧版 API 移除

随着在 `1.0.0-alpha26` 中发布的新版扩展 API，2019 年 8 月发布的旧版扩展 API 现已弃用。从版本 `1.0.0-alpha28` 开始，旧版扩展 API 已从库中移除。使用新版扩展 API 的应用现在必须获取启用了扩展的 `CameraSelector`，并用它来绑定用例。

使用旧版扩展 API 的应用应迁移到新版扩展 API，以确保与未来的 CameraX 发布版本的兼容性。

## 其他资源

要了解更多关于 CameraX 的信息，请参阅以下额外资源：

  * Codelab ：[开始使用 CameraX](https://developer.android.com/camerax)
  * 代码示例 ：[CameraX 扩展示例应用](https://github.com/android/CameraX-Samples)


---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }