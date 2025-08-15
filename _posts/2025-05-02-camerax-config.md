---
title: 探索 CameraX（2）：CameraX 配置选项
description: 系列介绍 CameraX 相关的基础技术。
author: Keyframe
date: 2025-05-02 18:08:08 +0800
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
## 配置选项

这篇文章主要介绍了如何配置 CameraX 的各个用例，从而控制用例操作的不同方面。例如，在图像捕获用例中，可以设置目标纵横比和闪光灯模式。以下是代码示例：

### Kotlin

```
val imageCapture = ImageCapture.Builder()
    .setFlashMode(...)
    .setTargetAspectRatio(...)
    .build()
```

### Java

```
ImageCapture imageCapture =
    new ImageCapture.Builder()
        .setFlashMode(...)
        .setTargetAspectRatio(...)
        .build();
```

除了配置选项外，一些用例还提供了 API，可以在用例创建后动态更改设置。有关各个用例特定配置的信息，请参见实现预览、分析图像和图像捕获。

## CameraXConfig

为了简化操作，CameraX 提供了适用于大多数使用场景的默认配置，例如内部执行器和处理程序。然而，如果你的应用有特殊要求或需要自定义这些配置，`CameraXConfig` 就派上用场了。

通过 `CameraXConfig`，应用可以实现以下功能：

  * 使用 `setAvailableCameraLimiter()` 优化启动延迟。
  * 通过 `setCameraExecutor()` 向 CameraX 提供应用的执行器。
  * 使用 `setSchedulerHandler()` 替换默认的调度处理程序。
  * 使用 `setMinimumLoggingLevel()` 更改日志记录级别。

### 使用模型

以下是使用 `CameraXConfig` 的步骤：

  1. 创建一个包含自定义配置的 `CameraXConfig` 对象。
  2. 在应用中实现 `CameraXConfig.Provider` 接口，并在 `getCameraXConfig()` 中返回你的 `CameraXConfig` 对象。
  3. 将应用类添加到 `AndroidManifest.xml` 文件中。

例如，以下代码示例将 CameraX 日志记录限制为仅显示错误消息：

### Kotlin

```
class CameraApplication : Application(), CameraXConfig.Provider {
   override fun getCameraXConfig(): CameraXConfig {
       return CameraXConfig.Builder.fromConfig(Camera2Config.defaultConfig())
           .setMinimumLoggingLevel(Log.ERROR).build()
   }
}
```

如果应用在设置后需要了解 CameraX 配置，建议保留 `CameraXConfig` 对象的本地副本。

### 相机限制器

在首次调用 `ProcessCameraProvider.getInstance()` 时，CameraX 会枚举并查询设备上可用摄像头的特性。由于需要与硬件组件通信，此过程可能需要较长时间，特别是在低端设备上。如果应用只使用设备上的特定摄像头（如默认前置摄像头），可以设置 CameraX 忽略其他摄像头，从而减少应用所用摄像头的启动延迟。

如果传递给 `CameraXConfig.Builder.setAvailableCamerasLimiter()` 的 `CameraSelector` 过滤掉某个摄像头，CameraX 会表现得好像该摄像头不存在一样。例如，以下代码将应用限制为仅使用设备的默认后置摄像头：

### Kotlin

```
class MainApplication : Application(), CameraXConfig.Provider {
   override fun getCameraXConfig(): CameraXConfig {
       return CameraXConfig.Builder.fromConfig(Camera2Config.defaultConfig())
              .setAvailableCamerasLimiter(CameraSelector.DEFAULT_BACK_CAMERA)
              .build()
   }
}
```

### 线程

CameraX 所基于的许多平台 API 需要进行阻塞式进程间通信（IPC），这有时可能需要数百毫秒才能响应。因此，CameraX 仅从后台线程调用这些 API，以避免阻塞主线程并保持 UI 流畅。CameraX 内部管理这些后台线程，使这种行为看起来是透明的。然而，一些应用需要严格控制线程。`CameraXConfig` 允许应用通过 `CameraXConfig.Builder.setCameraExecutor()` 和 `CameraXConfig.Builder.setSchedulerHandler()` 设置用于后台线程的执行器。

### 相机执行器

相机执行器用于所有内部相机平台 API 调用以及这些 API 的回调。CameraX 分配并管理一个内部 `Executor` 以执行这些任务。然而，如果你的应用需要更严格的线程控制，可以使用 `CameraXConfig.Builder.setCameraExecutor()`。

### 调度处理程序

调度处理程序用于以固定间隔安排内部任务，例如在相机不可用时重试打开相机。此处理程序不执行任务，仅将它们分派给相机执行器。在某些需要 `Handler` 回调的旧版 API 平台上，回调仍然直接分派给相机执行器。CameraX 分配并管理一个内部 `HandlerThread` 以执行这些任务，但你可以通过 `CameraXConfig.Builder.setSchedulerHandler()` 覆盖它。

### 日志记录

CameraX 日志记录允许应用过滤 logcat 消息。生产代码中避免冗长消息是一种良好做法。CameraX 支持四种日志级别，从最详细到最严重依次为：

  * `Log.DEBUG`（默认）
  * `Log.INFO`
  * `Log.WARN`
  * `Log.ERROR`

有关这些日志级别的详细描述，请参阅 Android Log 文档。使用 `CameraXConfig.Builder.setMinimumLoggingLevel(int)` 为应用设置适当的日志级别。

## 自动选择

CameraX 会根据应用运行的设备自动提供特定功能。例如，如果未指定分辨率或指定的分辨率不受支持，CameraX 会自动确定最佳分辨率。所有这些都由库处理，无需编写特定于设备的代码。

CameraX 的目标是成功初始化相机会话。这意味着基于设备能力，CameraX 会在分辨率和纵横比上做出妥协。这种妥协可能发生在以下情况：

  * 设备不支持请求的分辨率。
  * 设备存在兼容性问题，例如某些旧设备需要特定分辨率才能正常运行。
  * 在某些设备上，某些格式仅在特定纵横比下可用。
  * 设备倾向于使用 JPEG 或视频编码的 “最近 mod16” 分辨率。有关更多信息，请参见 `SCALER_STREAM_CONFIGURATION_MAP`。

尽管 CameraX 创建并管理会话，但始终在代码中检查用例输出返回的图像大小，并相应调整。

## 旋转

默认情况下，在用例创建期间，相机旋转设置为与默认显示的旋转相匹配。在这种默认情况下，CameraX 生成输出以使应用显示符合预期的预览。可以通过在配置用例对象时传入当前显示方向，或在创建后动态更改旋转设置来更改旋转，以支持多显示设备。

应用可以使用配置设置设置目标旋转，然后在生命周期处于运行状态时使用用例 API 的方法（如 `ImageAnalysis.setTargetRotation()`）更新旋转设置。例如，当应用锁定为 Portrait 模式时，可能需要这样做，因为此时不会发生旋转重配置，但照片或分析用例需要了解设备的当前旋转情况。例如，旋转感知可能对于面部检测需要正确定向面部，或者照片需要设置为横向或纵向。

捕获图像的数据可能未包含旋转信息。Exif 数据包含旋转信息，以便图库应用在保存后可以正确显示图像。

为了以正确的方向显示预览数据，可以使用 `Preview.PreviewOutput()` 的元数据输出来创建转换。

以下代码示例展示了如何在方向事件上设置旋转：

### Kotlin

```
override fun onCreate() {
    val imageCapture = ImageCapture.Builder().build()

    val orientationEventListener = object : OrientationEventListener(this as Context) {
        override fun onOrientationChanged(orientation : Int) {
            // 监控方向值以确定目标旋转值
            val rotation : Int = when (orientation) {
                in 45..134 -> Surface.ROTATION_270
                in 135..224 -> Surface.ROTATION_180
                in 225..314 -> Surface.ROTATION_90
                else -> Surface.ROTATION_0
            }

            imageCapture.targetRotation = rotation
        }
    }
    orientationEventListener.enable()
}
```

### Java

```
@Override
public void onCreate() {
    ImageCapture imageCapture = new ImageCapture.Builder().build();

    OrientationEventListener orientationEventListener = new OrientationEventListener((Context)this) {
       @Override
       public void onOrientationChanged(int orientation) {
           int rotation;

           // 监控方向值以确定目标旋转值
           if (orientation >= 45 && orientation < 135) {
               rotation = Surface.ROTATION_270;
           } else if (orientation >= 135 && orientation < 225) {
               rotation = Surface.ROTATION_180;
           } else if (orientation >= 225 && orientation < 315) {
               rotation = Surface.ROTATION_90;
           } else {
               rotation = Surface.ROTATION_0;
           }

           imageCapture.setTargetRotation(rotation);
       }
    };

    orientationEventListener.enable();
}
```

根据设置的旋转，每个用例要么直接旋转图像数据，要么向非旋转图像数据的使用者提供旋转元数据。

  * **预览** ：提供元数据输出，以便使用 `Preview.getTargetRotation()` 知道目标分辨率的旋转情况。
  * **图像分析** ：提供元数据输出，以便知道图像缓冲区坐标相对于显示坐标。
  * **图像捕获** ：图像 Exif 元数据、缓冲区或缓冲区和元数据都会被更改以记录旋转设置。更改的值取决于 HAL 实现。

## 裁剪矩形

默认情况下，裁剪矩形是完整的缓冲区矩形。可以使用 `ViewPort` 和 `UseCaseGroup` 自定义它。通过将用例分组并设置视口，CameraX 保证组中所有用例的裁剪矩形都指向相机传感器中的同一区域。

以下代码片段展示了如何使用这两个类：

### Kotlin

```
val viewPort =  ViewPort.Builder(Rational(width, height), display.rotation).build()
val useCaseGroup = UseCaseGroup.Builder()
    .addUseCase(preview)
    .addUseCase(imageAnalysis)
    .addUseCase(imageCapture)
    .setViewPort(viewPort)
    .build()
cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, useCaseGroup)
```

### Java

```
ViewPort viewPort = new ViewPort.Builder(
         new Rational(width, height),
         getDisplay().getRotation()).build();
UseCaseGroup useCaseGroup = new UseCaseGroup.Builder()
    .addUseCase(preview)
    .addUseCase(imageAnalysis)
    .addUseCase(imageCapture)
    .setViewPort(viewPort)
    .build();
cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, useCaseGroup);
```

`ViewPort` 定义了最终用户可见的缓冲区矩形。然后 CameraX 根据视口和附加用例的属性计算最大的可能裁剪矩形。通常，为了实现所见即所得的效果，可以根据预览用例配置视口。获取视口的简单方法是使用 `PreviewView`。

以下代码片段展示了如何获取 `ViewPort` 对象：

### Kotlin

```
val viewport = findViewById<PreviewView>(R.id.preview_view).viewPort
```

### Java

```
ViewPort viewPort = ((PreviewView)findViewById(R.id.preview_view)).getViewPort();
```

在上述示例中，假设 `PreviewView` 的缩放类型设置为默认的 `FILL_CENTER`，则应用从 `ImageAnalysis` 和 `ImageCapture` 获取的内容与最终用户在 `PreviewView` 中看到的内容一致。应用裁剪矩形和旋转后，所有用例的图像相同，但可能具有不同的分辨率。有关如何应用转换信息的更多信息，请参见转换输出。

## 相机选择

CameraX 会根据应用的要求和用例自动选择最佳相机设备。如果希望使用与自动选择不同的设备，有以下几种选项：

  * 使用 `CameraSelector.DEFAULT_FRONT_CAMERA` 请求默认前置摄像头。
  * 使用 `CameraSelector.DEFAULT_BACK_CAMERA` 请求默认后置摄像头。
  * 使用 `CameraSelector.Builder.addCameraFilter()` 根据 `CameraCharacteristics` 筛选可用设备列表。

以下代码示例展示了如何创建一个 `CameraSelector` 以影响设备选择：

### Kotlin

```
fun selectExternalOrBestCamera(provider: ProcessCameraProvider):CameraSelector? {
   val cam2Infos = provider.availableCameraInfos.map {
       Camera2CameraInfo.from(it)
   }.sortedByDescending {
       // HARDWARE_LEVEL 是 Int 类型，顺序为：
       // LEGACY < LIMITED < FULL < LEVEL_3 < EXTERNAL
       it.getCameraCharacteristic(CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL)
   }

   return when {
       cam2Infos.isNotEmpty() -> {
           CameraSelector.Builder()
               .addCameraFilter {
                   it.filter { camInfo ->
                       // cam2Infos[0] 是 EXTERNAL 或最佳内置相机
                       val thisCamId = Camera2CameraInfo.from(camInfo).cameraId
                       thisCamId == cam2Infos[0].cameraId
                   }
               }.build()
       }
       else -> null
    }
}

// 为 USB 相机（或最高级别内置相机）创建 CameraSelector
val selector = selectExternalOrBestCamera(processCameraProvider)
processCameraProvider.bindToLifecycle(this, selector, preview, analysis)
```

## 同时选择多个相机

从 CameraX 1.3 开始，还可以同时选择多个相机。例如，可以绑定前置和后置相机，以同时从两个视角拍照或录制视频。

当使用并发相机功能时，设备可以同时操作两个不同朝向的相机，或者同时操作两个后置相机。以下代码块展示了如何在调用 `bindToLifecycle` 时设置两个相机，以及如何从返回的 `ConcurrentCamera` 对象中获取两个相机对象。

### Kotlin

```
// 构建 ConcurrentCameraConfig
val primary = ConcurrentCamera.SingleCameraConfig(
    primaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
)

val secondary = ConcurrentCamera.SingleCameraConfig(
    secondaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
)

val concurrentCamera = cameraProvider.bindToLifecycle(
    listOf(primary, secondary)
)

val primaryCamera = concurrentCamera.cameras[0]
val secondaryCamera = concurrentCamera.cameras[1]
```

### Java

```
// 构建 ConcurrentCameraConfig
SingleCameraConfig primary = new SingleCameraConfig(
    primaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
);

SingleCameraConfig secondary = new SingleCameraConfig(
    primaryCameraSelector,
    useCaseGroup,
    lifecycleOwner
);

ConcurrentCamera concurrentCamera =
    mCameraProvider.bindToLifecycle(Arrays.asList(primary, secondary));

Camera primaryCamera = concurrentCamera.getCameras().get(0);
Camera secondaryCamera = concurrentCamera.getCameras().get(1);
```

## 相机分辨率

你可以选择让 CameraX 根据设备能力、设备支持的硬件级别、用例和提供的纵横比的组合来设置图像分辨率。或者，在支持该配置的用例中设置特定目标分辨率或纵横比。

### 自动分辨率

CameraX 可以根据在 `cameraProcessProvider.bindToLifecycle()` 中指定的用例自动确定最佳分辨率设置。只要可能，就在单个 `bindToLifecycle()` 调用中指定所有需要同时运行的用例。CameraX 根据绑定的用例集确定分辨率，考虑设备支持的硬件级别，并考虑设备特定的差异（设备超过或未达到可用流配置）。

图像捕获和图像分析用例的默认纵横比为 4:3。

用例具有可配置的纵横比，以便应用根据 UI 设计指定所需的纵横比。CameraX 输出生成以尽可能接近设备支持的请求纵横比。如果没有完全匹配的分辨率，则选择满足最多条件的分辨率。这样，应用决定相机在应用中的显示方式，CameraX 根据不同设备确定最佳相机分辨率设置。

例如，应用可以执行以下任一操作：

  * 为用例指定 4:3 或 16:9 的目标分辨率
  * 指定一个自定义分辨率，CameraX 尝试找到最接近的匹配项
  * 为 `ImageCapture` 指定裁剪纵横比

CameraX 自动选择内部 Camera2 表面分辨率。下表显示了分辨率：

| **用例**| **内部表面分辨率**| **输出数据分辨率**|
| ---| ---| ---|
| 预览| **纵横比：** 最符合目标设置的分辨率。| 内部表面分辨率。提供元数据以使视图能够裁剪、缩放和旋转以匹配目标纵横比。|
| **默认分辨率：** 最高预览分辨率，或与预览纵横比匹配的最高设备首选分辨率。| **最大分辨率：** 预览大小，即与设备屏幕分辨率最匹配的大小，或小于 1080p（1920x1080）的大小。|
| 图像分析| **纵横比：** 最符合目标设置的分辨率。| 内部表面分辨率。|
| **默认分辨率：** 默认目标分辨率设置为 640x480。同时调整目标分辨率和相应的纵横比，以获得最佳支持的分辨率。| **最大分辨率：** 相机设备的 YUV_420_888 格式最大输出分辨率，通过 `StreamConfigurationMap.getOutputSizes()` 获取。目标分辨率默认设置为 640x480，因此如果需要大于 640x480 的分辨率，则必须使用 `setTargetResolution()` 和 `setTargetAspectRatio()` 从支持的分辨率中获取最接近的一个。|
| 图像捕获| **纵横比：** 最符合设置的纵横比。| 内部表面分辨率。|
| **默认分辨率：** 可用的最高分辨率，或与 ImageCapture 的纵横比匹配的最高设备首选分辨率。| **最大分辨率：** 相机设备的 JPEG 格式最大输出分辨率。使用 `StreamConfigurationMap.getOutputSizes()` 获取。|

### 指定分辨率

在构建用例时，可以使用 `setTargetResolution(Size resolution)` 方法设置特定分辨率，如以下代码示例所示：

### Kotlin

```
val imageAnalysis = ImageAnalysis.Builder()
    .setTargetResolution(Size(1280, 720))
    .build()
```

### Java

```
ImageAnalysis imageAnalysis =
  new ImageAnalysis.Builder()
    .setTargetResolution(new Size(1280, 720))
    .build();
```

不能在同一用例中同时设置目标纵横比和目标分辨率。否则在构建配置对象时会抛出 `IllegalArgumentException`。

将分辨率 `Size` 表示为在目标旋转后支持的大小的坐标框架。例如，一个具有肖像自然方向的设备在自然目标旋转下请求肖像图像，可以指定 480x640，而同一设备旋转 90 度并针对横向方向可以指定 640x480。

目标分辨率尝试为图像分辨率建立一个最小界限。实际图像分辨率是 Camera 实现确定的不小于目标分辨率的最接近可用分辨率。但是，如果不存在等于或大于目标分辨率的分辨率，则选择比目标分辨率小的最接近可用分辨率。具有与提供的 `Size` 相同纵横比的分辨率比不同纵横比的分辨率具有更高优先级。

CameraX 根据请求应用最适合的分辨率。如果主要需求是满足纵横比，则仅指定 `setTargetAspectRatio`，CameraX 根据设备确定一个合适的特定分辨率。如果应用的主要需求是通过指定分辨率来提高图像处理效率（例如，基于设备处理能力的小型或中型图像），则使用 `setTargetResolution(Size resolution)`。

如果应用需要精确的分辨率，请参见 `createCaptureSession()` 中的表格，以确定每个硬件级别支持的最大分辨率。要检查当前设备支持的具体分辨率，请参见 `StreamConfigurationMap.getOutputSizes(int)`。

如果应用在 Android 10 或更高版本上运行，可以使用 `isSessionConfigurationSupported()` 验证特定的 `SessionConfiguration`。

## 控制相机输出

除了允许为每个单独的用例按需配置相机输出外，CameraX 还实现了以下接口，以支持所有绑定用例的通用相机操作：

  * `CameraControl` 允许配置通用相机功能。
  * `CameraInfo` 允许查询这些通用相机功能的状态。

以下是 `CameraControl` 支持的相机功能：

  * 缩放
  * 手电筒
  * 对焦和测光（轻触对焦）
  * 曝光补偿

### 获取 CameraControl 和 CameraInfo 实例

使用 `ProcessCameraProvider.bindToLifecycle()` 返回的 `Camera` 对象获取 `CameraControl` 和 `CameraInfo` 实例。以下代码展示了示例：

### Kotlin

```
val camera = processCameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview)

// 用于执行影响所有输出的操作。
val cameraControl = camera.cameraControl
// 用于查询信息和状态。
val cameraInfo = camera.cameraInfo
```

### Java

```
Camera camera = processCameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview)

// 用于执行影响所有输出的操作。
CameraControl cameraControl = camera.getCameraControl()
// 用于查询信息和状态。
CameraInfo cameraInfo = camera.getCameraInfo()
```

例如，可以在调用 `bindToLifecycle()` 后提交缩放和其他 `CameraControl` 操作。在用于绑定相机实例的活动停止或销毁后，`CameraControl` 不再执行操作，并返回失败的 `ListenableFuture`。

### 缩放

`CameraControl` 提供了两种更改缩放级别方法：

  * `setZoomRatio()` 通过缩放比率设置缩放。

比率必须在 `CameraInfo.getZoomState().getValue().getMinZoomRatio()` 和 `CameraInfo.getZoomState().getValue().getMaxZoomRatio()` 范围内。否则函数返回失败的 `ListenableFuture`。

  * `setLinearZoom()` 使用线性缩放值（从 0 到 1.0）设置当前缩放级别。

线性缩放的优势在于它使视野（FOV）随缩放变化而缩放。这使其非常适合与 `Slider` 视图一起使用。


`CameraInfo.getZoomState()` 返回当前缩放状态的 LiveData。当相机初始化或通过 `setZoomRatio()` 或 `setLinearZoom()` 设置缩放级别时，值会更改。调用任一方法会设置 `ZoomState.getZoomRatio()` 和 `ZoomState.getLinearZoom()` 的值。这在想要在滑块旁边显示缩放比率文本时很有帮助。只需观察 `ZoomState` LiveData 即可同时更新两者，无需进行转换。

两种 API 返回的 `ListenableFuture` 提供了在完成具有指定缩放值的重复请求时通知应用的选项。此外，如果在前一个操作仍在执行时设置新的缩放值，前一个缩放操作的 `ListenableFuture` 会立即失败。

### 手电筒

`CameraControl.enableTorch(boolean)` 启用手电筒（也称为闪光灯）。

可以使用 `CameraInfo.getTorchState()` 查询当前手电筒状态。可以通过检查 `CameraInfo.hasFlashUnit()` 返回的值来确定是否可用用手电筒。如果不支持手电筒，调用 `CameraControl.enableTorch(boolean)` 会导致返回的 `ListenableFuture` 立即完成并返回失败结果，并将手电筒状态设置为 `TorchState.OFF`。

当手电筒启用时，无论 flashMode 设置如何，在拍照和录像期间都会保持开启状态。`ImageCapture` 中的 flashMode 仅在手电筒禁用时起作用。

### 对焦和测光

`CameraControl.startFocusAndMetering()` 通过根据给定的 FocusMeteringAction 设置 AF/AE/AWB 测光区域触发自动对焦和曝光测光。这通常用于实现许多相机应用中的 “轻触对焦” 功能。

#### MeteringPoint

首先，使用 `MeteringPointFactory.createPoint(float x, float y, float size)` 创建 `MeteringPoint`。`MeteringPoint` 表示相机 `Surface` 上的一个点。它以归一化形式存储，以便可以轻松转换为传感器坐标，用于指定 AF/AE/AWB 区域。

`MeteringPoint` 的大小范围为 0 到 1，默认大小为 0.15f。调用 `MeteringPointFactory.createPoint(float x, float y, float size)` 时，CameraX 创建一个以 `(x, y)` 为中心的矩形区域，大小为提供的 `size`。

以下代码展示了如何创建 `MeteringPoint`：

### Kotlin

```
// 如果使用 PreviewView 进行预览，请使用 PreviewView.getMeteringPointFactory。
previewView.setOnTouchListener((view, motionEvent) ->  {
val meteringPoint = previewView.meteringPointFactory
    .createPoint(motionEvent.x, motionEvent.y)
…
}

// 如果使用 SurfaceView / TextureView 进行预览，请使用 DisplayOrientedMeteringPointFactory。
// 请注意，如果预览在视图中被缩放或裁剪，则由应用负责正确转换坐标。
// 此工厂的宽度和高度代表完整的预览 FOV。
// 传递给 create MeteringPoint 的 (x,y) 可能需要根据偏移量进行调整。
val meteringPointFactory = DisplayOrientedMeteringPointFactory(
     surfaceView.display,
     camera.cameraInfo,
     surfaceView.width,
     surfaceView.height
)

// 如果点是在 ImageAnalysis ImageProxy 中指定的，请使用 SurfaceOrientedMeteringPointFactory。
val meteringPointFactory = SurfaceOrientedMeteringPointFactory(
     imageWidth,
     imageHeight,
     imageAnalysis)
```

#### startFocusAndMetering 和 FocusMeteringAction

要调用 `startFocusAndMetering()`，应用必须构建一个 `FocusMeteringAction`，它由一个或多个 `MeteringPoints` 组成，并可选地组合 `FLAG_AF`、`FLAG_AE`、`FLAG_AWB` 测光模式。以下代码展示了这种用法：

### Kotlin

```
val meteringPoint1 = meteringPointFactory.createPoint(x1, x1)
val meteringPoint2 = meteringPointFactory.createPoint(x2, y2)
val action = FocusMeteringAction.Builder(meteringPoint1) // 默认 AF|AE|AWB
      // 可选地为 AF/AE 添加 meteringPoint2。
      .addPoint(meteringPoint2, FLAG_AF or FLAG_AE)
      // 行动在 3 秒后取消（如果不设置，默认为 5 秒）。
      .setAutoCancelDuration(3, TimeUnit.SECONDS)
      .build()

val result = cameraControl.startFocusAndMetering(action)
// 如果需要了解 focusMetering 结果，可以为 ListenableFuture 添加监听器。
result.addListener({
   // result.get().isFocusSuccessful 返回自动对焦是否成功。
}, ContextCompat.getMainExecutor(this)
```

如前所述，`startFocusAndMetering()` 接受一个 `FocusMeteringAction`，它由一个用于 AF/AE/AWB 测光区域的 `MeteringPoint` 和另一个仅用于 AF 和 AE 的 MeteringPoint 组成。

内部，CameraX 将其转换为 Camera2 `MeteringRectangles` 并设置相应的 `CONTROL_AF_REGIONS` / `CONTROL_AE_REGIONS` / `CONTROL_AWB_REGIONS` 参数到捕获请求。

由于并非所有设备都支持 AF/AE/AWB 和多个区域，CameraX 尽最大努力执行 `FocusMeteringAction`。CameraX 使用支持的最大 MeteringPoints 数量，并按添加顺序使用它们。在最大计数之后添加的所有 MeteringPoints 都被忽略。例如，如果在支持仅 2 个的平台上提供了一个包含 3 个 MeteringPoints 的 `FocusMeteringAction`，则只使用前两个 MeteringPoints。最后一个 MeteringPoint 被 CameraX 忽略。

#### 曝光补偿

当应用需要在自动曝光（AE）输出结果之外微调曝光值（EV）时，曝光补偿非常有用。曝光补偿值以以下方式组合，以确定当前图像条件所需的曝光：

`Exposure = ExposureCompensationIndex * ExposureCompensationStep`

CameraX 提供了 `Camera.CameraControl.setExposureCompensationIndex()` 函数，用于将曝光补偿设置为索引值。

正值使图像变亮，负值使图像变暗。应用可以通过 `CameraInfo.ExposureState.exposureCompensationRange()` 查询支持的范围。如果值受支持，返回的 `ListenableFuture` 在值成功启用于捕获请求时完成；如果指定的索引超出支持范围，则 `setExposureCompensationIndex()` 导致返回的 `ListenableFuture` 立即完成并返回失败结果。

CameraX 只保留最新的未完成的 `setExposureCompensationIndex()` 请求，并在前一个请求完成之前多次调用该函数会导致其被取消。

以下片段设置曝光补偿索引，并注册一个回调，用于在曝光更改请求执行后通知：

### Kotlin

```
camera.cameraControl.setExposureCompensationIndex(exposureCompensationIndex)
   .addListener({
      // 获取当前曝光补偿索引，它可能与请求的值不同，因为此请求可能被更新设置请求取消。
      val currentExposureIndex = camera.cameraInfo.exposureState.exposureCompensationIndex
      …
   }, mainExecutor)
```

  * `Camera.CameraInfo.getExposureState()` 检索当前 `ExposureState`，包括：

  * 曝光补偿控制的支持情况。
  * 当前曝光补偿索引。
  * 曝光补偿索引范围。
  * 用于曝光补偿值计算的曝光补偿步长。

例如，以下代码使用当前 `ExposureState` 值初始化曝光 `SeekBar`：

### Kotlin

```
val exposureState = camera.cameraInfo.exposureState
binding.seekBar.apply {
   isEnabled = exposureState.isExposureCompensationSupported
   max = exposureState.exposureCompensationRange.upper
   min = exposureState.exposureCompensationRange.lower
   progress = exposureState.exposureCompensationIndex
}
```

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