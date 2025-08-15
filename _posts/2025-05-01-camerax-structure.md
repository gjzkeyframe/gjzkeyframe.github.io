---
title: 探索 CameraX（1）：CameraX 结构
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-01 18:08:08 +0800
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


这篇文章涵盖了 CameraX 的架构，包括其结构、如何使用 API、如何处理生命周期以及如何组合用例。

## CameraX 结构

你可以通过一个名为用例（use case）的抽象层，使用 CameraX 来与设备的摄像头进行交互。可用的用例有：

  * **预览（Preview）** ：接受一个用于显示预览的表面（surface），例如 `PreviewView`。
  * **图像分析（Image analysis）** ：提供 CPU 可访问的缓冲区用于分析，例如用于机器学习。
  * **图像捕获（Image capture）** ：捕获并保存照片。
  * **视频捕获（Video capture）** ：使用 `VideoCapture` 捕获视频和音频。

用例可以组合在一起并同时保持活跃。例如，一个应用可以使用预览用例让用户查看摄像头所看到的图像，使用图像分析用例来判断照片中的人是否在微笑，以及使用图像捕获用例在用户微笑时拍照。

## API 模型

要使用该库，你需要指定以下内容：

  * 带有配置选项的所需用例。
  * 通过附加监听器来处理输出数据。
  * 通过将用例绑定到 Android 架构生命周期来确定工作流程，例如何时启用摄像头以及何时生成数据。

编写 CameraX 应用有两种方式：`CameraController`（如果你想以最简单的方式使用 CameraX，这是很好的选择）或 `CameraProvider`（如果你需要更多的灵活性，这是很好的选择）。

### CameraController

`CameraController` 在一个类中提供了大部分 CameraX 的核心功能。它需要的设置代码很少，并且它会自动处理摄像头初始化、用例管理、目标旋转、轻触对焦、捏合缩放等操作。`LifecycleCameraController` 类继承自 `CameraController`。

### Kotlin

```
val previewView: PreviewView = viewBinding.previewView
var cameraController = LifecycleCameraController(baseContext)
cameraController.bindToLifecycle(this)
cameraController.cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA
previewView.controller = cameraController
```

### Java

```
PreviewView previewView = viewBinding.previewView;
LifecycleCameraController cameraController = new LifecycleCameraController(baseContext);
cameraController.bindToLifecycle(this);
cameraController.setCameraSelector(CameraSelector.DEFAULT_BACK_CAMERA);
previewView.setController(cameraController);
```

`CameraController` 的默认 `UseCase` 包括 `Preview`、`ImageCapture` 和 `ImageAnalysis`。要关闭 `ImageCapture` 或 `ImageAnalysis`，或者开启 `VideoCapture`，可以使用 `setEnabledUseCases()` 方法。

有关 `CameraController` 的更多用法，请参见 [QR Code 扫描示例](https://developer.android.com/camerax) 或 [CameraController 基础视频](https://developer.android.com/camerax)。

### CameraProvider

`CameraProvider` 仍然易于使用，但由于应用开发者需要处理更多的设置工作，因此有更多的机会来自定义配置，例如在 `ImageAnalysis` 中启用输出图像旋转或设置输出图像格式。你还可以使用自定义 `Surface` 进行摄像头预览，这提供了更多的灵活性，而使用 CameraController 则要求你必须使用 `PreviewView` 。如果你现有的 `Surface` 代码已经是应用其他部分的输入，这可能很有用。

你可以使用 `set()` 方法配置用例，并使用 `build()` 方法完成配置。每个用例对象都提供了一组特定于用例的 API。例如，图像捕获用例提供了 `takePicture()` 方法调用。

应用不再需要在 `onResume()` 和 `onPause()` 中放置特定的启动和停止方法调用，而是通过 `cameraProvider.bindToLifecycle()` 指定一个生命周期来与摄像头关联。然后，该生命周期会通知 CameraX 何时配置摄像头捕获会话，并确保摄像头状态的变更与生命周期的转换相匹配。

有关每个用例的实现步骤，请参见 [实现预览](https://developer.android.com/camerax)、[分析图像](https://developer.android.com/camerax)、[图像捕获](https://developer.android.com/camerax) 和 [视频捕获](https://developer.android.com/camerax)。

预览用例与 `Surface` 交互以进行显示。应用可以使用以下代码使用配置选项创建用例：

### Kotlin

```
val preview = Preview.Builder().build()
val viewFinder: PreviewView = findViewById(R.id.previewView)

// 以下代码将用例绑定到 Android 生命周期
val camera = cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview)

// PreviewView 创建一个表面提供程序，是推荐的提供程序
preview.setSurfaceProvider(viewFinder.getSurfaceProvider())
```

### Java

```
Preview preview = new Preview.Builder().build();
PreviewView viewFinder = findViewById(R.id.view_finder);

// 以下代码将用例绑定到 Android 生命周期
Camera camera = cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview);

// PreviewView 创建一个表面提供程序，使用其他类型的视图的 Surface 将需要你实现自己的表面提供程序。
preview.previewSurfaceProvider = viewFinder.getSurfaceProvider();
```

有关更多示例代码，请参见官方 [CameraX 示例应用](https://github.com/android/CameraX-Samples)。

## CameraX 生命周期

CameraX 观察生命周期以确定何时打开摄像头、何时创建捕获会话以及何时停止并关闭。用例 API 提供了方法调用和回调来监控进度。

正如在 [组合用例](https://developer.android.com/camerax)中解释的那样，你可以将一些用例组合绑定到一个单一的生命周期。当你的应用需要支持无法组合的用例时，你可以执行以下操作之一：

  * 将兼容的用例分组到多个片段中，然后在片段之间切换。
  * 创建自定义生命周期组件并用它来手动控制摄像头生命周期。

如果你将视图和摄像头用例的生命周期所有者解耦（例如，如果你使用自定义生命周期或保留片段），则你必须确保使用 `ProcessCameraProvider.unbindAll()` 或分别解绑每个用例来解除 CameraX 上的所有用例绑定。或者，当你将用例绑定到生命周期时，你可以让 CameraX 管理捕获会话的打开和关闭以及用例的解绑。

如果你所有的摄像头功能都对应于一个单一的生命周期感知组件（如 `AppCompatActivity` 或 `AppCompat` 片段）的生命周期，那么使用该组件的生命周期来绑定所有所需的用例将确保摄像头功能在组件处于活动状态时可用，并在组件不处于活动状态时安全地释放资源，不消耗任何资源。

## 自定义 LifecycleOwners

在高级情况下，你可以创建自定义 `LifecycleOwner`，以使你的应用能够显式地控制 CameraX 会话生命周期，而不是将其绑定到标准的 Android `LifecycleOwner`。

以下代码示例展示了如何创建一个简单的自定义 `LifecycleOwner`：

### Kotlin

```
class CustomLifecycle : LifecycleOwner {
    private val lifecycleRegistry: LifecycleRegistry

    init {
        lifecycleRegistry = LifecycleRegistry(this)
        lifecycleRegistry.markState(Lifecycle.State.CREATED)
    }

    fun doOnResume() {
        lifecycleRegistry.markState(Lifecycle.State.RESUMED)
    }

    override fun getLifecycle(): Lifecycle {
        return lifecycleRegistry
    }
}
```

### Java

```
public class CustomLifecycle implements LifecycleOwner {
    private LifecycleRegistry lifecycleRegistry;

    public CustomLifecycle() {
        lifecycleRegistry = new LifecycleRegistry(this);
        lifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    public void doOnResume() {
        lifecycleRegistry.markState(Lifecycle.State.RESUMED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return lifecycleRegistry;
    }
}
```

使用此 `LifecycleOwner`，你的应用可以在代码中的期望位置放置状态转换。有关在应用中实现此功能的更多信息，请参见 [实现自定义 LifecycleOwner](https://developer.android.com/topic/libraries/architecture/lifecycle#custom-owners)。

## 并发用例

用例可以并发运行。虽然可以将用例依次绑定到生命周期，但最好使用单个 `CameraProcessProvider.bindToLifecycle()` 调用绑定所有用例。有关配置变更的最佳实践的更多信息，请参见 [处理配置变更](https://developer.android.com/camerax)。

在以下代码示例中，应用指定了两个要同时创建和运行的用例。它还指定了用于两个用例的生命周期，以便它们都根据生命周期开始和停止。

### Kotlin

```
private lateinit var imageCapture: ImageCapture

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    val cameraProviderFuture = ProcessCameraProvider.getInstance(this)

    cameraProviderFuture.addListener(Runnable {
        // 摄像头提供者现在可以保证可用
        val cameraProvider = cameraProviderFuture.get()

        // 设置预览用例以显示摄像头预览
        val preview = Preview.Builder().build()

        // 设置捕获用例以允许用户拍照
        imageCapture = ImageCapture.Builder()
                .setCaptureMode(ImageCapture.CAPTURE_MODE_MINIMIZE_LATENCY)
                .build()

        // 通过要求镜头方向来选择摄像头
        val cameraSelector = CameraSelector.Builder()
                .requireLensFacing(CameraSelector.LENS_FACING_FRONT)
                .build()

        // 将用例附加到具有相同生命周期所有者的摄像头
        val camera = cameraProvider.bindToLifecycle(
                this as LifecycleOwner, cameraSelector, preview, imageCapture)

        // 将预览用例连接到 previewView
        preview.setSurfaceProvider(
                previewView.getSurfaceProvider())
    }, ContextCompat.getMainExecutor(this))
}
```

### Java

```
private ImageCapture imageCapture;

@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    PreviewView previewView = findViewById(R.id.previewView);

    ListenableFuture<ProcessCameraProvider> cameraProviderFuture =
            ProcessCameraProvider.getInstance(this);

    cameraProviderFuture.addListener(() -> {
        try {
            // 摄像头提供者现在可以保证可用
            ProcessCameraProvider cameraProvider = cameraProviderFuture.get();

            // 设置视图查找器用例以显示摄像头预览
            Preview preview = new Preview.Builder().build();

            // 设置捕获用例以允许用户拍照
            imageCapture = new ImageCapture.Builder()
                    .setCaptureMode(ImageCapture.CAPTURE_MODE_MINIMIZE_LATENCY)
                    .build();

            // 通过要求镜头方向来选择摄像头
            CameraSelector cameraSelector = new CameraSelector.Builder()
                    .requireLensFacing(lensFacing)
                    .build();

            // 将用例附加到具有相同生命周期所有者的摄像头
            Camera camera = cameraProvider.bindToLifecycle(
                    ((LifecycleOwner) this),
                    cameraSelector,
                    preview,
                    imageCapture);

            // 将预览用例连接到 previewView
            preview.setSurfaceProvider(
                    previewView.getSurfaceProvider());
        } catch (InterruptedException | ExecutionException e) {
            // 目前没有抛出异常。cameraProviderFuture.get() 不应该阻塞，因为监听器正在被调用，所以不需要处理 InterruptedException。
        }
    }, ContextCompat.getMainExecutor(this));
}
```

CameraX 允许同时使用一个 `Preview`、`VideoCapture`、`ImageAnalysis` 和 `ImageCapture` 的实例。此外，

  * 每个用例都可以独立工作。例如，一个应用可以在不使用预览的情况下录制视频。
  * 当启用了扩展功能时，只有 `ImageCapture` 和 `Preview` 的组合才能保证正常工作。根据 OEM 的实现，可能无法添加 `ImageAnalysis`；扩展功能无法为 `VideoCapture` 用例启用。有关详细信息，请参阅扩展参考文档。
  * 根据摄像头能力，某些摄像头可能在较低分辨率模式下支持组合，但在某些较高分辨率模式下可能不支持。
  * 在硬件级别为 `FULL` 或更低的设备上，组合 `Preview`、`VideoCapture` 以及 `ImageCapture` 或 `ImageAnalysis` 中的任意一个可能会迫使 CameraX 为 `Preview` 和 `VideoCapture` 复制摄像头的 `PRIV` 流。这种复制称为流共享，它使得这些功能能够同时使用，但会增加处理需求。你可能会因此经历略微较高的延迟和电池寿命的减少。

可以从 `Camera2CameraInfo` 中检索支持的硬件级别。例如，以下代码检查默认的后置摄像头是否为 `LEVEL_3` 设备：

### Kotlin

```
@androidx.annotation.OptIn(ExperimentalCamera2Interop::class)
fun isBackCameraLevel3Device(cameraProvider: ProcessCameraProvider) : Boolean {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        return CameraSelector.DEFAULT_BACK_CAMERA
            .filter(cameraProvider.availableCameraInfos)
            .firstOrNull()
            ?.let { Camera2CameraInfo.from(it) }
            ?.getCameraCharacteristic(CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL) ==
            CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL_3
    }
    return false
}
```

### Java

```
@androidx.annotation.OptIn(markerClass = ExperimentalCamera2Interop.class)
Boolean isBackCameraLevel3Device(ProcessCameraProvider cameraProvider) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        List<CameraInfo> filteredCameraInfos = CameraSelector.DEFAULT_BACK_CAMERA
                .filter(cameraProvider.getAvailableCameraInfos());
        if (!filteredCameraInfos.isEmpty()) {
            return Objects.equals(
                Camera2CameraInfo.from(filteredCameraInfos.get(0)).getCameraCharacteristic(
                        CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL),
                CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL_3);
        }
    }
    return false;
}
```

## 权限

你的应用需要 `CAMERA` 权限。为了将图像保存到文件中，在运行 Android 10 或更高版本的设备上，它还需要 `WRITE_EXTERNAL_STORAGE` 权限。

有关为应用配置权限的更多信息，请参阅[请求应用权限](https://developer.android.com/training/permissions/requesting)。

## 要求

CameraX 有以下最低版本要求：

  * Android API 级别 21
  * Android 架构组件 1.1.1

对于生命周期感知活动，请使用 `FragmentActivity` 或 `AppCompatActivity`。

## 声明依赖项

要添加对 CameraX 的依赖项，你必须将 Google Maven 仓库添加到项目中。

打开项目的 `settings.gradle` 文件，并添加 `google()` 仓库，如下所示：

### Groovy

```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

### Kotlin

```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

在 Android 块的末尾添加以下内容：

### Groovy

```
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    // 对于 Kotlin 项目
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```

### Kotlin

```
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    // 对于 Kotlin 项目
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```

在应用的每个模块的 `build.gradle` 文件中添加以下内容：

### Groovy

```
dependencies {
  // 使用 camera2 实现的 CameraX 核心库
  def camerax_version = "1.5.0-alpha06"
  // 以下一行是可选的，因为核心库已被 camera-camera2 间接包含
  implementation "androidx.camera:camera-core:${camerax_version}"
  implementation "androidx.camera:camera-camera2:${camerax_version}"
  // 如果你想要另外使用 CameraX 生命周期库
  implementation "androidx.camera:camera-lifecycle:${camerax_version}"
  // 如果你想要另外使用 CameraX VideoCapture 库
  implementation "androidx.camera:camera-video:${camerax_version}"
  // 如果你想要另外使用 CameraX View 类
  implementation "androidx.camera:camera-view:${camerax_version}"
  // 如果你想要另外添加 CameraX ML Kit Vision 集成
  implementation "androidx.camera:camera-mlkit-vision:${camerax_version}"
  // 如果你想要另外使用 CameraX 扩展库
  implementation "androidx.camera:camera-extensions:${camerax_version}"
}
```

### Kotlin

```
dependencies {
    // 使用 camera2 实现的 CameraX 核心库
    val camerax_version = "1.5.0-alpha06"
    // 以下一行是可选的，因为核心库已被 camera-camera2 间接包含
    implementation("androidx.camera:camera-core:${camerax_version}")
    implementation("androidx.camera:camera-camera2:${camerax_version}")
    // 如果你想要另外使用 CameraX 生命周期库
    implementation("androidx.camera:camera-lifecycle:${camerax_version}")
    // 如果你想要另外使用 CameraX VideoCapture 库
    implementation("androidx.camera:camera-video:${camerax_version}")
    // 如果你想要另外使用 CameraX View 类
    implementation("androidx.camera:camera-view:${camerax_version}")
    // 如果你想要另外添加 CameraX ML Kit Vision 集成
    implementation("androidx.camera:camera-mlkit-vision:${camerax_version}")
    // 如果你想要另外使用 CameraX 扩展库
    implementation("androidx.camera:camera-extensions:${camerax_version}")
}
```

有关配置应用以符合这些要求的更多信息，请参见[声明依赖项](https://developer.android.com/build.dependencies)。

## CameraX 与 Camera2 的互操作性

CameraX 基于 Camera2 构建，并且 CameraX 提供了读取甚至编写 Camera2 实现中的属性的方法。有关详细信息，请参阅 Interop 包。

要了解 CameraX 如何配置 Camera2 属性，可以使用 `Camera2CameraInfo` 来读取底层的 `CameraCharacteristics` 。你还可以选择以下两种方式之一来编写底层的 Camera2 属性：

  * 使用 `Camera2CameraControl`，它允许你在底层的 `CaptureRequest` 上设置属性，例如自动对焦模式。
  * 使用 `Camera2Interop.Extender` 扩展 CameraX `UseCase` 。这使你可以在 `CaptureRequest` 上设置属性，就像 `Camera2CameraControl` 一样。它还提供了一些额外的控制，例如设置流用例以优化摄像头以适应你的使用场景。有关信息，请参见使用流用例以获得更好的性能。

以下代码示例使用流用例来优化视频通话：

### Kotlin

```
// 将底层 Camera2 流用例设置为优化视频通话。

val videoCallStreamId =
    CameraMetadata.SCALER_AVAILABLE_STREAM_USE_CASES_VIDEO_CALL.toLong()

// 检查可用的 CameraInfos 以找到第一个支持视频通话流用例的 CameraInfo。
val frontCameraInfo = cameraProvider.getAvailableCameraInfos()
    .first { cameraInfo ->
        val isVideoCallStreamingSupported = Camera2CameraInfo.from(cameraInfo)
            .getCameraCharacteristic(
                CameraCharacteristics.SCALER_AVAILABLE_STREAM_USE_CASES
            )?.contains(videoCallStreamId)
        val isFrontFacing = (cameraInfo.getLensFacing() ==
                             CameraSelector.LENS_FACING_FRONT)
        (isVideoCallStreamingSupported == true) && isFrontFacing
    }

val cameraSelector = frontCameraInfo.cameraSelector

// 从 Preview Builder 开始。
val previewBuilder = Preview.Builder()
    .setTargetAspectRatio(screenAspectRatio)
    .setTargetRotation(rotation)

// 使用 Camera2Interop.Extender 设置视频通话流用例。
Camera2Interop.Extender(previewBuilder).setStreamUseCase(videoCallStreamId)

// 绑定 Preview UseCase 和对应的 CameraSelector。
val preview = previewBuilder.build()
camera = cameraProvider.bindToLifecycle(this, cameraSelector, preview)
```

### Java

```
// 将底层 Camera2 流用例设置为优化视频通话。

Long videoCallStreamId =
    CameraMetadata.SCALER_AVAILABLE_STREAM_USE_CASES_VIDEO_CALL.toLong();

// 检查可用的 CameraInfos 以找到第一个支持视频通话流用例的 CameraInfo。
List<CameraInfo> cameraInfos = cameraProvider.getAvailableCameraInfos();
CameraInfo frontCameraInfo = null;
for (CameraInfo cameraInfo : cameraInfos) {
    Long[] availableStreamUseCases = Camera2CameraInfo.from(cameraInfo)
        .getCameraCharacteristic(
            CameraCharacteristics.SCALER_AVAILABLE_STREAM_USE_CASES
        );
    boolean isVideoCallStreamingSupported = Arrays.asList(availableStreamUseCases)
                .contains(videoCallStreamId);
    boolean isFrontFacing = (cameraInfo.getLensFacing() ==
                             CameraSelector.LENS_FACING_FRONT);

    if (isVideoCallStreamingSupported && isFrontFacing) {
        frontCameraInfo = cameraInfo;
    }
}

if (frontCameraInfo == null) {
    // 处理视频通话流不可用的情况。
}

CameraSelector cameraSelector = frontCameraInfo.getCameraSelector();

// 从 Preview Builder 开始。
Preview.Builder previewBuilder = Preview.Builder()
    .setTargetAspectRatio(screenAspectRatio)
    .setTargetRotation(rotation);

// 使用 Camera2Interop.Extender 设置视频通话流用例。
Camera2Interop.Extender(previewBuilder).setStreamUseCase(videoCallStreamId);

// 绑定 Preview UseCase 和对应的 CameraSelector。
Preview preview = previewBuilder.build();
Camera camera = cameraProvider.bindToLifecycle(this, cameraSelector, preview);
```

## 其他资源

要了解更多关于 CameraX 的信息，请参考以下额外资源。

### Codelab

[开始使用 CameraX](https://developer.android.com/camerax)

### 代码示例

[CameraX 示例应用](https://github.com/android/CameraX-Samples)



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