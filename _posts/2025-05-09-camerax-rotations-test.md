---
title: 探索 CameraX（9）：用例旋转
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-09 18:08:08 +0800
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



## 如何确定目标旋转

以下示例展示了如何根据设备的自然方向确定目标旋转。

### 示例 1：肖像自然方向

| 设备示例：Pixel 3 XL|
| ---|
| 自然方向 = 肖像<br>当前方向 = 肖像<br>显示旋转 = 0<br>目标旋转 = 0| ![示例1-1](https://developer.android.com/static/images/training/camera/camerax/rotations-tests/rotations-tests-02.png)|
| 自然方向 = 肖像<br>当前方向 = 横向<br>显示旋转 = 90<br>目标旋转 = 90| ![示例1-2](https://developer.android.com/static/images/training/camera/camerax/rotations-tests/rotations-tests-03.png)|

### 示例 2：横向自然方向

| 设备示例：Pixel C|
| ---|
| 自然方向 = 横向<br>当前方向 = 横向<br>显示旋转 = 0<br>目标旋转 = 0| ![示例2-1](https://developer.android.com/static/images/training/camera/camerax/rotations-tests/rotations-tests-04.png)|
| 自然方向 = 横向<br>当前方向 = 肖像<br>显示旋转 = 270<br>目标旋转 = 270| ![示例2-2](https://developer.android.com/static/images/training/camera/camerax/rotations-tests/rotations-tests-05.png)|

## 图像旋转

传感器方向在 Android 中定义为一个常数值，表示当设备处于自然位置时，传感器相对于设备顶部顺时针旋转的度数（0、90、180、270）。在下图的所有示例中，图像旋转描述了数据应如何顺时针旋转以保持直立。

以下示例展示了根据相机传感器方向图像旋转应如何变化。它们还假设目标旋转设置为显示旋转。

### 示例 1：传感器旋转 90 度

| 设备示例：Pixel 3 XL|
| ---|
| 显示旋转 = 0<br>显示方向 = 肖像<br>图像旋转 = 90| ![示例1-1](https://developer.android.com/static/images/training/camera/camerax/rotations-tests/rotations-tests-06.svg)|
| 显示旋转 = 90<br>显示方向 = 横向<br>图像旋转 = 0| ![示例1-2](https://developer.android.com/static/images/training/camera/camerax/rotations-tests/rotations-tests-07.svg)|

### 示例 2：传感器旋转 270 度

| 设备示例：Nexus 5X|
| ---|
| 显示旋转 = 0<br>显示方向 = 肖像<br>图像旋转 = 270| |
| 显示旋转 = 90<br>显示方向 = 横向<br>图像旋转 = 180| |

### 示例 3：传感器旋转 0 度

| 设备示例：Pixel C（平板电脑）|
| ---|
| 显示旋转 = 0<br>显示方向 = 横向<br>图像旋转 = 0| |
| 显示旋转 = 270<br>显示方向 = 肖像<br>图像旋转 = 90| |

## 计算图像旋转

### ImageAnalysis

`ImageAnalysis` 的 `Analyzer` 以 `ImageProxy` 的形式接收来自相机的图像。每张图像都包含可通过以下方式访问的旋转信息：

```kotlin
val rotation = imageProxy.imageInfo.rotationDegrees
```

此值表示图像需要顺时针旋转的度数，以匹配 `ImageAnalysis` 的目标旋转。在 Android 应用的上下文中，`ImageAnalysis` 的目标旋转通常与屏幕方向一致。

### ImageCapture

向 `ImageCapture` 实例附加一个回调，以指示何时准备好捕获结果。结果可以是捕获的图像或错误。

拍照时，提供的回调可以是以下类型之一：

  * **`OnImageCapturedCallback`** ：以 `ImageProxy` 的形式接收具有内存访问权限的图像。
  * **`OnImageSavedCallback`** ：在捕获的图像已成功存储在 `ImageCapture.OutputFileOptions` 指定的位置时调用。选项可以指定 `File`、`OutputStream` 或 `MediaStore` 中的位置。

无论捕获图像的格式（`ImageProxy`、`File`、`OutputStream`、`MediaStore Uri`）如何，捕获图像的旋转表示捕获图像需要顺时针旋转的度数，以匹配 `ImageCapture` 的目标旋转，这在 Android 应用的上下文中，通常与屏幕方向一致。

可以通过以下方式之一检索捕获图像的旋转：

`ImageProxy`

```kotlin
val rotation = imageProxy.imageInfo.rotationDegrees
```

`File`

```kotlin
val exif = Exif.createFromFile(file)
val rotation = exif.rotation
```

`OutputStream`

```kotlin
val byteArray = outputStream.toByteArray()
val exif = Exif.createFromInputStream(ByteArrayInputStream(byteArray))
val rotation = exif.rotation
```

`MediaStore uri`

```kotlin
val inputStream = contentResolver.openInputStream(outputFileResults.savedUri)
val exif = Exif.createFromInputStream(inputStream)
val rotation = exif.rotation
```

### 验证图像旋转

在成功捕获请求后，`ImageAnalysis` 和 `ImageCapture` 用例从相机接收 `ImageProxy`。`ImageProxy` 包裹图像及其信息，包括其旋转。此旋转信息表示图像需要旋转的度数以匹配用例的目标旋转。

## ImageCapture / ImageAnalysis 目标旋转指南

由于许多设备默认不旋转到反向肖像或反向横向，一些 Android 应用不支持这些方向。应用是否支持这些方向会改变更新用例目标旋转的方式。

以下是两个表格，定义如何使目标旋转与显示旋转同步。第一个表格展示了如何支持所有四种方向；第二个表格仅处理设备默认旋转的方向。

选择在应用中遵循哪些指南：

  1. 确认应用的相机 `Activity` 是否有锁定的方向、未锁定的方向，或者是否覆盖方向配置更改。
  2. 决定应用的相机 `Activity` 是否应处理所有四种设备方向（肖像、反向肖像、横向和反向横向），或者是否仅处理设备默认支持的方向。

### 支持所有四种方向

| 场景| 指南| 单窗口模式| 多窗口分屏模式|
| ---| ---| ---| ---|
| 未锁定方向| 每次创建 `Activity` 时设置用例，例如在 `Activity` 的 `onCreate()` 回调中。| | |
| 使用 `OrientationEventListener` 的 `onOrientationChanged()`。在回调中，更新用例的目标旋转。这处理了即使在方向更改后系统也不会重新创建 `Activity` 的情况，例如当设备旋转 180 度时。| 还处理显示处于反向肖像方向且设备默认不旋转到反向肖像的情况。| 还处理设备旋转时（例如 90 度）`Activity` 不会被重新创建的情况。这在小尺寸设备上应用占据屏幕一半，以及在大尺寸设备上占据三分之二屏幕时发生。|
| 可选：在 `AndroidManifest` 文件中将 `Activity` 的 `screenOrientation` 属性设置为 `fullSensor` 。| 这允许在设备处于反向肖像方向时 UI 保持直立，并允许系统在设备旋转 90 度时重新创建 `Activity` 。| 对默认不旋转到反向肖像的设备无效。多窗口模式不支持在显示处于反向肖像方向时使用。|
| 锁定方向| 仅在 `Activity` 首次创建时设置用例，例如在 `Activity` 的 `onCreate()` 回调中。| | |
| 使用 `OrientationEventListener` 的 `onOrientationChanged()`。在回调中，更新用例的目标旋转。| | 还处理设备旋转时（例如 90 度）`Activity` 不会被重新创建的情况。这在小尺寸设备上应用占据屏幕一半，以及在大尺寸设备上占据三分之二屏幕时发生。|
| 覆盖方向配置更改| 仅在 `Activity` 首次创建时设置用例，例如在 `Activity` 的 `onCreate()` 回调中。| | |
| 使用 `OrientationEventListener` 的 `onOrientationChanged()`。在回调中，更新用例的目标旋转。| | 还处理设备旋转时（例如 90 度）`Activity` 不会被重新创建的情况。这在小尺寸设备上应用占据屏幕一半，以及在大尺寸设备上占据三分之二屏幕时发生。|
| 可选：在 `AndroidManifest` 文件中将 Activity 的 screenOrientation 属性设置为 fullSensor 。| 允许在设备处于反向肖像方向时 UI 保持直立。| 对默认不旋转到反向肖像的设备无效。多窗口模式不支持在显示处于反向肖像方向时使用。|

### 仅支持设备默认方向

仅支持设备默认支持的方向（可能包括或不包括反向肖像 / 反向横向）。

| 场景| 指南| 多窗口分屏模式|
| ---| ---| ---|
| 未锁定方向| 每次创建 `Activity` 时设置用例，例如在 `Activity` 的 `onCreate()` 回调中。| |
| 使用 `DisplayListener` 的 `onDisplayChanged()`。在回调中，更新用例的目标旋转，例如当设备旋转 180 度时。| 还处理设备旋转时（例如 90 度）`Activity` 不会被重新创建的情况。这在小尺寸设备上应用占据屏幕一半，以及在大尺寸设备上占据三分之二屏幕时发生。|
| 锁定方向| 仅在 `Activity` 首次创建时设置用例，例如在 `Activity` 的 `onCreate()` 回调中。| |
| 使用 `OrientationEventListener` 的 `onOrientationChanged()`。在回调中，更新用例的目标旋转。| 还处理设备旋转时（例如 90 度）`Activity` 不会被重新创建的情况。这在小尺寸设备上应用占据屏幕一半，以及在大尺寸设备上占据三分之二屏幕时发生。|
| 覆盖方向配置更改| 仅在 `Activity` 首次创建时设置用例，例如在 `Activity` 的 `onCreate()` 回调中。| |
| 使用 `DisplayListener` 的 `onDisplayChanged()`。在回调中，更新用例的目标旋转，例如当设备旋转 180 度时。| 还处理设备旋转时（例如 90 度）`Activity` 不会被重新创建的情况。这在小尺寸设备上应用占据屏幕一半，以及在大尺寸设备上占据三分之二屏幕时发生。|

### 未锁定方向

当 `Activity` 的显示方向（例如肖像或横向）与设备的物理方向匹配时，它具有未锁定的方向，反向肖像 / 横向除外，一些设备默认不支持这些方向。要强制设备旋转到所有四种方向，请将 `Activity` 的 `screenOrientation` 属性设置为 `fullSensor`。

在多窗口模式下，即使 `screenOrientation` 属性设置为 `fullSensor`，默认不支持反向肖像 / 横向的设备也不会旋转到反向肖像 / 横向。

```
<!-- 该 Activity 具有未锁定的方向，但如果设备默认不支持反向肖像 / 横向，在单窗口模式下可能不会旋转到这些方向。 -->
<activity android:name=".UnlockedOrientationActivity" />

<!-- 该 Activity 具有未锁定的方向，在单窗口模式下将旋转到所有四种方向。 -->
<activity
   android:name=".UnlockedOrientationActivity"
   android:screenOrientation="fullSensor" />
```

### 锁定方向

当显示保持在同一方向（例如肖像或横向）而不考虑设备的物理方向时，它具有锁定的方向。这可以通过在 `AndroidManifest.xml` 文件中的 `Activity` 声明中指定 `Activity` 的 `screenOrientation` 属性来实现。

当显示具有锁定的方向时，系统不会在设备旋转时销毁并重新创建 `Activity`。

```
<!-- 该 Activity 即使在设备旋转时也保持肖像方向。 -->
<activity
   android:name=".LockedOrientationActivity"
   android:screenOrientation="portrait" />
```

### 覆盖方向配置更改

当 `Activity` 覆盖方向配置更改时，系统不会在设备物理方向更改时销毁并重新创建它。不过，系统会更新 UI 以匹配设备的物理方向。

```
<!-- 如果设备默认不支持反向肖像 / 横向，该 Activity 的 UI 可能不会旋转到这些方向。 -->
<activity
   android:name=".OrientationConfigChangesOverriddenActivity"
   android:configChanges="orientation|screenSize" />

<!-- 在单窗口模式下，该 Activity 的 UI 将旋转到所有四种方向。 -->
<activity
   android:name=".OrientationConfigChangesOverriddenActivity"
   android:configChanges="orientation|screenSize"
   android:screenOrientation="fullSensor" />
```

### 相机用例设置

在上述场景中，相机用例可以在 `Activity` 首次创建时设置。

对于具有未锁定方向的 `Activity`，每次设备旋转时都会进行此设置，因为系统会在方向更改时销毁并重新创建 `Activity`。这导致用例每次默认将其目标旋转设置为与显示的方向匹配。

对于具有锁定方向或覆盖方向配置更改的 `Activity`，此设置仅在 `Activity` 首次创建时进行。

```
class CameraActivity : AppCompatActivity() {
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)

       val cameraProcessFuture = ProcessCameraProvider.getInstance(this)
       cameraProcessFuture.addListener(Runnable {
          val cameraProvider = cameraProcessFuture.get()

          // 默认情况下，用例将其目标旋转设置为与显示的方向匹配。
          val preview = buildPreview()
          val imageAnalysis = buildImageAnalysis()
          val imageCapture = buildImageCapture()

          cameraProvider.bindToLifecycle(
              this, cameraSelector, preview, imageAnalysis, imageCapture)
       }, mainExecutor)
   }
}
```

### OrientationEventListener 设置

使用 `OrientationEventListener` 可以在设备方向更改时持续更新相机用例的目标旋转。

```
class CameraActivity : AppCompatActivity() {

    private val orientationEventListener by lazy {
        object : OrientationEventListener(this) {
            override fun onOrientationChanged(orientation: Int) {
                if (orientation == ORIENTATION_UNKNOWN) {
                    return
                }

                val rotation = when (orientation) {
                     in 45 until 135 -> Surface.ROTATION_270
                     in 135 until 225 -> Surface.ROTATION_180
                     in 225 until 315 -> Surface.ROTATION_90
                     else -> Surface.ROTATION_0
                 }

                 imageAnalysis.targetRotation = rotation
                 imageCapture.targetRotation = rotation
            }
        }
    }

    override fun onStart() {
        super.onStart()
        orientationEventListener.enable()
    }

    override fun onStop() {
        super.onStop()
        orientationEventListener.disable()
    }
}
```

### DisplayListener 设置

使用 `DisplayListener` 可以在某些情况下更新相机用例的目标旋转，例如当设备旋转 180 度后系统不会销毁并重新创建 `Activity`。

```
class CameraActivity : AppCompatActivity() {

    private val displayListener = object : DisplayManager.DisplayListener {
        override fun onDisplayChanged(displayId: Int) {
            if (rootView.display.displayId == displayId) {
                val rotation = rootView.display.rotation
                imageAnalysis.targetRotation = rotation
                imageCapture.targetRotation = rotation
            }
        }

        override fun onDisplayAdded(displayId: Int) {
        }

        override fun onDisplayRemoved(displayId: Int) {
        }
    }

    override fun onStart() {
        super.onStart()
        val displayManager = getSystemService(Context.DISPLAY_SERVICE) as DisplayManager
        displayManager.registerDisplayListener(displayListener, null)
    }

    override fun onStop() {
        super.onStop()
        val displayManager = getSystemService(Context.DISPLAY_SERVICE) as DisplayManager
        displayManager.unregisterDisplayListener(displayListener)
    }
}
```



---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }