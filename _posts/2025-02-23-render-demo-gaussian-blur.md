---
title: RenderDemo（3）：用 OpenGL 实现高斯模糊
description: 介绍用 OpenGL 实现高斯模糊的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 13:28:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, Android, OpenGL, 渲染]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
>_微信扫码关注我们_
>
>您还可以加入知识星球 `关键帧的音视频开发圈` 来一起交流工作中的**技术难题、职场经验**：
>
>![知识星球](assets/img/keyframe-zsxq.png)
>_微信扫码加入星球_
{: .prompt-tip }

这里是 RenderDemo 的第三篇：**用 OpenGL 实现高斯模糊**。我们分别在 iOS 和 Android 平台实现了用 OpenGL 对图像进行高斯模糊处理并渲染出来。效果图如下：

![高斯模糊处理图片效果](assets/resource/av-demo/render-demo-gaussian-blur-1.png)
_高斯模糊处理图片效果_


到目前我们已经在我们的付费知识星球中提供了下面这些音视频 Demo 和渲染 Demo 的工程源码，均可直接下载运行：

- iOS AVDemo（1）：音频采集
- iOS AVDemo（2）：音频编码
- iOS AVDemo（3）：音频封装
- iOS AVDemo（4）：音频解封装
- iOS AVDemo（5）：音频解码
- iOS AVDemo（6）：音频渲染
- iOS AVDemo（7）：视频采集
- iOS AVDemo（8）：视频编码
- iOS AVDemo（9）：视频封装
- iOS AVDemo（10）：视频解封装
- iOS AVDemo（11）：视频转封装
- iOS AVDemo（12）：视频编码
- iOS AVDemo（13）：视频渲染
- Android AVDemo（1）：音频采集
- Android AVDemo（2）：音频编码
- Android AVDemo（3）：音频封装
- Android AVDemo（4）：音频解封装
- Android AVDemo（5）：音频解码
- Android AVDemo（6）：音频渲染
- Android AVDemo（7）：视频采集
- Android AVDemo（8）：视频编码
- Android AVDemo（9）：视频封装
- Android AVDemo（10）：视频解封装
- Android AVDemo（11）：视频转封装
- Android AVDemo（12）：视频解码
- Android AVDemo（13）：视频渲染
- RenderDemo（1）：用 OpenGL 画一个三角形（iOS+Android）
- RenderDemo（2）：用 OpenGL 渲染视频（iOS+Android）
- RenderDemo（3）：用 OpenGL 实现高斯模糊（iOS+Android）

这些源码对于学习和理解 iOS/Android 音视频开发非常容易上手，有需要的朋友可以扫描下面二维码加入星球获取全部源码：

![微信扫码二维码加入我们](assets/img/keyframe-zsxq.png)



---

高斯模糊是一种柔和模糊的图像效果，模糊后的图像可以被更复杂的算法用来产生例如炫光、景深、热浪或者毛玻璃的效果。本文将会给大家介绍高斯模糊的数学原理，以及用 OpenGL 完成高斯模糊的代码实现。

## 1、高斯模糊基础知识

高斯模糊（Gaussian Blur），也叫高斯平滑，是在图像处理中广泛使用的处理效果，通常用它来减少图像噪声以及降低细节层次。因为其视觉效果就像是经过一个半透明屏幕在观察图像，所以常用于生成毛玻璃效果。

从数学的角度来看，图像的高斯模糊过程就是图像与正态分布做卷积，由于正态分布又叫作高斯分布，所以这项技术就叫作高斯模糊。

### 1.1、基本原理

让我们先看一个直观的例子来理解模糊这个概念。

![数据平滑](assets/resource/av-demo/DataSmooth1.png)
_数据平滑_

上图中，中间点是 2，周边点都是 1。

假如现在我们想让中间点和周围的点数值上更加接近来达成我们模糊中间点和周围点边界的目的。我们可以让中间点取周围点的平均值，那么中间点就会从 2 变成 1，中间点就会靠近周围的值，这就是数值上的平滑，也就是模糊。

![数据平滑](assets/resource/av-demo/DataSmooth2.png)
_数据平滑_

我们将这个想法应用到图像上，对图像中的每一个像素点，取周围像素的平均值，自然而然就会让这幅图产生模糊效果。

当我们取周围点的时候，所参考的范围呈现一个圆形，圆形半径越大，模糊效果就会越强烈。

![高斯模糊效果图](assets/resource/av-demo/1200px-Cappadocia_Gaussian_Blur.png)
_高斯模糊效果图_

如果使用简单平均，显然不是很合理，因为图像都是连续的，越靠近的点关系越密切，越远离的点关系越疏远。因此，加权平均更合理，距离越近的点权重越大，距离越远的点权重越小。

高斯模糊就是一种加权平均的模糊效果。

### 1.2、高斯函数的数学表达

正态分布的密度函数叫做`高斯函数（Gaussian function）`。

在图形上，正态分布是一种钟形曲线，越接近中心，取值越大，越远离中心，取值越小。计算平均值的时候，我们只需要将中心点作为原点，其他点按照其在正态曲线上的位置，分配权重，就可以得到一个加权平均值。

![一维高斯函数的图像表示](assets/resource/av-demo/normal_distribution.png)
_一维高斯函数的图像表示_

因为我们处理的是图像，而图像可以表示为二维矩阵，其中每个元素为 ARGB 像素值，因此我们在这里需要延伸到二维高斯函数。


![二维高斯函数的图像表示](assets/resource/av-demo/gaussian_graph.png)
_二维高斯函数的图像表示_


高斯函数的一维形式是：

![高斯函数一维函数方程](assets/resource/av-demo/gaussian_fuction_d1.png)
_高斯函数一维函数方程_

高斯函数的二维形式是：

![高斯函数二维函数方程](assets/resource/av-demo/gaussian_fuction_d2.png)
_高斯函数二维函数方程_


假如目前有一张宽高为 1024x1024 的图像，我们使用上述所说的方法对这个图像上的每个点计算二维正态分布的加权，参考当前坐标附近距离半径为 33 的所有像素。那么可以得知，我们需要进行的计算次数为 `1024 * 1024 * 33 * 33 ≈ 11.4 亿`次，这显然是一个不可接受的算法，我们需要对算法的效率进行优化。

因为二维高斯函数具有分离性，所以二维高斯函数可以拆分为两个一维高斯函数来计算（证明过程参考：[二维高斯卷积核拆分成两个一维的高斯卷积核](https://www.zhihu.com/question/364517094/answer/2258633530 "二维高斯卷积核拆分成两个一维的高斯卷积核")），所以我们可以将算法优化为先计算水平方向的加权函数再计算垂直方向的加权函数。这样我们的算法计算量就从之前的 `1024 * 1024 * 33 * 33 ≈ 11.4 亿`次 下降为 `1024 * 1024 * 33 * 2 ≈ 6900 万`次。
 
![优化后的算法](assets/resource/av-demo/vhblur.png)
_优化后的算法_

所以，我们的算法优化为水平方向运行一次着色器后再在垂直方向运行一次着色器。


## 2、iOS Demo

### 2.1、渲染模块

渲染模块与 [OpenGL 渲染视频](https://mp.weixin.qq.com/s/eBB6jkvsufXbIWCLwsueOQ) 中讲到的一致，最终是封装出一个渲染视图 `KFOpenGLView` 用于展示最后的渲染结果。这里就不再细讲，只贴一下主要的类和类具体的功能：

- `KFOpenGLView`：使用 OpenGL 实现的渲染 View，提供了设置画面填充模式的接口和渲染一帧纹理的接口。
- `KFGLFilter`：实现 shader 的加载、编译和着色器程序链接，以及 FBO 的管理。同时作为渲染处理节点，提供给了接口支持多级渲染。
- `KFGLProgram`：封装了使用 GL 程序的部分 API。
- `KFGLFrameBuffer`：封装了使用 FBO 的 API。
- `KFTextureFrame`：表示一帧纹理对象。
- `KFFrame`：表示一帧，类型可以是数据缓冲或纹理。
- `KFGLTextureAttributes`：对纹理 Texture 属性的封装。
- `KFGLBase`：定义了默认的 VertexShader 和 FragmentShader。


### 2.2、高斯模糊 Shader 实现

我们使用 `KFGLFilter` 为它设置高斯模糊的 Shader 来实现我们高斯模糊效果，对应的顶点着色器和片段着色器的代码如下：

`KFGLGaussianBlur.h`

```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

#define STRINGIZE(x) #x
#define STRINGIZE2(x) STRINGIZE(x)
#define SHADER_STRING(text) @ STRINGIZE2(text)

extern NSString *const KFGLGaussianBlurVertexShader;
extern NSString *const KFGLGaussianBlurFragmentShader;

NS_ASSUME_NONNULL_END
```

`KFGLGaussianBlur.m`

```objc
#import "KFGLGaussianBlur.h"

NSString *const KFGLGaussianBlurVertexShader = SHADER_STRING
(
    attribute vec4 position; // 通过 attribute 通道获取顶点信息。4 维向量。
    attribute vec4 inputTextureCoordinate; // 通过 attribute 通道获取纹理坐标信息。4 维向量。
    varying vec2 textureCoordinate; // 用于 vertex shader 和 fragment shader 间传递纹理坐标。2 维向量。

    const int GAUSSIAN_SAMPLES = 9; // 被参考的点数目。
    uniform float wOffset; // 水平方向单位偏移。Offset 越大结果越模糊。
    uniform float hOffset; // 垂直方向单位偏移。Offset 越大结果越模糊。

    varying vec2 blurCoordinates[GAUSSIAN_SAMPLES]; // 被参考点的纹理坐标数组，将在 vertex shader 和 fragment shader 间传递。2 维向量数组。

    void main()
    {
        gl_Position = position;

        textureCoordinate = inputTextureCoordinate.xy; // 将通过 attribute 通道获取的纹理坐标数据中的 2 维分量传给 fragment shader。

        int multiplier = 0;
        vec2 blurStep;
        vec2 singleStepOffset = vec2(hOffset, wOffset);

        for (int i = 0; i < GAUSSIAN_SAMPLES; i++)
        {
            multiplier = (i - ((GAUSSIAN_SAMPLES - 1) / 2)); // 每一个被参考点距离当前纹理坐标的偏移乘数
            blurStep = float(multiplier) * singleStepOffset; // 每一个被参考点距离当前纹理坐标的偏移
            blurCoordinates[i] = inputTextureCoordinate.xy + blurStep; // 每一个被参考点的纹理坐标
        }
    }
);

NSString *const KFGLGaussianBlurFragmentShader = SHADER_STRING
(
    varying highp vec2 textureCoordinate; // 从 vertex shader 传递来的纹理坐标。
    uniform sampler2D inputImageTexture; // 通过 uniform 通道获取纹理信息。2D 纹理。

    const lowp int GAUSSIAN_SAMPLES = 9; // 被参考的点数目。

    varying highp vec2 blurCoordinates[GAUSSIAN_SAMPLES]; // 从 vertex shader 传递来的被参考点的纹理坐标数组。

    void main()
    {
        lowp vec4 sum = vec4(0.0);

        // 根据距离当前点距离远近分配权重。分配原则越近权重越大。
        sum += texture2D(inputImageTexture, blurCoordinates[0]) * 0.05;
        sum += texture2D(inputImageTexture, blurCoordinates[1]) * 0.09;
        sum += texture2D(inputImageTexture, blurCoordinates[2]) * 0.12;
        sum += texture2D(inputImageTexture, blurCoordinates[3]) * 0.15;
        sum += texture2D(inputImageTexture, blurCoordinates[4]) * 0.18;
        sum += texture2D(inputImageTexture, blurCoordinates[5]) * 0.15;
        sum += texture2D(inputImageTexture, blurCoordinates[6]) * 0.12;
        sum += texture2D(inputImageTexture, blurCoordinates[7]) * 0.09;
        sum += texture2D(inputImageTexture, blurCoordinates[8]) * 0.05;

        // 加权。
        gl_FragColor = sum;
    }
);
```

### 2.3、图像转纹理

我们还需要实现一个 `KFUIImageConvertTexture` 类用于实现图片转纹理，之后再对纹理使用 OpenGL 进行处理。代码如下：

`KFUIImageConvertTexture.h`

```objc
#import <Foundation/Foundation.h>
#import <OpenGLES/EAGL.h>
#import <UIKit/UIKit.h>
#import "KFTextureFrame.h"

@interface KFUIImageConvertTexture : NSObject

+ (KFTextureFrame *)renderImage:(UIImage *)image;

@end
```


`KFUIImageConvertTexture.m`

```objc
#import "KFUIImageConvertTexture.h"

@implementation KFUIImageConvertTexture

+ (KFTextureFrame *)renderImage:(UIImage *)image {
    CGImageRef cgImageRef = [image CGImage];
    GLuint width = (GLuint)CGImageGetWidth(cgImageRef);
    GLuint height = (GLuint)CGImageGetHeight(cgImageRef);
    CGRect rect = CGRectMake(0, 0, width, height);
    
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    
    void *imageData = malloc(width * height * 4);
    CGContextRef context = CGBitmapContextCreate(imageData, width, height, 8, width * 4, colorSpace, kCGImageAlphaPremultipliedLast | kCGBitmapByteOrder32Big);
        
    CGColorSpaceRelease(colorSpace);
    CGContextClearRect(context, rect);
    CGContextDrawImage(context, rect, cgImageRef);
    
    glEnable(GL_TEXTURE_2D);
    
    GLuint textureID;
    glGenTextures(1, &textureID);
    glBindTexture(GL_TEXTURE_2D, textureID);
    
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, imageData);
    
    // 结束后要做清理
    glBindTexture(GL_TEXTURE_2D, 0); // 解绑
    CGContextRelease(context);
    free(imageData);
    
    KFTextureFrame *inputFrame = [[KFTextureFrame alloc] initWithTextureId:textureID textureSize:CGSizeMake(width, height) time:kCMTimeZero];
    return inputFrame;
}
```

### 2.4、展示高斯模糊渲染结果

我们在一个 ViewController 串联对图片进行高斯模糊处理的逻辑，并展示最后的效果。代码如下：


```objc
#import "KFGaussianBlurViewController.h"
#import "KFUIImageConvertTexture.h"
#import "KFOpenGLView.h"
#import "KFGLFilter.h"
#import "KFGLGaussianBlur.h"

@interface KFGaussianBlurViewController ()
@property (nonatomic, strong) KFOpenGLView *glView;
@property (nonatomic, strong) KFUIImageConvertTexture *imageConvertTexture;
@property (nonatomic, strong) EAGLContext *context;
@property (nonatomic, strong) KFGLFilter *verticalGaussianBlurFilter;
@property (nonatomic, strong) KFGLFilter *horizonalGaussianBlurFilter;
@end

@implementation KFGaussianBlurViewController


#pragma mark - Property
- (EAGLContext *)context {
    if (!_context) {
        _context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    }
    
    return _context;
}

- (KFUIImageConvertTexture *)imageConvertTexture {
    if (!_imageConvertTexture) {
        _imageConvertTexture = [[KFUIImageConvertTexture alloc] init];
    }
    return _imageConvertTexture;
}

- (KFGLFilter *)verticalGaussianBlurFilter {
    if (!_verticalGaussianBlurFilter) {
        _verticalGaussianBlurFilter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFGLGaussianBlurVertexShader fragmentShader:KFGLGaussianBlurFragmentShader];
        [_verticalGaussianBlurFilter setFloatUniformValue:@"hOffset" floatValue:0.00390625f];
    }
    return _verticalGaussianBlurFilter;
}

- (KFGLFilter *)horizonalGaussianBlurFilter {
    if (!_horizonalGaussianBlurFilter) {
        _horizonalGaussianBlurFilter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFGLGaussianBlurVertexShader fragmentShader:KFGLGaussianBlurFragmentShader];
        [_horizonalGaussianBlurFilter setFloatUniformValue:@"wOffset" floatValue:0.00390625f];
    }
    return _horizonalGaussianBlurFilter;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    [self setupUI];
    
    [self applyGaussianBlurEffect];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.glView.frame = self.view.bounds;
}

- (void)setupUI {
    self.edgesForExtendedLayout = UIRectEdgeAll;
    self.extendedLayoutIncludesOpaqueBars = YES;
    self.title = @"Gaussian Blur";
    self.view.backgroundColor = [UIColor whiteColor];
    
    // 渲染 view。
    _glView = [[KFOpenGLView alloc] initWithFrame:self.view.bounds context:self.context];
    _glView.fillMode = KFGLViewContentModeFit;
    [self.view addSubview:self.glView];
}

- (void)applyGaussianBlurEffect {
    [EAGLContext setCurrentContext:self.context];
    UIImage *baseImage = [UIImage imageNamed:@"KeyframeLogo"];
    KFTextureFrame *textureFrame = [KFUIImageConvertTexture renderImage:baseImage];
    
    // 垂直方向做一次高斯模糊。
    KFTextureFrame *verticalTexture = [self.verticalGaussianBlurFilter render:textureFrame];
    
    // 水平方向做一次高斯模糊。
    KFTextureFrame *horizonalTexture = [self.horizonalGaussianBlurFilter render:verticalTexture];
    
    [self.glView displayFrame:horizonalTexture];
    [EAGLContext setCurrentContext:nil];
}

@end
```

通过上面的代码，可以看到我们是用 `KFGLFilter` 来封装一次 OpenGL 的处理节点，它可以接收一个 `KFTextureFrame` 对象，加载 Shader 对其进行渲染处理，处理完后输出处理后的 `KFTextureFrame`，然后可以接着交给下一个 `KFGLFilter` 来处理，就像一条渲染链。

这里我们把高斯模糊用到的二维高斯卷积核拆成两个一维的高斯卷积核，所以是分别做了一次垂直方向和一次水平风向的处理。







## 3、Android Demo

Android 实现高斯模糊的 Demo 我们是在 [OpenGL 渲染视频 Demo](https://mp.weixin.qq.com/s/eBB6jkvsufXbIWCLwsueOQ) 的基础上在相机返回的视频帧被渲染前增加了高斯模糊的处理。对应视频采集模糊和视频渲染模块这里就不再细讲，只贴一下主要的类和类具体的功能：

- `KFGLContext`：负责创建 OpenGL 环境，负责管理和组装 EGLDisplay、EGLSurface、EGLContext。
- `KFGLFilter`：实现 shader 的加载、编译和着色器程序链接，以及 FBO 的管理。同时作为渲染处理节点，提供给了接口支持多级渲染。
- `KFGLProgram`：负责加载和编译着色器，创建着色器程序容器。
- `KFGLBase`：定义了默认的 VertexShader 和 FragmentShader。
- `KFSurfaceView`：KFSurfaceView 继承自 SurfaceView 来实现渲染。
- `KFTextureView`：KFTextureView 继承自 TextureView 来实现渲染。
- `KFFrame`：表示一帧，类型可以是数据缓冲或纹理。
- `KFRenderView`：KFRenderView 是一个容器，可以选择使用 KFSurfaceView 或 KFTextureView 作为实际的渲染视图。





实现高斯模糊的顶点着色器代码和片段着色器代码如下：

```java
public static String defaultGaussianVertexShader =
        "attribute vec4 position; \n" +
        "attribute vec4 inputTextureCoordinate; \n" +
        "varying vec2 textureCoordinate; \n" +
        "const int GAUSSIAN_SAMPLES = 9; \n" +
        "uniform float wOffset; \n" +
        "uniform float hOffset; \n" +
        "varying vec2 blurCoordinates[GAUSSIAN_SAMPLES]; \n" +
        "void main() \n" +
        "{ \n" +
        "  gl_Position = position; \n" +
        "  textureCoordinate = inputTextureCoordinate.xy; \n" +
        "  int multiplier = 0; \n" +
        "  vec2 blurStep; \n" +
        "  vec2 singleStepOffset = vec2(hOffset, wOffset); \n" +
        "  for (int i = 0; i < GAUSSIAN_SAMPLES; i++) \n" +
        "  { \n" +
        "    multiplier = (i - ((GAUSSIAN_SAMPLES - 1) / 2)); \n" +
        "    blurStep = float(multiplier) * singleStepOffset; \n" +
        "    blurCoordinates[i] = inputTextureCoordinate.xy + blurStep; \n" +
        "  } \n" +
        "} \n" ;
```



```java
public static String defaultGaussianFragmentShader =
            "varying highp vec2 textureCoordinate; \n" +
            "uniform sampler2D inputImageTexture; \n" +
            "const lowp int GAUSSIAN_SAMPLES = 9; \n" +
            "varying highp vec2 blurCoordinates[GAUSSIAN_SAMPLES]; \n" +
            "void main() \n" +
            "{\n" +
            "  lowp vec4 sum = vec4(0.0); \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[0]) * 0.05; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[1]) * 0.09; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[2]) * 0.12; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[3]) * 0.15; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[4]) * 0.18; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[5]) * 0.15; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[6]) * 0.12; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[7]) * 0.09; \n" +
            "  sum += texture2D(inputImageTexture, blurCoordinates[8]) * 0.05; \n" +
            "  gl_FragColor = sum; \n" +
            "} \n" ;
```

对 `MainActivity` 的改动则主要是在 `KFVideoCaptureListener` 的 `onFrameAvailable` 回调中增加对图像帧做高斯模糊的处理逻辑，再进行渲染即可。代码如下：


```java
public void onFrameAvailable(KFFrame frame) {
    mGLContext.bind();
    if (mVerticalGLFilter == null) {
        mVerticalGLFilter = new KFGLFilter(false, defaultGaussianVertexShader,defaultGaussianFragmentShader);
        mVerticalGLFilter.setFloatUniformValue("hOffset",0.00390625f);
    }
    if (mHoritizalGLFilter == null) {
        mHoritizalGLFilter = new KFGLFilter(false, defaultGaussianVertexShader,defaultGaussianFragmentShader);
        mHoritizalGLFilter.setFloatUniformValue("wOffset",0.00390625f);

    }
    KFFrame filterFrame = mVerticalGLFilter.render((KFTextureFrame)frame);
    KFFrame hFilterFrame = mHoritizalGLFilter.render((KFTextureFrame)filterFrame);
    mRenderView.render((KFTextureFrame) hFilterFrame);
    mGLContext.unbind();
}
```

可见，当我们用 `KFGLFilter` 将 OpenGL 渲染能力封装起来，并可以像增加渲染处理节点一样往现有渲染链中增加新的图像处理功能时，相关改动就变得很方便了。



参考：

- [高斯模糊的分离性](https://www.zhihu.com/question/364517094/answer/2258633530 "高斯模糊的分离性")
- [高斯模糊的原理](https://zhuanlan.zhihu.com/p/493628728 "高斯模糊的原理")
- [Efficient Gaussian blur with linear sampling](https://www.rastergrid.com/blog/2010/09/efficient-gaussian-blur-with-linear-sampling/ "Efficient Gaussian blur with linear sampling")
- [高斯模糊算法 ](https://www.ruanyifeng.com/blog/2012/11/gaussian_blur.html "高斯模糊算法")
- [高斯模糊 OpenGL 代码实现](https://cloud.tencent.com/developer/article/1806913 "高斯模糊OpenGL代码实现")

