---
title: 探索 CameraX（11）：Camera1 迁移到 CameraX
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-11 18:08:08 +0800
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



## 迁移前的准备

### 比较 CameraX 与 Camera1 的用法

尽管代码看起来有所不同，但 Camera1 和 CameraX 的底层概念非常相似。CameraX 将常见的相机功能抽象为用例，因此开发者无需过多关注从头构建相机体验，而是更多地关注应用的差异化。

以下是在 CameraX 中的用例与 Camera1 中的概念对比：

|CameraX|Camera1|
|---|---|
|**CameraController / CameraProvider 配置**|**相机配置**|
|**↓**|**↓**|
|**Preview（预览）**|管理预览 Surface 并将其设置到相机|
|**ImageCapture（图像捕获）**|设置 PictureCallback 并在相机上调用 takePicture()|
|**VideoCapture（视频捕获）**|管理相机和 MediaRecorder 的配置顺序|
|**ImageAnalysis（图像分析）**|在预览 Surface 的基础上构建自定义分析代码|
|**↑**|**↑**|
|**设备特定代码**|处理设备旋转和缩放|
|**↑**|**↑**|
|**相机会话管理（相机选择、生命周期管理）**|相机选择和生命周期管理|

## 兼容性和性能

CameraX 支持运行 Android 5.0（API 级别 21）及更高版本的设备，覆盖了超过 98% 的现有 Android 设备。CameraX 能够自动处理设备间的差异，减少应用中的设备特定代码。

## Android 开发概念

在深入代码之前，了解以下概念很有帮助：

  * **视图绑定** ：为 XML 布局文件生成绑定类，允许在活动中轻松引用视图。
  * **异步协程** ：一种并发设计模式，可用于处理返回 `ListenableFuture` 的 CameraX 方法。

## 迁移常见场景

### 选择相机

在相机应用中，提供选择不同相机的功能通常是首要任务之一。

#### Camera1

在 Camera1 中，可以通过调用 `Camera.open()`（不带参数以打开第一个后置相机）或传入要打开的相机的整数 ID 来调用它。

##### 示例代码

```kotlin
// Camera1: 通过 ID 选择相机
private fun safeCameraOpen(id: Int): Boolean {
    return try {
        releaseCameraAndPreview()
        camera = Camera.open(id)
        true
    } catch (e: Exception) {
        Log.e(TAG, "打开相机失败", e)
        false
    }
}

private fun releaseCameraAndPreview() {
    preview?.setCamera(null)
    camera?.release()
    camera = null
}
```

#### CameraX: CameraController

在 CameraX 中，相机选择由 `CameraSelector` 类处理。你可以指定使用默认前置相机或后置相机。

##### 示例代码

```kotlin
// CameraX: 使用 CameraController 选择相机
var cameraController = LifecycleCameraController(baseContext)
val selector = CameraSelector.Builder()
    .requireLensFacing(CameraSelector.LENS_FACING_BACK).build()
cameraController.cameraSelector = selector
```

#### CameraX: CameraProvider

以下是使用 `CameraProvider` 选择默认前置相机的示例：

##### 示例代码

```kotlin
// CameraX: 使用 CameraProvider 选择相机
private suspend fun startCamera() {
    val cameraProvider = ProcessCameraProvider.getInstance(this).await()

    // 设置 UseCases（后续场景中详细介绍）
    var useCases:Array = ...

    // 设置 cameraSelector 以使用默认前置（自拍）相机
    val cameraSelector = CameraSelector.DEFAULT_FRONT_CAMERA

    try {
        // 解绑 UseCases 以重新绑定
        cameraProvider.unbindAll()

        // 将 UseCases 绑定到相机。此函数返回一个相机对象，可用于执行变焦、闪光灯和对焦等操作
        var camera = cameraProvider.bindToLifecycle(
            this, cameraSelector, useCases)
    } catch(exc: Exception) {
        Log.e(TAG, "UseCase 绑定失败", exc)
    }
}
```

### 显示预览

大多数相机应用需要在屏幕上显示相机图像流。

#### Camera1

需要自己编写实现 `android.view.SurfaceHolder.Callback` 接口的 `Preview` 类，用于将图像数据从相机硬件传递到应用。

##### 示例代码

```kotlin
// Camera1: 设置相机预览
class Preview(
    context: Context,
    private val camera: Camera
) : SurfaceView(context), SurfaceHolder.Callback {

    private val holder: SurfaceHolder = holder.apply {
        addCallback(this@Preview)
        setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS)
    }

    override fun surfaceCreated(holder: SurfaceHolder) {
        // Surface 已创建，告诉相机在哪里绘制预览
        camera.apply {
            try {
                setPreviewDisplay(holder)
                startPreview()
            } catch (e: IOException) {
                Log.d(TAG, "设置相机预览失败", e)
            }
        }
    }

    override fun surfaceDestroyed(holder: SurfaceHolder) {
        // 在活动中处理相机预览的释放
    }

    override fun surfaceChanged(holder: SurfaceHolder, format: Int, w: Int, h: Int) {
        // 如果预览可以更改或旋转，请在此处理这些事件。在更改之前确保停止预览
        if (holder.surface == null) {
            return  // 预览 Surface 不存在
        }

        // 在进行更改之前停止预览
        try {
            camera.stopPreview()
        } catch (e: Exception) {
            // 尝试停止不存在的预览；无需执行任何操作
        }

        // 设置预览大小并进行调整

        // 以新设置启动预览
        camera.apply {
            try {
                setPreviewDisplay(holder)
                startPreview()
            } catch (e: Exception) {
                Log.d(TAG, "启动相机预览失败", e)
            }
        }
    }
}

class CameraActivity : AppCompatActivity() {
    private lateinit var viewBinding: ActivityMainBinding
    private var camera: Camera? = null
    private var preview: Preview? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewBinding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(viewBinding.root)

        // 创建 Camera 实例
        camera = getCameraInstance()

        preview = camera?.let {
            // 创建 Preview 视图
            Preview(this, it)
        }

        // 将 Preview 视图设置为活动的内容
        val cameraPreview: FrameLayout = viewBinding.cameraPreview
        cameraPreview.addView(preview)
    }
}
```

#### CameraX: CameraController

如果使用 `CameraController`，则还必须使用 `PreviewView`，这意味着隐式使用了 `Preview` `UseCase`，从而减少了设置工作。

##### 示例代码

```kotlin
// CameraX: 使用 CameraController 设置相机预览
class MainActivity : AppCompatActivity() {
    private lateinit var viewBinding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewBinding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(viewBinding.root)

        // 创建 CameraController 并将其设置到 PreviewView
        var cameraController = LifecycleCameraController(baseContext)
        cameraController.bindToLifecycle(this)
        val previewView: PreviewView = viewBinding.cameraPreview
        previewView.controller = cameraController
    }
}
```

#### CameraX: CameraProvider

使用 CameraX 的 `CameraProvider` 时，无需使用 `PreviewView`，但它仍大大简化了预览设置。

##### 示例代码

```kotlin
// CameraX: 使用 CameraProvider 设置相机预览
private suspend fun startCamera() {
    val cameraProvider = ProcessCameraProvider.getInstance(this).await()

    // 创建 Preview UseCase
    val preview = Preview.Builder()
        .build()
        .also {
            it.setSurfaceProvider(
                viewBinding.viewFinder.surfaceProvider
            )
        }

    // 选择默认后置相机
    val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA

    try {
        // 解绑 UseCases 以重新绑定
        cameraProvider.unbindAll()

        // 将 UseCases 绑定到相机。此函数返回一个相机对象，可用于执行变焦、闪光灯和对焦等操作
        var camera = cameraProvider.bindToLifecycle(
            this, cameraSelector, useCases)
    } catch(exc: Exception) {
        Log.e(TAG, "UseCase 绑定失败", exc)
    }
}
```

### 轻触对焦

当屏幕上有相机预览时，轻触预览设置对焦点是一种常见的控制方式。

#### Camera1

在 Camera1 中，必须计算最佳对焦 `Area` 以指示相机应尝试对焦的位置。此 `Area` 传递给 `setFocusAreas()`。此外，必须在相机上设置兼容的对焦模式。

##### 示例代码

```kotlin
// Camera1: 实现轻触对焦
class TapToFocusHandler : Camera.AutoFocusCallback {
    private fun handleFocus(event: MotionEvent) {
        val camera = camera ?: return
        val parameters = try {
            camera.parameters
        } catch (e: RuntimeException) {
            return
        }

        // 取消之前的自动对焦功能（如果有）
        camera.cancelAutoFocus()

        // 创建对焦 Area
        val rect = calculateFocusAreaCoordinates(event.x, event.y)
        val weight = 1  // 由于只有一个 Area，所以权重值不重要
        val focusArea = Camera.Area(rect, weight)

        // 设置对焦参数
        parameters.focusMode = Parameters.FOCUS_MODE_AUTO
        parameters.focusAreas = listOf(focusArea)

        // 将参数设置回相机并启动自动对焦
        camera.parameters = parameters
        camera.autoFocus(this)
    }

    private fun calculateFocusAreaCoordinates(x: Int, y: Int): Rect {
        // 定义要返回的 Area 的大小。此值应针对应用进行优化
        val focusAreaSize = 100

        // 必须定义函数将 x 和 y 值旋转并缩放到 0 到 1 之间，其中 (0, 0) 是预览的左上角，(1, 1) 是预览的右下角
        val normalizedX = (rotateAndScaleX(x) - 0.5) * 2000
        val normalizedY = (rotateAndScaleY(y) - 0.5) * 2000

        // 计算要返回的 Rect 的左、上、右、下值。如果 Rect 超出允许的范围（-1000, -1000, 1000, 1000），则裁剪值以使其在边界内
        val left = max(normalizedX - focusAreaSize / 2, -1000)
        val top = max(normalizedY - focusAreaSize / 2, -1000)
        val right = min(left + focusAreaSize, 1000)
        val bottom = min(top + focusAreaSize, 1000)

        return Rect(left, top, left + focusAreaSize, top + focusAreaSize)
    }

    override fun onAutoFocus(focused: Boolean, camera: Camera) {
        if (!focused) {
            Log.d(TAG, "轻触对焦失败")
        }
    }
}
```

#### CameraX: CameraController

`CameraController` 监听 `PreviewView` 的触摸事件以自动处理轻触对焦。

##### 示例代码

```kotlin
// CameraX: 跟踪 PreviewView 生命周期内的轻触对焦状态
val tapToFocusStateObserver = Observer { state ->
    when (state) {
        CameraController.TAP_TO_FOCUS_NOT_STARTED ->
            Log.d(TAG, "轻触对焦初始化")
        CameraController.TAP_TO_FOCUS_STARTED ->
            Log.d(TAG, "轻触对焦开始")
        CameraController.TAP_TO_FOCUS_FOCUSED ->
            Log.d(TAG, "轻触对焦完成（对焦成功）")
        CameraController.TAP_TO_FOCUS_NOT_FOCUSED ->
            Log.d(TAG, "轻触对焦完成（对焦不成功）")
        CameraController.TAP_TO_FOCUS_FAILED ->
            Log.d(TAG, "轻触对焦失败")
    }
}

cameraController.getTapToFocusState().observe(this, tapToFocusStateObserver)
```

#### CameraX: CameraProvider

使用 `CameraProvider` 时，需要一些设置才能使轻触对焦工作。

##### 示例代码

```kotlin
// CameraX: 使用 CameraProvider 实现轻触对焦
val gestureDetector = GestureDetectorCompat(context,
    object : SimpleOnGestureListener() {
        override fun onSingleTapUp(e: MotionEvent): Boolean {
            val previewView = previewView ?: return false
            val camera = camera ?: return false
            val meteringPointFactory = previewView.meteringPointFactory
            val focusPoint = meteringPointFactory.createPoint(e.x, e.y)
            val meteringAction = FocusMeteringAction.Builder(focusPoint).build()
            lifecycleScope.launch {
                val focusResult = camera.cameraControl.startFocusAndMetering(meteringAction).await()
                if (!focusResult.isFocusSuccessful()) {
                    Log.d(TAG, "轻触对焦失败")
                }
            }
            return true
        }
    }
)

// 在 PreviewView 的触摸监听器中设置手势检测器
previewView.setOnTouchListener { _, event ->
    var didConsume = scaleGestureDetector.onTouchEvent(event)
    if (!scaleGestureDetector.isInProgress) {
        didConsume = gestureDetector.onTouchEvent(event)
    }
    didConsume
}
```

### 捏合缩放

实现预览的缩放是常见的直接操作之一。

#### Camera1

使用 Camera1 有两种缩放方式：`Camera.startSmoothZoom()` 和 `Camera.Parameters.setZoom()`。

##### 示例代码

```kotlin
// Camera1: 实现捏合缩放
val scaleGestureDetector = ScaleGestureDetector(context,
    object : ScaleGestureDetector.OnScaleGestureListener {
        override fun onScale(detector: ScaleGestureDetector): Boolean {
            val camera = camera ?: return false
            val parameters = try {
                camera.parameters
            } catch (e: RuntimeException) {
                return false
            }

            // 如果有任何对焦正在进行，则停止对焦
            camera.cancelAutoFocus()

            // 在 Camera.Parameters 上设置缩放级别，并将参数设置回相机
            val currentZoom = parameters.zoom
            parameters.zoom = (detector.scaleFactor * currentZoom).toInt()
            camera.parameters = parameters
            return true
        }
    }
)

class ZoomTouchListener : View.OnTouchListener {
    override fun onTouch(v: View, event: MotionEvent): Boolean =
        scaleGestureDetector.onTouchEvent(event)
}

// 如果当前相机支持缩放，则为预览视图设置 ZoomTouchListener 以处理触摸事件
if (camera.parameters.isZoomSupported) {
    view.setOnTouchListener(ZoomTouchListener())
}
```

#### CameraX: CameraController

类似于轻触对焦，`CameraController` 监听 `PreviewView` 的触摸事件以自动处理捏合缩放。

##### 示例代码

```kotlin
// CameraX: 跟踪 PreviewView 生命周期内的捏合缩放状态
val pinchToZoomStateObserver = Observer { state ->
    val zoomRatio = state.zoomRatio
    Log.d(TAG, "捏合缩放比例 $zoomRatio")
}

cameraController.zoomState.observe(this, pinchToZoomStateObserver)
```

#### CameraX: CameraProvider

要使 `CameraProvider` 的捏合缩放工作，需要进行一些设置。

##### 示例代码

```kotlin
// CameraX: 使用 CameraProvider 实现捏合缩放
val scaleGestureDetector = ScaleGestureDetector(context,
    object : SimpleOnGestureListener() {
        override fun onScale(detector: ScaleGestureDetector): Boolean {
            val camera = camera ?: return false
            val zoomState = camera.cameraInfo.zoomState
            val currentZoomRatio = zoomState.value?.zoomRatio ?: 1f
            camera.cameraControl.setZoomRatio(detector.scaleFactor * currentZoomRatio)
            return true
        }
    }
)

// 在 PreviewView 的触摸监听器中设置缩放手势检测器
previewView.setOnTouchListener { _, event ->
    var didConsume = scaleGestureDetector.onTouchEvent(event)
    if (!scaleGestureDetector.isInProgress) {
        didConsume = gestureDetector.onTouchEvent(event)
    }
    didConsume
}
```

### 拍照

此部分介绍如何触发拍照，无论是在按下快门按钮、计时器到期还是其他事件时。

#### Camera1

在 Camera1 中，首先定义一个 `Camera.PictureCallback` 来在请求图片数据时进行处理。

##### 示例代码

```kotlin
// Camera1: 定义一个处理 JPEG 数据的 Camera.PictureCallback
private val picture = Camera.PictureCallback { data, _ ->
    val pictureFile: File = getOutputMediaFile(MEDIA_TYPE_IMAGE) ?: run {
        Log.d(TAG, "创建媒体文件失败，请检查存储权限")
        return@PictureCallback
    }

    try {
        val fos = FileOutputStream(pictureFile)
        fos.write(data)
        fos.close()
    } catch (e: FileNotFoundException) {
        Log.d(TAG, "文件未找到", e)
    } catch (e: IOException) {
        Log.d(TAG, "访问文件失败", e)
    }
}
```

然后，调用相机实例上的 `takePicture()` 方法。

##### 示例代码

```kotlin
// Camera1: 调用相机实例上的 takePicture() 方法，并传入我们的 PictureCallback
camera?.takePicture(null, null, picture)
```

#### CameraX: CameraController

CameraX 的 `CameraController` 通过实现自己的 `takePicture()` 方法，简化了图像捕获的流程。

##### 示例代码

```kotlin
// CameraX: 定义一个使用 CameraController 拍照的函数
private val FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS"

private fun takePhoto() {
    // 创建时间戳名称和 MediaStore 条目
    val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US).format(System.currentTimeMillis())
    val contentValues = ContentValues().apply {
        put(MediaStore.MediaColumns.DISPLAY_NAME, name)
        put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
            put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/CameraX-Image")
        }
    }

    // 创建输出选项对象，包含文件和元数据
    val outputOptions = ImageCapture.OutputFileOptions.Builder(
        context.contentResolver,
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
        contentValues
    ).build()

    // 设置图像捕获监听器，在拍照后触发
    cameraController.takePicture(
        outputOptions,
        ContextCompat.getMainExecutor(this),
        object : ImageCapture.OnImageSavedCallback {
            override fun onError(e: ImageCaptureException) {
                Log.e(TAG, "拍照失败", e)
            }

            override fun onImageSaved(output: ImageCapture.OutputFileResults) {
                val msg = "拍照成功：${output.savedUri}"
                Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
                Log.d(TAG, msg)
            }
        }
    )
}
```

#### CameraX: CameraProvider

使用 `CameraProvider` 拍照的方式与使用 `CameraController` 几乎完全相同，但需要先创建并绑定一个 `ImageCapture` `UseCase`，以便有一个可以调用 `takePicture()` 的对象。

##### 示例代码

```kotlin
// CameraX: 创建并绑定一个 ImageCapture UseCase
private var imageCapture: ImageCapture? = null

imageCapture = ImageCapture.Builder().build()

var camera = cameraProvider.bindToLifecycle(
    this, cameraSelector, preview, imageCapture)
```

然后，调用 `ImageCapture.takePicture()`。

##### 示例代码

```kotlin
// CameraX: 定义一个使用 CameraController 拍照的函数
private fun takePhoto() {
    val imageCapture = imageCapture ?: return
    imageCapture.takePicture(...)
}
```

### 录制视频

录制视频比之前介绍的场景复杂得多。CameraX 同样为你处理了大部分复杂性。

#### Camera1

使用 Camera1 进行视频录制需要仔细管理相机和 MediaRecorder，并且方法调用必须按照特定顺序进行。

##### 示例代码

```kotlin
// Camera1: 设置 MediaRecorder 并定义开始和停止视频录制的函数
private var mediaRecorder: MediaRecorder? = null
private var isRecording = false

private fun prepareMediaRecorder(): Boolean {
    mediaRecorder = MediaRecorder()

    // 解锁相机并将其设置给 MediaRecorder
    camera?.unlock()

    mediaRecorder?.run {
        setCamera(camera)

        // 设置音频和视频源
        setAudioSource(MediaRecorder.AudioSource.CAMCORDER)
        setVideoSource(MediaRecorder.VideoSource.CAMERA)

        // 设置 CamcorderProfile（需要 API 级别 8 或更高）
        setProfile(CamcorderProfile.get(CamcorderProfile.QUALITY_HIGH))

        // 设置输出文件
        setOutputFile(getOutputMediaFile(MEDIA_TYPE_VIDEO).toString())

        // 设置预览输出
        setPreviewDisplay(preview?.holder?.surface)

        setOutputFormat(MediaRecorder.OutputFormat.MPEG_4)
        setAudioEncoder(MediaRecorder.AudioEncoder.DEFAULT)
        setVideoEncoder(MediaRecorder.VideoEncoder.DEFAULT)

        // 准备配置的 MediaRecorder
        return try {
            prepare()
            true
        } catch (e: IllegalStateException) {
            Log.d(TAG, "准备 MediaRecorder 失败", e)
            releaseMediaRecorder()
            false
        } catch (e: IOException) {
            Log.d(TAG, "设置 MediaRecorder 文件失败", e)
            releaseMediaRecorder()
            false
        }
    }
    return false
}

private fun releaseMediaRecorder() {
    mediaRecorder?.reset()
    mediaRecorder?.release()
    mediaRecorder = null
    camera?.lock()
}

private fun startStopVideo() {
    if (isRecording) {
        // 停止录制并释放相机
        mediaRecorder?.stop()
        releaseMediaRecorder()
        camera?.lock()
        isRecording = false

        // 这里是通知用户视频录制已停止的好地方
    } else {
        // 初始化视频相机
        if (prepareVideoRecorder()) {
            // 相机可用且已解锁，MediaRecorder 已准备好，现在可以开始录制
            mediaRecorder?.start()
            isRecording = true

            // 这里是通知用户录制已开始的好地方
        } else {
            // 准备失败，释放相机
            releaseMediaRecorder()

            // 这里通知用户
        }
    }
}
```

#### CameraX: CameraController

使用 CameraX 的 `CameraController`，可以独立切换 `ImageCapture`、`VideoCapture` 和 `ImageAnalysis` `UseCase`，只要这些用例可以同时使用。默认情况下启用了 `ImageCapture` 和 `ImageAnalysis` `UseCase`，这就是为什么拍照时不需要调用 `setEnabledUseCases()`。

要使用 `CameraController` 进行视频录制，需要先调用 `setEnabledUseCases()` 启用 `VideoCapture` `UseCase`。

##### 示例代码

```kotlin
// CameraX: 在 CameraController 上启用 VideoCapture UseCase
cameraController.setEnabledUseCases(VIDEO_CAPTURE)
```

当需要开始录制视频时，可以调用 `CameraController.startRecording()` 函数。

##### 示例代码

```kotlin
// CameraX: 使用 CameraController 实现视频捕获
private val FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS"

class VideoSaveCallback : OnVideoSavedCallback {
    override fun onVideoSaved(outputFileResults: OutputFileResults) {
        val msg = "视频捕获成功：${outputFileResults.savedUri}"
        Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
        Log.d(TAG, msg)
    }

    override fun onError(videoCaptureError: Int, message: String, cause: Throwable?) {
        Log.d(TAG, "保存视频失败：$message", cause)
    }
}

private fun startStopVideo() {
    if (cameraController.isRecording()) {
        // 停止当前录制会话
        cameraController.stopRecording()
        return
    }

    // 定义保存视频的文件选项
    val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US).format(System.currentTimeMillis())
    val outputFileOptions = OutputFileOptions.Builder(File(this.filesDir, name)).build()

    // 调用 CameraController 的 startRecording 方法
    cameraController.startRecording(outputFileOptions, ContextCompat.getMainExecutor(this), VideoSaveCallback())
}
```

#### CameraX: CameraProvider

如果使用 `CameraProvider`，则需要创建一个 `VideoCapture` `UseCase` 并传入一个 `Recorder` 对象。

##### 示例代码

```kotlin
// CameraX: 创建并绑定一个 VideoCapture UseCase
private lateinit var videoCapture: VideoCapture
private var recording: Recording? = null

val recorder = Recorder.Builder()
    .setQualitySelector(QualitySelector.from(Quality.FHD))
    .build()
videoCapture = VideoCapture.withOutput(recorder)

var camera = cameraProvider.bindToLifecycle(this, cameraSelector, preview, videoCapture)
```

此时，可以通过 `videoCapture.output` 属性访问 `Recorder`。`Recorder` 可以启动视频录制，这些视频可以保存到 `File`、`ParcelFileDescriptor` 或 `MediaStore`。

##### 示例代码

```kotlin
// CameraX: 使用 CameraProvider 实现视频捕获
private val FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS"

private fun startStopVideo() {
    val videoCapture = this.videoCapture ?: return

    if (recording != null) {
        // 停止当前录制会话
        recording?.stop()
        recording = null
        return
    }

    // 创建并启动新的录制会话
    val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US).format(System.currentTimeMillis())
    val contentValues = ContentValues().apply {
        put(MediaStore.MediaColumns.DISPLAY_NAME, name)
        put(MediaStore.MediaColumns.MIME_TYPE, "video/mp4")
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
            put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/CameraX-Video")
        }
    }

    val mediaStoreOutputOptions = MediaStoreOutputOptions.Builder(
        contentResolver,
        MediaStore.Video.Media.EXTERNAL_CONTENT_URI
    ).setContentValues(contentValues).build()

    recording = videoCapture.output
        .prepareRecording(this, mediaStoreOutputOptions)
        .withAudioEnabled()
        .start(ContextCompat.getMainExecutor(this)) { recordEvent ->
            when (recordEvent) {
                is VideoRecordEvent.Start -> {
                    viewBinding.videoCaptureButton.apply {
                        text = getString(R.string.stop_capture)
                        isEnabled = true
                    }
                }
                is VideoRecordEvent.Finalize -> {
                    if (!recordEvent.hasError()) {
                        val msg = "视频捕获成功：${recordEvent.outputResults.outputUri}"
                        Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
                        Log.d(TAG, msg)
                    } else {
                        recording?.close()
                        recording = null
                        Log.e(TAG, "视频捕获失败", recordEvent.error)
                    }
                    viewBinding.videoCaptureButton.apply {
                        text = getString(R.string.start_capture)
                        isEnabled = true
                    }
                }
            }
        }
}
```

## 其他资源

我们有多个完整的 CameraX 应用在 [Camera Samples GitHub Repository](https://github.com/android/CameraX-Samples) 中。这些示例展示了本指南中的场景如何融入一个完整的 Android 应用。

如果你在迁移到 CameraX 的过程中需要更多支持，或者有关 Android 相机 API 的问题，请在 [CameraX Discussion Group](https://groups.google.com/g/camerax-developers) 上与我们联系。


---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }