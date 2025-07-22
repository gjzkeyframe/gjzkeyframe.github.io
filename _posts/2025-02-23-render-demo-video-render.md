---
title: RenderDemo（2）：用 OpenGL 渲染视频
description: 介绍用 OpenGL 渲染视频的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 13:18:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, Android, OpenGL, 渲染]
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


**渲染**是音视频技术栈相关的一个非常重要的方向，视频图像在设备上的展示、各种流行的视频特效都离不开渲染技术的支持。

在 RenderDemo 这个工程示例系列，我们将为大家展示一些渲染相关的 Demo，来向大家介绍如何在 iOS/Android 平台上手一些渲染相关的开发。

这里是第二篇：**用 OpenGL 渲染视频**。我们分别在 iOS 和 Android 实现了用 OpenGL 渲染视频数据的 Demo。在本文中，包括如下内容：

- 1）iOS 视频 OpenGL 渲染 Demo；
- 2）Android 视频 OpenGL 渲染 Demo；
- 3）详尽的代码注释，帮你理解代码逻辑和原理。

**如果你想要获得我们所有 Demo 的工程源码，可以在关注本公众号后，在公众号发送消息『AVDemo』来咨询。**




## 1、iOS Demo

其实我们在之前的 [iOS 视频采集的 Demo](https://mp.weixin.qq.com/s/CJAhkk9BmhMOXgD2pl_rjg) 中已经使用了系统的 API `AVCaptureVideoPreviewLayer` 来实现了视频数据的渲染，不过现在我们准备深入渲染的细节，所以我们这里会使用 OpenGL 来自己实现渲染模块替换掉 `AVCaptureVideoPreviewLayer`。


### 1.1、视频采集模块

视频采集模块与 [iOS 视频采集的 Demo](https://mp.weixin.qq.com/s/CJAhkk9BmhMOXgD2pl_rjg) 中讲到的一致，这里就不再细讲，只贴一下主要代码：


首先，实现一个 `KFVideoCaptureConfig` 类用于定义视频采集参数的配置。


`KFVideoCaptureConfig.h`

```objc
#import <Foundation/Foundation.h>
#import <AVFoundation/AVFoundation.h>

NS_ASSUME_NONNULL_BEGIN

typedef NS_ENUM(NSInteger, KFVideoCaptureMirrorType) {
    KFVideoCaptureMirrorNone = 0,
    KFVideoCaptureMirrorFront = 1 << 0,
    KFVideoCaptureMirrorBack = 1 << 1,
    KFVideoCaptureMirrorAll = (KFVideoCaptureMirrorFront | KFVideoCaptureMirrorBack),
};

@interface KFVideoCaptureConfig : NSObject
@property (nonatomic, copy) AVCaptureSessionPreset preset; // 视频采集参数，比如分辨率等，与画质相关。
@property (nonatomic, assign) AVCaptureDevicePosition position; // 摄像头位置，前置/后置摄像头。
@property (nonatomic, assign) AVCaptureVideoOrientation orientation; // 视频画面方向。
@property (nonatomic, assign) NSInteger fps; // 视频帧率。
@property (nonatomic, assign) OSType pixelFormatType; // 颜色空间格式。
@property (nonatomic, assign) KFVideoCaptureMirrorType mirrorType; // 镜像类型。
@end

NS_ASSUME_NONNULL_END
```


`KFVideoCaptureConfig.m`

```objc
#import "KFVideoCaptureConfig.h"

@implementation KFVideoCaptureConfig

- (instancetype)init {
    self = [super init];
    if (self) {
        _preset = AVCaptureSessionPreset1920x1080;
        _position = AVCaptureDevicePositionFront;
        _orientation = AVCaptureVideoOrientationPortrait;
        _fps = 30;
        _mirrorType = KFVideoCaptureMirrorFront;

        // 设置颜色空间格式，这里要注意了：
        // 1、一般我们采集图像用于后续的编码时，这里设置 kCVPixelFormatType_420YpCbCr8BiPlanarFullRange 即可。
        // 2、如果想支持 HDR 时（iPhone12 及之后设备才支持），这里设置为：kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange。
        _pixelFormatType = kCVPixelFormatType_420YpCbCr8BiPlanarFullRange;
    }
    
    return self;
}

@end
```


接下来，我们实现一个 `KFVideoCapture` 类来实现视频采集。


`KFVideoCapture.h`


```objc
#import <Foundation/Foundation.h>
#import "KFVideoCaptureConfig.h"

NS_ASSUME_NONNULL_BEGIN

@interface KFVideoCapture : NSObject
+ (instancetype)new NS_UNAVAILABLE;
- (instancetype)init NS_UNAVAILABLE;
- (instancetype)initWithConfig:(KFVideoCaptureConfig *)config;

@property (nonatomic, strong, readonly) KFVideoCaptureConfig *config;
@property (nonatomic, strong, readonly) AVCaptureVideoPreviewLayer *previewLayer; // 视频预览渲染 layer。
@property (nonatomic, copy) void (^sampleBufferOutputCallBack)(CMSampleBufferRef sample); // 视频采集数据回调。
@property (nonatomic, copy) void (^sessionErrorCallBack)(NSError *error); // 视频采集会话错误回调。
@property (nonatomic, copy) void (^sessionInitSuccessCallBack)(void); // 视频采集会话初始化成功回调。

- (void)startRunning; // 开始采集。
- (void)stopRunning; // 停止采集。
- (void)changeDevicePosition:(AVCaptureDevicePosition)position; // 切换摄像头。
@end

NS_ASSUME_NONNULL_END
```


`KFVideoCapture.m`

```objc
#import "KFVideoCapture.h"
#import <UIKit/UIKit.h>

@interface KFVideoCapture () <AVCaptureVideoDataOutputSampleBufferDelegate>
@property (nonatomic, strong, readwrite) KFVideoCaptureConfig *config;
@property (nonatomic, strong, readonly) AVCaptureDevice *captureDevice; // 视频采集设备。
@property (nonatomic, strong) AVCaptureDeviceInput *backDeviceInput; // 后置摄像头采集输入。
@property (nonatomic, strong) AVCaptureDeviceInput *frontDeviceInput; // 前置摄像头采集输入。
@property (nonatomic, strong) AVCaptureVideoDataOutput *videoOutput; // 视频采集输出。
@property (nonatomic, strong) AVCaptureSession *captureSession; // 视频采集会话。
@property (nonatomic, strong, readwrite) AVCaptureVideoPreviewLayer *previewLayer; // 视频预览渲染 layer。
@property (nonatomic, assign, readonly) CMVideoDimensions sessionPresetSize; // 视频采集分辨率。
@property (nonatomic, strong) dispatch_queue_t captureQueue;
@end

@implementation KFVideoCapture
#pragma mark - Property
- (AVCaptureDevice *)backCamera {
    return [self cameraWithPosition:AVCaptureDevicePositionBack];
}

- (AVCaptureDeviceInput *)backDeviceInput {
    if (!_backDeviceInput) {
        _backDeviceInput = [[AVCaptureDeviceInput alloc] initWithDevice:[self backCamera] error:nil];
    }
    
    return _backDeviceInput;
}

- (AVCaptureDevice *)frontCamera {
    return [self cameraWithPosition:AVCaptureDevicePositionFront];
}

- (AVCaptureDeviceInput *)frontDeviceInput {
    if (!_frontDeviceInput) {
        _frontDeviceInput = [[AVCaptureDeviceInput alloc] initWithDevice:[self frontCamera] error:nil];
    }
    
    return _frontDeviceInput;
}

- (AVCaptureVideoDataOutput *)videoOutput {
    if (!_videoOutput) {
        _videoOutput = [[AVCaptureVideoDataOutput alloc] init];
        [_videoOutput setSampleBufferDelegate:self queue:self.captureQueue]; // 设置返回采集数据的代理和回调。
        _videoOutput.videoSettings = @{(id)kCVPixelBufferPixelFormatTypeKey: @(_config.pixelFormatType)};
        _videoOutput.alwaysDiscardsLateVideoFrames = YES; // YES 表示：采集的下一帧到来前，如果有还未处理完的帧，丢掉。
    }

    return _videoOutput;
}

- (AVCaptureSession *)captureSession {
    if (!_captureSession) {
        AVCaptureDeviceInput *deviceInput = self.config.position == AVCaptureDevicePositionBack ? self.backDeviceInput : self.frontDeviceInput;
        if (!deviceInput) {
            return nil;
        }
        // 1、初始化采集会话。
        _captureSession = [[AVCaptureSession alloc] init];
        
        // 2、添加采集输入。
        for (AVCaptureSessionPreset selectPreset in [self sessionPresetList]) {
            if ([_captureSession canSetSessionPreset:selectPreset]) {
                [_captureSession setSessionPreset:selectPreset];
                if ([_captureSession canAddInput:deviceInput]) {
                    [_captureSession addInput:deviceInput];
                    break;
                }
            }
        }
        
        // 3、添加采集输出。
        if ([_captureSession canAddOutput:self.videoOutput]) {
            [_captureSession addOutput:self.videoOutput];
        }
        
        // 4、更新画面方向。
        [self _updateOrientation];
        
        // 5、更新画面镜像。
        [self _updateMirror];
    
        // 6、更新采集实时帧率。
        [self.captureDevice lockForConfiguration:nil];
        [self _updateActiveFrameDuration];
        [self.captureDevice unlockForConfiguration];
        
        // 7、回报成功。
        if (self.sessionInitSuccessCallBack) {
            self.sessionInitSuccessCallBack();
        }
    }
    
    return _captureSession;
}

- (AVCaptureVideoPreviewLayer *)previewLayer {
    if (!_captureSession) {
        return nil;
    }
    if (!_previewLayer) {
        // 初始化预览渲染 layer。这里就直接用系统提供的 API 来渲染。
        _previewLayer = [[AVCaptureVideoPreviewLayer alloc] initWithSession:_captureSession];
        [_previewLayer setVideoGravity:AVLayerVideoGravityResizeAspectFill];
    }
    
    return _previewLayer;
}

- (AVCaptureDevice *)captureDevice {
    // 视频采集设备。
    return (self.config.position == AVCaptureDevicePositionBack) ? [self backCamera] : [self frontCamera];
}

- (CMVideoDimensions)sessionPresetSize {
    // 视频采集分辨率。
    return CMVideoFormatDescriptionGetDimensions([self captureDevice].activeFormat.formatDescription);
}

#pragma mark - LifeCycle
- (instancetype)initWithConfig:(KFVideoCaptureConfig *)config {
    self = [super init];
    if (self) {
        _config = config;
        _captureQueue = dispatch_queue_create("com.KeyFrameKit.videoCapture", DISPATCH_QUEUE_SERIAL);
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(sessionRuntimeError:) name:AVCaptureSessionRuntimeErrorNotification object:nil];
    }
    
    return self;
}

- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

#pragma mark - Public Method
- (void)startRunning {
    typeof(self) __weak weakSelf = self;
    dispatch_async(_captureQueue, ^{
        [weakSelf _startRunning];
    });
}

- (void)stopRunning {
    typeof(self) __weak weakSelf = self;
    dispatch_async(_captureQueue, ^{
        [weakSelf _stopRunning];
    });
}

- (void)changeDevicePosition:(AVCaptureDevicePosition)position {
    typeof(self) __weak weakSelf = self;
    dispatch_async(_captureQueue, ^{
        [weakSelf _updateDeveicePosition:position];
    });
}

#pragma mark - Private Method
- (void)_startRunning {
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (status == AVAuthorizationStatusAuthorized) {
        if (!self.captureSession.isRunning) {
            [self.captureSession startRunning];
        }
    } else {
        NSLog(@"没有相机使用权限");
    }
}

- (void)_stopRunning {
    if (_captureSession && _captureSession.isRunning) {
        [_captureSession stopRunning];
    }
}

- (void)_updateDeveicePosition:(AVCaptureDevicePosition)position {
    // 切换采集的摄像头。
    
    if (position == self.config.position || !_captureSession.isRunning) {
        return;
    }
    
    // 1、切换采集输入。
    AVCaptureDeviceInput *curInput = self.config.position == AVCaptureDevicePositionBack ? self.backDeviceInput : self.frontDeviceInput;
    AVCaptureDeviceInput *addInput = self.config.position == AVCaptureDevicePositionBack ? self.frontDeviceInput : self.backDeviceInput;
    if (!curInput || !addInput) {
        return;
    }
    [self.captureSession removeInput:curInput];
    for (AVCaptureSessionPreset selectPreset in [self sessionPresetList]) {
        if ([_captureSession canSetSessionPreset:selectPreset]) {
            [_captureSession setSessionPreset:selectPreset];
            if ([_captureSession canAddInput:addInput]) {
                [_captureSession addInput:addInput];
                self.config.position = position;
                break;
            }
        }
    }
    
    // 2、更新画面方向。
    [self _updateOrientation];
    
    // 3、更新画面镜像。
    [self _updateMirror];

    // 4、更新采集实时帧率。
    [self.captureDevice lockForConfiguration:nil];
    [self _updateActiveFrameDuration];
    [self.captureDevice unlockForConfiguration];
}

- (void)_updateOrientation {
    // 更新画面方向。
    AVCaptureConnection *connection = [self.videoOutput connectionWithMediaType:AVMediaTypeVideo]; // AVCaptureConnection 用于把输入和输出连接起来。
    if ([connection isVideoOrientationSupported] && connection.videoOrientation != self.config.orientation) {
        connection.videoOrientation = self.config.orientation;
    }
}

- (void)_updateMirror {
    // 更新画面镜像。
    AVCaptureConnection *connection = [self.videoOutput connectionWithMediaType:AVMediaTypeVideo];
    if ([connection isVideoMirroringSupported]) {
        if ((self.config.mirrorType & KFVideoCaptureMirrorFront) && self.config.position == AVCaptureDevicePositionFront) {
            connection.videoMirrored = YES;
        } else if ((self.config.mirrorType & KFVideoCaptureMirrorBack) && self.config.position == AVCaptureDevicePositionBack) {
            connection.videoMirrored = YES;
        } else {
            connection.videoMirrored = NO;
        }
    }
}

- (BOOL)_updateActiveFrameDuration {
    // 更新采集实时帧率。
    
    // 1、帧率换算成帧间隔时长。
    CMTime frameDuration = CMTimeMake(1, (int32_t) self.config.fps);
    
    // 2、设置帧率大于 30 时，找到满足该帧率及其他参数，并且当前设备支持的 AVCaptureDeviceFormat。
    if (self.config.fps > 30) {
        for (AVCaptureDeviceFormat *vFormat in [self.captureDevice formats]) {
            CMFormatDescriptionRef description = vFormat.formatDescription;
            CMVideoDimensions dims = CMVideoFormatDescriptionGetDimensions(description);
            float maxRate = ((AVFrameRateRange *) [vFormat.videoSupportedFrameRateRanges objectAtIndex:0]).maxFrameRate;
            if (maxRate >= self.config.fps && CMFormatDescriptionGetMediaSubType(description) == self.config.pixelFormatType && self.sessionPresetSize.width * self.sessionPresetSize.height == dims.width * dims.height) {
                self.captureDevice.activeFormat = vFormat;
                break;
            }
        }
    }
    
    // 3、检查设置的帧率是否在当前设备的 activeFormat 支持的最低和最高帧率之间。如果是，就设置帧率。
    __block BOOL support = NO;
    [self.captureDevice.activeFormat.videoSupportedFrameRateRanges enumerateObjectsUsingBlock:^(AVFrameRateRange * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (CMTimeCompare(frameDuration, obj.minFrameDuration) >= 0 &&
            CMTimeCompare(frameDuration, obj.maxFrameDuration) <= 0) {
            support = YES;
            *stop = YES;
        }
    }];
    if (support) {
        [self.captureDevice setActiveVideoMinFrameDuration:frameDuration];
        [self.captureDevice setActiveVideoMaxFrameDuration:frameDuration];
        return YES;
    }
    
    return NO;
}

#pragma mark - NSNotification
- (void)sessionRuntimeError:(NSNotification *)notification {
    if (self.sessionErrorCallBack) {
        self.sessionErrorCallBack(notification.userInfo[AVCaptureSessionErrorKey]);
    }
}

#pragma mark - Utility
- (AVCaptureDevice *)cameraWithPosition:(AVCaptureDevicePosition)position {
    // 从当前手机寻找符合需要的采集设备。
    NSArray *devices = nil;
    NSString *version = [UIDevice currentDevice].systemVersion;
    if (version.doubleValue >= 10.0) {
        AVCaptureDeviceDiscoverySession *deviceDiscoverySession = [AVCaptureDeviceDiscoverySession  discoverySessionWithDeviceTypes:@[AVCaptureDeviceTypeBuiltInWideAngleCamera] mediaType:AVMediaTypeVideo position:position];
        devices = deviceDiscoverySession.devices;
    } else {
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wdeprecated-declarations"
        devices = [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo];
#pragma GCC diagnostic pop
    }
    
    for (AVCaptureDevice *device in devices) {
        if ([device position] == position) {
            return device;
        }
    }
    
    return nil;
}

- (NSArray *)sessionPresetList {
    return @[self.config.preset, AVCaptureSessionPreset3840x2160, AVCaptureSessionPreset1920x1080, AVCaptureSessionPreset1280x720, AVCaptureSessionPresetLow];
}

#pragma mark - AVCaptureVideoDataOutputSampleBufferDelegate
- (void)captureOutput:(AVCaptureOutput *)output didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection {
    // 向外回调数据。
    if (output == self.videoOutput) {
        if (self.sampleBufferOutputCallBack) {
            self.sampleBufferOutputCallBack(sampleBuffer);
        }
    }
}

@end
```

上面是 `KFVideoCapture` 的实现。



### 1.2、视频渲染模块


**1）渲染视图 KFOpenGLView**


接下来，我们来用 OpenGL 实现一个支持视频数据渲染的 View，对应的接口如下：

`KFOpenGLView.h`


```objc
#import <UIKit/UIKit.h>
#import <OpenGLES/ES2/gl.h>
#import <OpenGLES/ES2/glext.h>
#import "KFTextureFrame.h"

// 渲染画面填充模式。
typedef NS_ENUM(NSInteger, KFGLViewContentMode) {
    // 自动填充满，可能会变形。
    KFGLViewContentModeStretch = 0,
    // 按比例适配，可能会有黑边。
    KFGLViewContentModeFit = 1,
    // 根据比例裁剪后填充满。
    KFGLViewContentModeFill = 2
};

// 使用 OpenGL 实现渲染 View。
@interface KFOpenGLView : UIView

- (instancetype)initWithFrame:(CGRect)frame context:(nullable EAGLContext *)context;

@property (nonatomic, assign) KFGLViewContentMode fillMode; // 画面填充模式。

- (void)displayFrame:(nonnull KFTextureFrame *)frame; // 渲染一帧纹理。

@end
```

核心功能是提供了设置画面填充模式的接口和渲染一帧纹理的接口。下面是对应的实现：

`KFOpenGLView.m`

```objc
#import "KFOpenGLView.h"
#import <QuartzCore/QuartzCore.h>
#import <AVFoundation/AVUtilities.h>
#import <mach/mach_time.h>
#import <GLKit/GLKit.h>
#import "KFGLFilter.h"
#import "KFGLBase.h"
#import <GLKit/GLKit.h>

@interface KFOpenGLView() {
    // The pixel dimensions of the CAEAGLLayer.
    GLint _backingWidth;
    GLint _backingHeight;
    
    GLuint _frameBufferHandle;
    GLuint _colorBufferHandle;
    
    KFGLFilter *_filter;
    GLfloat _customVertices[8];
}

@property (nonatomic, assign) CGSize currentViewSize; // 当前 view 大小。
@property (nonatomic, assign) CGSize frameSize; // 当前被渲染的纹理大小。

@end

@implementation KFOpenGLView

+ (Class)layerClass {
    return [CAEAGLLayer class];
}

- (instancetype)initWithFrame:(CGRect)frame context:(nullable EAGLContext *)context{
    if (self = [super initWithFrame:frame]) {
        self.contentScaleFactor = [[UIScreen mainScreen] scale];
        // 设定 layer 相关属性。
        CAEAGLLayer *eaglLayer = (CAEAGLLayer *) self.layer;
        eaglLayer.opaque = YES;
        eaglLayer.drawableProperties = @{ kEAGLDrawablePropertyRetainedBacking: @(NO),
                                          kEAGLDrawablePropertyColorFormat: kEAGLColorFormatRGBA8};
        _fillMode = KFGLViewContentModeFit;
        
        // 设置当前 OpenGL 上下文，并初始化相关 GL 环境。
        if (context) {
            EAGLContext *preContext = [EAGLContext currentContext];
            [EAGLContext setCurrentContext:context];
            [self _setupGL];
            [EAGLContext setCurrentContext:preContext];
        } else {
            NSLog(@"KFOpenGLView context nil");
        }
    }
    
    return self;
}

- (void)layoutSubviews {
    // 视图自动调整布局，同步至渲染视图。
    [super layoutSubviews];
    _currentViewSize = self.bounds.size;
}

- (void)dealloc {
    if(_frameBufferHandle != 0){
        glDeleteFramebuffers(1, &_frameBufferHandle);
    }
    if(_colorBufferHandle != 0){
        glDeleteRenderbuffers(1, &_colorBufferHandle);
    }
}

# pragma mark - OpenGL Setup
- (void)_setupGL {
    // 1、申请并绑定帧缓冲区对象 FBO。
    glGenFramebuffers(1, &_frameBufferHandle);
    glBindFramebuffer(GL_FRAMEBUFFER, _frameBufferHandle);
    
    // 2、申请并绑定渲染缓冲区对象 RBO。
    glGenRenderbuffers(1, &_colorBufferHandle);
    glBindRenderbuffer(GL_RENDERBUFFER, _colorBufferHandle);
    
    // 3、将渲染图层（_eaglLayer）的存储绑定到 RBO。
    [[EAGLContext currentContext] renderbufferStorage:GL_RENDERBUFFER fromDrawable:(CAEAGLLayer *)self.layer];
    // 当渲染缓冲区 RBO 绑定存储空间完成后，可以通过 glGetRenderbufferParameteriv 获取渲染缓冲区的宽高，实际跟上面设置的 layer 的宽高一致。
    glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_WIDTH, &_backingWidth);
    glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_HEIGHT, &_backingHeight);

    // 4、将 RBO 绑定为 FBO 的一个附件。绑定后，OpenGL 对 FBO 的绘制会同步到 RBO 后再上屏。
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_RENDERBUFFER, _colorBufferHandle);
    if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE) {
        NSLog(@"Failed to make complete framebuffer object %x", glCheckFramebufferStatus(GL_FRAMEBUFFER));
    }
    
    // 5、KFGLFilter 封装了 shader 的加载、编译和着色器程序链接，以及 FBO 的管理。这里用一个 Filter 来实现具体的渲染细节。
    _filter = [[KFGLFilter alloc] initWithCustomFBO:YES vertexShader:KFDefaultVertexShader fragmentShader:KFDefaultFragmentShader]; // 这里 isCustomFBO 传 YES，表示直接用外部的 FBO（即上面创建的 FBO 对象 _frameBufferHandle）。vertexShader 和 fragmentShader 则都使用默认的。
    __weak typeof(self) wself = self;
    _filter.preDrawCallBack = ^(){
        // 在渲染前回调中，关联顶点位置数据。通过渲染回调接口，可以在外部更新顶点数据。
        __strong typeof(wself) sself = wself;
        if (sself) {
            glVertexAttribPointer([[sself->_filter getProgram] getAttribLocation:@"position"], 2, GL_FLOAT, 0, 0, sself->_customVertices);
        }
    };
}

- (void)_updaterVertices {
    // 根据视频画面填充模式计算顶点数据。
    float heightScaling = 1.0;
    float widthScaling = 1.0;
    
    if (!CGSizeEqualToSize(_currentViewSize, CGSizeZero) && !CGSizeEqualToSize(_frameSize, CGSizeZero)) {
        CGRect insetRect = AVMakeRectWithAspectRatioInsideRect(_frameSize, CGRectMake(0, 0, _currentViewSize.width, _currentViewSize.height));
        
        switch (_fillMode) {
            case KFGLViewContentModeStretch: {
                widthScaling = 1.0;
                heightScaling = 1.0;
                break;
            }
            case KFGLViewContentModeFit: {
                widthScaling = insetRect.size.width / _currentViewSize.width;
                heightScaling = insetRect.size.height / _currentViewSize.height;
                break;
            }
            case KFGLViewContentModeFill: {
                widthScaling = _currentViewSize.height / insetRect.size.height;
                heightScaling = _currentViewSize.width / insetRect.size.width;
                break;
            }
        }
    }
    
    _customVertices[0] = -widthScaling;
    _customVertices[1] = -heightScaling;
    _customVertices[2] = widthScaling;
    _customVertices[3] = -heightScaling;
    _customVertices[4] = -widthScaling;
    _customVertices[5] = heightScaling;
    _customVertices[6] = widthScaling;
    _customVertices[7] = heightScaling;
}

#pragma mark - OpenGLES Render
// 渲染一帧纹理。
- (void)displayFrame:(KFTextureFrame *)frame {
    if (![EAGLContext currentContext] || !frame) {
        return;
    }
    
    // 1、绑定 FBO、RBO 到 OpenGL 渲染管线。
    glBindFramebuffer(GL_FRAMEBUFFER, _frameBufferHandle);
    glBindRenderbuffer(GL_RENDERBUFFER, _colorBufferHandle);
    
    // 2、设置视口大小为整个渲染缓冲区的区域。
    glViewport(0, 0, _backingWidth, _backingHeight);
    
    // 3、渲染传进来的一帧纹理。
    KFTextureFrame *renderFrame = frame.copy; // 获取纹理。
    _frameSize = renderFrame.textureSize; // 记录纹理大小。
    
    // 将 GL 的坐标系（↑→）适配屏幕坐标系（↓→），生成新的 mvp 矩阵。
    GLKVector4 scale = {1, -1, 1, 1};
    renderFrame.mvpMatrix = GLKMatrix4ScaleWithVector4(GLKMatrix4Identity, scale);
    
    [self _updaterVertices]; // 更新一下顶点位置数据。外部如何更改了画面填充模式会影响顶点位置。
    [_filter render:renderFrame]; // 渲染。
    
    // 4、把 RBO 的内容显示到窗口系统 (CAEAGLLayer) 中。
    [[EAGLContext currentContext] presentRenderbuffer:GL_RENDERBUFFER];
 
    // 5、将 FBO、RBO 从 OpenGL 渲染管线解绑。
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    glBindRenderbuffer(GL_RENDERBUFFER, 0);
}

@end
```




相关的代码解释，我们已经在代码注释里进行了说明。这里面最关键部分是我们不再将所有的 OpenGL 代码都放在一个类里，而是根据功能模块进行了封装。在 `KFOpenGLView` 中，除了常规的 OpenGL 环境初始化，我们封装了一个 `KFGLFilter` 类实现 shader 的加载、编译和着色器程序链接，以及 FBO 的管理，并用一个 KFGLFilter 示例去完成具体的渲染细节。


**2）渲染节点 KFGLFilter**


KFGLFilter 的代码如下：

`KFGLFilter.h`

```objc
#import <Foundation/Foundation.h>
#import "KFGLFrameBuffer.h"
#import "KFGLProgram.h"
#import "KFTextureFrame.h"

NS_ASSUME_NONNULL_BEGIN

// KFGLFilter 封装了 shader 的加载、编译和着色器程序链接，以及 FBO 的管理
@interface KFGLFilter : NSObject

// KFGLFilter 初始化。
// 这里 isCustomFBO 传 YES，表示直接用外部的 FBO（即上面创建的 FBO 对象 _frameBufferHandle）。
- (instancetype)initWithCustomFBO:(BOOL)isCustomFBO vertexShader:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader;
- (instancetype)initWithCustomFBO:(BOOL)isCustomFBO vertexShader:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader textureAttributes:(KFGLTextureAttributes *)textureAttributes;

@property (nonatomic, copy) void (^preDrawCallBack)(void); // 渲染前回调。
@property (nonatomic, copy) void (^postDrawCallBack)(void); // 渲染后回调。

- (KFGLFrameBuffer *)getOutputFrameBuffer; // 获取内部的 FBO。
- (KFGLProgram *)getProgram; // 获取 GL 程序。
- (KFTextureFrame *)render:(KFTextureFrame*)frame; // 渲染一帧纹理。

// 设置 GL 程序变量值。
- (void)setIntegerUniformValue:(NSString *)uniformName intValue:(int)intValue;
- (void)setFloatUniformValue:(NSString *)uniformName floatValue:(float)floatValue;

@end

NS_ASSUME_NONNULL_END
```


从 `KFGLFilter` 的接口设计中我们可以看到主要提供了获取内部的 FBO、获取 GL 程序、设置 GL 程序变量值、渲染一帧纹理、渲染前回调、渲染后回调等接口。具体实现代码如下：


`KFGLFilter.mm`


```objc
#import "KFGLFilter.h"
#import <OpenGLES/ES2/glext.h>

@interface KFGLFilter() {
    BOOL _mIsCustomFBO;
    KFGLFrameBuffer *_mFrameBuffer;
    KFGLProgram *_mProgram;
    KFGLTextureAttributes *_mGLTextureAttributes;

    int _mTextureUniform;
    int _mPostionMatrixUniform;
    int _mPositionAttribute;
    int _mTextureCoordinateAttribute;
}

@end

@implementation KFGLFilter

- (instancetype)initWithCustomFBO:(BOOL)isCustomFBO vertexShader:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader {
    return [self initWithCustomFBO:isCustomFBO vertexShader:vertexShader fragmentShader:fragmentShader textureAttributes:[KFGLTextureAttributes new]];
}

- (instancetype)initWithCustomFBO:(BOOL)isCustomFBO vertexShader:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader textureAttributes:(KFGLTextureAttributes *)textureAttributes {
    self = [super init];
    if (self) {
        // 初始化。
        _mTextureUniform = -1;
        _mPostionMatrixUniform = -1;
        _mPositionAttribute = -1;
        _mTextureCoordinateAttribute = -1;
        _mIsCustomFBO = isCustomFBO;
        _mGLTextureAttributes = textureAttributes;
        // 加载和编译 shader，并链接到着色器程序。
        [self _setupProgram:vertexShader fragmentShader:fragmentShader];
    }
    return self;
}

- (void)dealloc {
    if (_mFrameBuffer != nil) {
        _mFrameBuffer = nil;
    }

    if (_mProgram != nil) {
        _mProgram = nil;
    }
}

- (KFGLFrameBuffer *)getOutputFrameBuffer {
    // 当没有指定外部 FBO 时，内部会生成一个 FBO，这里返回的是内部的 FBO。
    return _mFrameBuffer;
}

-(KFGLProgram *)getProgram {
    // 返回 GL 程序。
    return _mProgram;
}

- (void)setIntegerUniformValue:(NSString *)uniformName intValue:(int)intValue {
    // 设置 GL 程序变量值。
    if (_mProgram != nil) {
        int uniforamIndex = [_mProgram getUniformLocation:uniformName];
        [_mProgram use];
        glUniform1i(uniforamIndex, intValue);
    }
}

- (void)setFloatUniformValue:(NSString *)uniformName floatValue:(float)floatValue {
    // 设置 GL 程序变量值。
    if (_mProgram != nil) {
        int uniforamIndex = [_mProgram getUniformLocation:uniformName];
        [_mProgram use];
        glUniform1f(uniforamIndex, floatValue);
    }
}

- (void)_setupFrameBuffer:(CGSize)size {
    // 如果指定使用外部的 FBO，则这里就直接返回。
    if (_mIsCustomFBO) {
        return;
    }

    // 如果没指定使用外部的 FBO，这里就再创建一个 FBO。
    if (_mFrameBuffer == nil || _mFrameBuffer.getSize.width != size.width || _mFrameBuffer.getSize.height != size.height) {
        if (_mFrameBuffer != nil) {
            _mFrameBuffer = nil;
        }

        _mFrameBuffer = [[KFGLFrameBuffer alloc] initWithSize:size textureAttributes:_mGLTextureAttributes];
    }
}

- (void)_setupProgram:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader {
    // 加载和编译 shader，并链接到着色器程序。
    if (_mProgram == nil) {
        _mProgram = [[KFGLProgram alloc] initWithVertexShader:vertexShader fragmentShader:fragmentShader];
        // 获取与 Shader 中对应参数的位置值：
        _mTextureUniform = [_mProgram getUniformLocation:@"inputImageTexture"];
        _mPostionMatrixUniform = [_mProgram getUniformLocation:@"mvpMatrix"];
        _mPositionAttribute = [_mProgram getAttribLocation:@"position"];
        _mTextureCoordinateAttribute = [_mProgram getAttribLocation:@"inputTextureCoordinate"];
    }
}

- (KFTextureFrame *)render:(KFTextureFrame *)frame {
    // 渲染一帧纹理。
    
    if (frame == nil) {
        return frame;
    }

    KFTextureFrame *resultFrame = frame.copy;
    [self _setupFrameBuffer:frame.textureSize];

    if (_mFrameBuffer != nil) {
        [_mFrameBuffer bind];
    }

    if (_mProgram != nil) {
        // 使用 GL 程序。
        [_mProgram use];
        
        // 清理窗口颜色。
        glClearColor(0, 0, 0, 1);
        glClear(GL_COLOR_BUFFER_BIT);

        // 激活和绑定纹理单元，并设置 uniform 采样器与之对应。
        glActiveTexture(GL_TEXTURE1); // 在绑定纹理之前先激活纹理单元。默认激活的纹理单元是 GL_TEXTURE0，这里激活了 GL_TEXTURE1。
        glBindTexture(GL_TEXTURE_2D, frame.textureId); // 绑定这个纹理到当前激活的纹理单元 GL_TEXTURE1。
        glUniform1i(_mTextureUniform, 1); // 设置 _mTextureUniform 的对应的纹理单元为 1，即 GL_TEXTURE1，从而保证每个 uniform 采样器对应着正确的纹理单元。

        if (_mPostionMatrixUniform >= 0) {
            glUniformMatrix4fv(_mPostionMatrixUniform, 1, false, frame.mvpMatrix.m); // 把矩阵数据发送给着色器对应的参数。
        }

        // 启用顶点位置属性通道。
        glEnableVertexAttribArray(_mPositionAttribute);
        // 启用纹理坐标属性通道。
        glEnableVertexAttribArray(_mTextureCoordinateAttribute);

        static const GLfloat squareVertices[] = {
            -1.0f, -1.0f,
            1.0f, -1.0f,
            -1.0f,  1.0f,
            1.0f,  1.0f,
        };
        // 关联顶点位置数据。
        glVertexAttribPointer(_mPositionAttribute, 2, GL_FLOAT, 0, 0, squareVertices);
        
        static GLfloat textureCoordinates[] = {
            0.0f, 0.0f,
            1.0f, 0.0f,
            0.0f, 1.0f,
            1.0f, 1.0f,
        };
        // 关联纹理坐标数据。
        glVertexAttribPointer(_mTextureCoordinateAttribute, 2, GL_FLOAT, 0, 0, textureCoordinates);

        // 绘制前回调。回调中可以更新绘制需要的相关数据。
        if (self.preDrawCallBack) {
            self.preDrawCallBack();
        }
        
        // 绘制所有图元。
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
        
        // 绘制后回调。
        if (self.postDrawCallBack) {
            self.postDrawCallBack();
        }

        // 解绑纹理。
        glBindTexture(GL_TEXTURE_2D, 0);

        // 关闭顶点位置属性通道。
        glDisableVertexAttribArray(_mPositionAttribute);
        // 关闭纹理坐标属性通道。
        glDisableVertexAttribArray(_mTextureCoordinateAttribute);
    }

    if (_mFrameBuffer != nil) {
        // 解绑内部 FBO。
        [_mFrameBuffer unbind];
    }

    if (_mFrameBuffer != nil) {
        // 清理内部 FBO。
        resultFrame.textureId = _mFrameBuffer.getTextureId;
        resultFrame.textureSize = _mFrameBuffer.getSize;
    }
    
    // 返回渲染好的纹理。
    return resultFrame;
}

@end
```

从上面的实现代码中可以看到，KFGLFilter 的核心接口是 `- (KFTextureFrame *)render:(KFTextureFrame *)frame`，这个接口接受输入一帧纹理，进行渲染处理后再输出一帧纹理，这也是 KFGLFilter 的核心功能。这样一来，KFGLFilter 作为一个渲染处理节点，可以支持多个节点串起来做更复杂的处理。


KFGLFilter 提供的获取内部的 FBO、获取 GL 程序、设置 GL 程序变量值、渲染一帧纹理、渲染前回调、渲染后回调等接口则可以支持该渲染节点与外部的数据交互。


**3）OpenGL 模块：KFGLProgram、KFGLFrameBuffer、KFTextureFrame、KFGLTextureAttributes**

KFGLFilter 中我们还使用了 `KFGLProgram`、`KFGLFrameBuffer`、`KFTextureFrame`、`KFGLTextureAttributes` 以及一些基础定义类，他们都是对 OpenGL API 的一些封装：

- `KFGLProgram`：封装了使用 GL 程序的部分 API。
- `KFGLFrameBuffer`：封装了使用 FBO 的 API。
- `KFTextureFrame`：表示一帧纹理对象。
- `KFFrame`：表示一帧，类型可以是数据缓冲或纹理。
- `KFGLTextureAttributes`：对纹理 Texture 属性的封装。
- `KFGLBase`：定义了默认的 VertexShader 和 FragmentShader。
- `KFPixelBufferConvertTexture`：将 CVPixelBuffer 转换为纹理 Texture 的工具类，兼容颜色空间的转换处理。

对应代码如下：


`KFGLProgram.h`

```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

// KFGLProgram 封装了使用 GL 程序的部分 API
@interface KFGLProgram : NSObject

- (instancetype)initWithVertexShader:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader;

- (void)use; // 使用 GL 程序
- (int)getUniformLocation:(NSString *)name; // 根据名字获取 uniform 位置值
- (int)getAttribLocation:(NSString *)name; // 根据名字获取 attribute 位置值

@end

NS_ASSUME_NONNULL_END

```

`KFGLProgram.mm`

```objc
#import "KFGLProgram.h"
#import <OpenGLES/EAGL.h>
#import <OpenGLES/ES2/gl.h>

@interface KFGLProgram () {
    int _mProgram;
    int _mVertexShader;
    int _mFragmentShader;
}

@end

@implementation KFGLProgram

- (instancetype)initWithVertexShader:(NSString *)vertexShader fragmentShader:(NSString *)fragmentShader {
    self = [super init];
    if (self) {
        [self _createProgram:vertexShader fragmentSource:fragmentShader];
    }
    return self;
}

- (void)dealloc {
    if (_mVertexShader != 0) {
        glDeleteShader(_mVertexShader);
        _mVertexShader = 0;
    }

    if (_mFragmentShader != 0) {
        glDeleteShader(_mFragmentShader);
        _mFragmentShader = 0;
    }

    if (_mProgram != 0) {
        glDeleteProgram(_mProgram);
        _mProgram = 0;
    }
}

// 使用 GL 程序。
- (void)use {
    if (_mProgram != 0) {
        glUseProgram(_mProgram);
    }
}

// 根据名字获取 uniform 位置值
- (int)getUniformLocation:(NSString *)name {
    return glGetUniformLocation(_mProgram, [name UTF8String]);
}

// 根据名字获取 attribute 位置值
- (int)getAttribLocation:(NSString *)name {
    return glGetAttribLocation(_mProgram, [name UTF8String]);
}

// 加载和编译 shader，并链接 GL 程序。
- (void)_createProgram:(NSString *)vertexSource fragmentSource:(NSString *)fragmentSource {
    _mVertexShader = [self _loadShader:GL_VERTEX_SHADER source:vertexSource];
    _mFragmentShader = [self _loadShader:GL_FRAGMENT_SHADER source:fragmentSource];

    if (_mVertexShader != 0 && _mFragmentShader != 0) {
        _mProgram = glCreateProgram();
        glAttachShader(_mProgram, _mVertexShader);
        glAttachShader(_mProgram, _mFragmentShader);

        glLinkProgram(_mProgram);
        GLint linkStatus;
        glGetProgramiv(_mProgram, GL_LINK_STATUS, &linkStatus);
        if (linkStatus != GL_TRUE) {
            glDeleteProgram(_mProgram);
            _mProgram = 0;
        }
    }
}

// 加载和编译 shader。
- (int)_loadShader:(int)shaderType source:(NSString *)source {
    int shader = glCreateShader(shaderType);
    const GLchar *cSource = (GLchar *) [source UTF8String];
    glShaderSource(shader,1, &cSource,NULL);
    glCompileShader(shader);

    GLint compiled;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &compiled);
    if (compiled != GL_TRUE) {
        glDeleteShader(shader);
        shader = 0;
    }

    return shader;
}

@end
```

`KFGLFrameBuffer.h`

```objc
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
#import "KFGLTextureAttributes.h"

NS_ASSUME_NONNULL_BEGIN

// 封装了对 FBO 使用的 API
@interface KFGLFrameBuffer : NSObject

- (instancetype)initWithSize:(CGSize)size;
- (instancetype)initWithSize:(CGSize)size textureAttributes:(KFGLTextureAttributes *)textureAttributes;
- (CGSize)getSize; // 纹理 size
- (GLuint)getTextureId; // 纹理 id
- (void)bind; // 绑定 FBO
- (void)unbind; // 解绑 FBO

@end

NS_ASSUME_NONNULL_END
```

`KFGLFrameBuffer.mm`

```objc
#import "KFGLFrameBuffer.h"
#import <OpenGLES/EAGL.h>
#import <OpenGLES/ES2/gl.h>

@interface KFGLFrameBuffer () {
    GLuint _mTextureId;
    GLuint _mFboId;
    KFGLTextureAttributes *_mTextureAttributes;
    CGSize _mSize;
    int _mLastFboId;
}

@end

@implementation KFGLFrameBuffer

- (instancetype)initWithSize:(CGSize)size {
    return [self initWithSize:size textureAttributes:[KFGLTextureAttributes new]];
}

- (instancetype)initWithSize:(CGSize)size textureAttributes:(KFGLTextureAttributes*)textureAttributes{
    self = [super init];
    if (self) {
        _mTextureId = -1;
        _mFboId = -1;
        _mLastFboId = -1;
        _mSize = size;
        _mTextureAttributes = textureAttributes;
        [self _setup];
    }
    return self;
}

- (void)dealloc {
    if (_mTextureId != -1) {
        glDeleteTextures(1, &_mTextureId);
        _mTextureId = -1;
    }

    if (_mFboId != -1) {
        glDeleteFramebuffers(1, &_mFboId);
        _mFboId = -1;
    }
}

- (CGSize)getSize {
    return _mSize;
}

- (GLuint)getTextureId {
    return _mTextureId;
}

- (void)bind {
    glGetIntegerv(GL_FRAMEBUFFER_BINDING, &_mLastFboId);
    if (_mFboId != -1) {
        glBindFramebuffer(GL_FRAMEBUFFER, _mFboId);
        glViewport(0, 0, _mSize.width, _mSize.height);
    }
}

- (void)unbind {
    glBindFramebuffer(GL_FRAMEBUFFER, _mLastFboId);
}

- (void)_setup {
    [self _setupTexture];
    [self _setupFrameBuffer];
    [self _bindTexture2FrameBuffer];
}

-(void)_setupTexture {
    if (_mTextureId == -1) {
        glGenTextures(1, &_mTextureId);
        glActiveTexture(GL_TEXTURE0);
        glBindTexture(GL_TEXTURE_2D, _mTextureId);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, _mTextureAttributes.minFilter);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, _mTextureAttributes.magFilter);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, _mTextureAttributes.wrapS);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, _mTextureAttributes.wrapT);
        if ((int)_mSize.width % 4 != 0) {
            glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
        }
        glTexImage2D(GL_TEXTURE_2D, 0, _mTextureAttributes.internalFormat, _mSize.width, _mSize.height, 0, _mTextureAttributes.format, _mTextureAttributes.type, NULL);
        glBindTexture(GL_TEXTURE_2D, 0);
    }
}

- (void)_setupFrameBuffer {
    if (_mFboId == -1) {
        glGenFramebuffers(1, &_mFboId);
    }
}
 
- (void)_bindTexture2FrameBuffer {
    if (_mFboId != -1 && _mTextureId != -1 && _mSize.width != 0 && _mSize.height != 0) {
        glBindFramebuffer(GL_FRAMEBUFFER, _mFboId);
        glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, _mTextureId, 0);
        GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
        if (status != GL_FRAMEBUFFER_COMPLETE) {
            NSAssert(status == GL_FRAMEBUFFER_COMPLETE, @"Incomplete filter FBO: %d", status);
        }
        glBindFramebuffer(GL_FRAMEBUFFER, 0);
    }
}

@end
```

`KFTextureFrame.h`

```objc
#import "KFFrame.h"
#import <UIKit/UIKit.h>
#import <CoreMedia/CoreMedia.h>
#import <GLKit/GLKit.h>

NS_ASSUME_NONNULL_BEGIN

// 表示一帧纹理对象
@interface KFTextureFrame : KFFrame

@property (nonatomic, assign) CGSize textureSize;
@property (nonatomic, assign) GLuint textureId;
@property (nonatomic, assign) CMTime time;
@property (nonatomic, assign) GLKMatrix4 mvpMatrix;

- (instancetype)initWithTextureId:(GLuint)textureId textureSize:(CGSize)textureSize time:(CMTime)time;

@end

NS_ASSUME_NONNULL_END
```

`KFTextureFrame.m`

```objc
#import "KFTextureFrame.h"

@implementation KFTextureFrame

- (instancetype)initWithTextureId:(GLuint)textureId textureSize:(CGSize)textureSize time:(CMTime)time {
    self = [super init];
    if(self){
        _textureId = textureId;
        _textureSize = textureSize;
        _time = time;
        _mvpMatrix = GLKMatrix4Identity;
    }
    return self;
}

- (id)copyWithZone:(NSZone *)zone {
    KFTextureFrame *copy = [[KFTextureFrame allocWithZone:zone] init];
    copy.textureId = _textureId;
    copy.textureSize = _textureSize;
    copy.time = _time;
    copy.mvpMatrix = _mvpMatrix;
    return copy;
}

@end
```


`KFFrame.h`


```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

typedef NS_ENUM(NSInteger, KFFrameType) {
    KFFrameBuffer = 0, // 数据缓冲区类型
    KFFrameTexture = 1, // 纹理类型
};

@interface KFFrame : NSObject

@property (nonatomic, assign) KFFrameType frameType;

- (instancetype)initWithType:(KFFrameType)type;

@end

NS_ASSUME_NONNULL_END
```


`KFFrame.m`


```objc
#import "KFFrame.h"

@implementation KFFrame

- (instancetype)initWithType:(KFFrameType)type {
    self = [super init];
    if(self){
        _frameType = type;
    }
    return self;
}

- (instancetype)init {
    self = [super init];
    if(self){
        _frameType = KFFrameBuffer;
    }
    return self;
}

@end
```

`KFGLTextureAttributes.h`

```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

// 对纹理 Texture 属性的封装
@interface KFGLTextureAttributes : NSObject

@property(nonatomic, assign) int minFilter; // GL_TEXTURE_MIN_FILTER，多个纹素对应一个片元时的处理方式
@property(nonatomic, assign) int magFilter; // GL_TEXTURE_MAG_FILTER，没有足够的纹素来映射片元时的处理方式
@property(nonatomic, assign) int wrapS; // GL_TEXTURE_WRAP_S，超出范围的纹理处理方式，ST 坐标 S
@property(nonatomic, assign) int wrapT; // GL_TEXTURE_WRAP_T，超出范围的纹理处理方式，ST 坐标 T
@property(nonatomic, assign) int internalFormat;
@property(nonatomic, assign) int format;
@property(nonatomic, assign) int type;

@end

NS_ASSUME_NONNULL_END
```


`KFGLTextureAttributes.m`


```objc
#import "KFGLTextureAttributes.h"
#import <OpenGLES/EAGL.h>
#import <OpenGLES/ES2/gl.h>

@implementation KFGLTextureAttributes

- (instancetype)init {
    self = [super init];
    if (self) {
        _minFilter = GL_LINEAR; // 混合附近纹素的颜色来计算片元的颜色。
        _magFilter = GL_LINEAR; // 混合附近纹素的颜色来计算片元的颜色。
        _wrapS = GL_CLAMP_TO_EDGE; // 采样纹理边缘，即剩余部分显示纹理临近的边缘颜色值。
        _wrapT = GL_CLAMP_TO_EDGE; // 采样纹理边缘，即剩余部分显示纹理临近的边缘颜色值。
        _internalFormat = GL_RGBA;
        _format = GL_RGBA;
        _type = GL_UNSIGNED_BYTE;
    }
    
    return self;
}

@end
```


`KFGLBase.h`

```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

#define STRINGIZE(x) #x
#define STRINGIZE2(x) STRINGIZE(x)
#define SHADER_STRING(text) @ STRINGIZE2(text)

extern NSString *const KFDefaultVertexShader;
extern NSString *const KFDefaultFragmentShader;

NS_ASSUME_NONNULL_END
```


`KFGLBase.m`

```objc
#import "KFGLBase.h"

NSString *const KFDefaultVertexShader = SHADER_STRING
(
    attribute vec4 position; // 通过 attribute 通道获取顶点信息。4 维向量。
    attribute vec4 inputTextureCoordinate; // 通过 attribute 通道获取纹理坐标信息。4 维向量。
 
    varying vec2 textureCoordinate; // 用于 vertex shader 和 fragment shader 间传递纹理坐标。2 维向量。
 
    uniform mat4 mvpMatrix; // 通过 uniform 通道获取 mvp 矩阵信息。4x4 矩阵。
 
    void main()
    {
        gl_Position = mvpMatrix * position; // 根据 mvp 矩阵和顶点信息计算渲染管线最终要用的顶点信息。
        textureCoordinate = inputTextureCoordinate.xy; // 将通过 attribute 通道获取的纹理坐标数据中的 2 维分量传给 fragment shader。
    }
);

NSString *const KFDefaultFragmentShader = SHADER_STRING
(
    varying highp vec2 textureCoordinate; // 从 vertex shader 传递来的纹理坐标。
    uniform sampler2D inputImageTexture; // 通过 uniform 通道获取纹理信息。2D 纹理。
 
    void main()
    {
        gl_FragColor = texture2D(inputImageTexture, textureCoordinate); // texture2D 获取指定纹理在对应坐标位置的 rgba 颜色值，作为渲染管线最终要用的颜色信息。
    }
);
```


`KFPixelBufferConvertTexture.h`


```objc
#import <Foundation/Foundation.h>
#import <OpenGLES/EAGL.h>
#import <CoreVideo/CoreVideo.h>
#import "KFTextureFrame.h"

NS_ASSUME_NONNULL_BEGIN

// KFPixelBufferConvertTexture 是一个将 CVPixelBuffer 转换为纹理 Texture 的工具类，兼容颜色空间的转换处理
@interface KFPixelBufferConvertTexture : NSObject

- (instancetype)initWithContext:(EAGLContext *)context;
- (KFTextureFrame *)renderFrame:(CVPixelBufferRef)pixelBuffer time:(CMTime)time; // 将 CVPixelBuffer 转换为纹理 Texture

@end

NS_ASSUME_NONNULL_END
```


`KFPixelBufferConvertTexture.mm`


```objc
#import "KFPixelBufferConvertTexture.h"
#import <OpenGLES/gltypes.h>
#import "KFGLFilter.h"
#import <UIKit/UIKit.h>
#import <Foundation/Foundation.h>
#import <CoreMedia/CoreMedia.h>
#import "KFGLBase.h"

static const GLfloat kFTColorConversion601VideoRange[] = {
    1.164,  1.164, 1.164,
    0.0, -0.392, 2.017,
    1.596, -0.813,   0.0,
};

static const GLfloat kFTColorConversion601FullRange[] = {
    1.0,    1.0,    1.0,
    0.0,    -0.343, 1.765,
    1.4,    -0.711, 0.0,
};

static const GLfloat kFTColorConversion709VideoRange[] = {
    1.164,  1.164, 1.164,
    0.0, -0.213, 2.112,
    1.793, -0.533,   0.0,
};

static const GLfloat kFTColorConversion709FullRange[] = {
    1.0,    1.0,    1.0,
    0.0,    -0.187, 1.856,
    1.575,    -0.468, 0.0,
};

NSString *const kFYUV2RGBShader = SHADER_STRING
(
    varying highp vec2 textureCoordinate;
 
    uniform sampler2D inputImageTexture;
    uniform sampler2D chrominanceTexture;
    uniform mediump mat3 colorConversionMatrix;
    uniform mediump int isFullRange;
 
    void main()
    {
        mediump vec3 yuv;
        lowp vec3 rgb;
     
        if (isFullRange == 1) {
            yuv.x = texture2D(inputImageTexture, textureCoordinate).r;
        } else {
            yuv.x = texture2D(inputImageTexture, textureCoordinate).r -(16.0 / 255.0);
        }
        yuv.yz = texture2D(chrominanceTexture, textureCoordinate).ra - vec2(0.5, 0.5);
        rgb = colorConversionMatrix * yuv;
     
        gl_FragColor = vec4(rgb, 1);
    }
);

@interface KFPixelBufferConvertTexture () {
    KFGLFilter *_filter;
    GLuint _chrominanceTexture;
    BOOL _isFullRange;
    const GLfloat *_yuvColorMatrix;
    CVOpenGLESTextureCacheRef _textureCache;
}

@end

@implementation KFPixelBufferConvertTexture

- (instancetype)initWithContext:(EAGLContext *)context {
    self = [super init];
    if (self) {
        CVOpenGLESTextureCacheCreate(kCFAllocatorDefault, NULL, context, NULL, &_textureCache);
    }
    return self;
}

- (void)dealloc {
    if (_textureCache) {
        CVOpenGLESTextureCacheFlush(_textureCache, 0);
        CFRelease(_textureCache);
        _textureCache = NULL;
    }
    
    _filter = nil;
}

- (KFTextureFrame *)renderFrame:(CVPixelBufferRef)pixelBuffer time:(CMTime)time {
    if (!pixelBuffer) {
        return nil;
    }
    
    if (CVPixelBufferGetPlaneCount(pixelBuffer) > 0) {
        return [self _yuvRenderFrame:pixelBuffer time:time];
    }
    
    return nil;
}
    
- (void)_setupYUVProgramMatrix:(BOOL)isFullRange colorSpace:(CFTypeRef)colorSpace {
    if (colorSpace == kCVImageBufferYCbCrMatrix_ITU_R_601_4) {
        _yuvColorMatrix = isFullRange ? kFTColorConversion601FullRange : kFTColorConversion601VideoRange;
    } else {
        _yuvColorMatrix = isFullRange ? kFTColorConversion709FullRange : kFTColorConversion709VideoRange;
    }
    _isFullRange = isFullRange;
    
    if (!_filter) {
        _filter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFDefaultVertexShader fragmentShader:kFYUV2RGBShader];
        __weak typeof(self) _self = self;
        _filter.preDrawCallBack = ^() {
            __strong typeof(_self) sself = _self;
            if (!sself) {
                return;
            }
            glActiveTexture(GL_TEXTURE5);
            glBindTexture(GL_TEXTURE_2D, sself->_chrominanceTexture);
            glUniform1i([sself->_filter.getProgram getUniformLocation:@"chrominanceTexture"], 5);
            
            glUniformMatrix3fv([sself->_filter.getProgram getUniformLocation:@"colorConversionMatrix"], 1, GL_FALSE, sself->_yuvColorMatrix);
            glUniform1i([sself->_filter.getProgram getUniformLocation:@"isFullRange"], sself->_isFullRange ? 1 : 0);
        };
    }
}

- (BOOL)_pixelBufferIsFullRange:(CVPixelBufferRef)pixelBuffer {
    // 判断 YUV 数据是否为 full range。
    if (@available(iOS 15, *)) {
        CFDictionaryRef cfDicAttributes = CVPixelBufferCopyCreationAttributes(pixelBuffer);
        NSDictionary *dicAttributes = (__bridge_transfer NSDictionary*)cfDicAttributes;
        if (dicAttributes && [dicAttributes objectForKey:@"PixelFormatDescription"]) {
            NSDictionary *pixelFormatDescription = [dicAttributes objectForKey:@"PixelFormatDescription"];
            if (pixelFormatDescription && [pixelFormatDescription objectForKey:(__bridge NSString*)kCVPixelFormatComponentRange]) {
                NSString *componentRange = [pixelFormatDescription objectForKey:(__bridge NSString *)kCVPixelFormatComponentRange];
                return [componentRange isEqualToString:(__bridge NSString *)kCVPixelFormatComponentRange_FullRange];
            }
        }
    } else {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        OSType formatType = CVPixelBufferGetPixelFormatType(pixelBuffer);
        return formatType == kCVPixelFormatType_420YpCbCr8BiPlanarFullRange;
#pragma clang diagnostic pop
    }
    
    return NO;
}

- (KFTextureFrame *)_yuvRenderFrame:(CVPixelBufferRef)pixelBuffer time:(CMTime)time{
    BOOL isFullYUVRange = [self _pixelBufferIsFullRange:pixelBuffer];
    CFTypeRef matrixKey = kCVImageBufferYCbCrMatrix_ITU_R_601_4;
    if (@available(iOS 15, *)) {
        matrixKey = CVBufferCopyAttachment(pixelBuffer, kCVImageBufferYCbCrMatrixKey, NULL);
    }else{
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        matrixKey = CVBufferGetAttachment(pixelBuffer, kCVImageBufferYCbCrMatrixKey, NULL);
#pragma clang diagnostic pop
    }
    
    [self _setupYUVProgramMatrix:isFullYUVRange colorSpace:matrixKey];
    
    CVOpenGLESTextureRef luminanceTextureRef = NULL;
    CVOpenGLESTextureRef chrominanceTextureRef = NULL;
    
    CVPixelBufferLockBaseAddress(pixelBuffer, 0);
    
    CVReturn err;
    glActiveTexture(GL_TEXTURE4);
    
    size_t width = CVPixelBufferGetWidthOfPlane(pixelBuffer, 0);
    size_t height = CVPixelBufferGetHeightOfPlane(pixelBuffer, 0);
    err = CVOpenGLESTextureCacheCreateTextureFromImage(kCFAllocatorDefault, _textureCache, pixelBuffer, NULL, GL_TEXTURE_2D, GL_LUMINANCE, (GLsizei)width, (GLsizei)height, GL_LUMINANCE, GL_UNSIGNED_BYTE, 0, &luminanceTextureRef);
    if (err){
        NSLog(@"KFPixelBufferConvertTexture CVOpenGLESTextureCacheCreateTextureFromImage error");
        return nil;
    }
    
    GLuint luminanceTexture = CVOpenGLESTextureGetName(luminanceTextureRef);
    glBindTexture(GL_TEXTURE_2D, luminanceTexture);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    
    // UV-plane
    glActiveTexture(GL_TEXTURE5);
    size_t width_uv = CVPixelBufferGetWidthOfPlane(pixelBuffer, 1);
    size_t height_uv = CVPixelBufferGetHeightOfPlane(pixelBuffer, 1);
    err = CVOpenGLESTextureCacheCreateTextureFromImage(kCFAllocatorDefault, _textureCache, pixelBuffer, NULL, GL_TEXTURE_2D, GL_LUMINANCE_ALPHA, (GLsizei)width_uv, (GLsizei)height_uv, GL_LUMINANCE_ALPHA, GL_UNSIGNED_BYTE, 1, &chrominanceTextureRef);
    if (err){
        NSLog(@"KFPixelBufferConvertTexture CVOpenGLESTextureCacheCreateTextureFromImage error");
        return nil;
    }
    
    _chrominanceTexture = CVOpenGLESTextureGetName(chrominanceTextureRef);
    glBindTexture(GL_TEXTURE_2D, _chrominanceTexture);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    
    KFTextureFrame *inputFrame = [[KFTextureFrame alloc] initWithTextureId:luminanceTexture textureSize:CGSizeMake(width, height) time:time];
    KFTextureFrame *resultFrame = [_filter render:inputFrame];
    
    CVPixelBufferUnlockBaseAddress(pixelBuffer, 0);
    if(luminanceTextureRef) CFRelease(luminanceTextureRef);
    if(chrominanceTextureRef) CFRelease(chrominanceTextureRef);
    
    return resultFrame;
}
    
@end
```



### 1.3、串联采集和渲染

最后就是在 ViewController 里将采集模块和渲染模块串起来了。代码如下：


`KFVideoRenderViewController.m`


```objc
#import "KFVideoRenderViewController.h"
#import "KFVideoCapture.h"
#import "KFPixelBufferConvertTexture.h"
#import "KFOpenGLView.h"

@interface KFVideoRenderViewController ()
@property (nonatomic, strong) KFVideoCaptureConfig *videoCaptureConfig;
@property (nonatomic, strong) KFVideoCapture *videoCapture;
@property (nonatomic, strong) KFOpenGLView *glView;
@property (nonatomic, strong) KFPixelBufferConvertTexture *pixelBufferConvertTexture;
@property (nonatomic, strong) EAGLContext *context;
@end

@implementation KFVideoRenderViewController
#pragma mark - Property
- (KFVideoCaptureConfig *)videoCaptureConfig {
    if (!_videoCaptureConfig) {
        _videoCaptureConfig = [[KFVideoCaptureConfig alloc] init];
    }
    
    return _videoCaptureConfig;
}

- (EAGLContext *)context {
    if (!_context) {
        _context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    }
    
    return _context;
}

- (KFPixelBufferConvertTexture *)pixelBufferConvertTexture {
    if (!_pixelBufferConvertTexture) {
        _pixelBufferConvertTexture = [[KFPixelBufferConvertTexture alloc] initWithContext:self.context];
    }
    
    return _pixelBufferConvertTexture;
}

- (KFVideoCapture *)videoCapture {
    if (!_videoCapture) {
        _videoCapture = [[KFVideoCapture alloc] initWithConfig:self.videoCaptureConfig];
        __weak typeof(self) weakSelf = self;
        _videoCapture.sampleBufferOutputCallBack = ^(CMSampleBufferRef sampleBuffer) {
             // 视频采集数据回调。将采集回来的数据给渲染模块渲染。
            [EAGLContext setCurrentContext:weakSelf.context];
            KFTextureFrame *textureFrame = [weakSelf.pixelBufferConvertTexture renderFrame:CMSampleBufferGetImageBuffer(sampleBuffer) time:CMSampleBufferGetPresentationTimeStamp(sampleBuffer)];
            [weakSelf.glView displayFrame:textureFrame];
            [EAGLContext setCurrentContext:nil];
        };
        _videoCapture.sessionErrorCallBack = ^(NSError* error) {
            NSLog(@"KFVideoCapture Error:%zi %@", error.code, error.localizedDescription);
        };
    }
    
    return _videoCapture;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    [self requestAccessForVideo];
    [self setupUI];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.glView.frame = self.view.bounds;
}

#pragma mark - Action
- (void)changeCamera {
    [self.videoCapture changeDevicePosition:self.videoCapture.config.position == AVCaptureDevicePositionBack ? AVCaptureDevicePositionFront : AVCaptureDevicePositionBack];
}

#pragma mark - Private Method
- (void)requestAccessForVideo {
    __weak typeof(self) weakSelf = self;
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    switch (status) {
        case AVAuthorizationStatusNotDetermined:{
            // 许可对话没有出现，发起授权许可。
            [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {
                if (granted) {
                    [weakSelf.videoCapture startRunning];
                } else {
                    // 用户拒绝。
                }
            }];
            break;
        }
        case AVAuthorizationStatusAuthorized:{
            // 已经开启授权，可继续。
            [weakSelf.videoCapture startRunning];
            break;
        }
        default:
            break;
    }
}

- (void)setupUI {
    self.edgesForExtendedLayout = UIRectEdgeAll;
    self.extendedLayoutIncludesOpaqueBars = YES;
    self.title = @"Video Render";
    self.view.backgroundColor = [UIColor whiteColor];
    
    // Navigation item.
    UIBarButtonItem *cameraBarButton = [[UIBarButtonItem alloc] initWithTitle:@"Camera" style:UIBarButtonItemStylePlain target:self action:@selector(changeCamera)];
    self.navigationItem.rightBarButtonItems = @[cameraBarButton];
    
    // 渲染 view。
    _glView = [[KFOpenGLView alloc] initWithFrame:self.view.bounds context:self.context];
    _glView.fillMode = KFGLViewContentModeFill;
    [self.view addSubview:self.glView];
}

@end
```







## 2、Android Demo

### 2.1、视频采集模块

**1）配置类**

定义一个 `KFVideoCaptureConfig` 用来配置视频采集参数。实际应用的采集分辨率与相机硬件有关，一般会根据配置的分辨率查找对应最合适的分辨率。


```java
public class KFVideoCaptureConfig {
    ///< 摄像头方向
    public Integer cameraFacing = CameraCharacteristics.LENS_FACING_FRONT;
    ///< 分辨率
    public Size resolution = new Size(1080, 1920);
    ///< 帧率
    public Integer fps = 30;
}
```


**2）视频采集接口和实现**

视频采集接口包含以下方法：

- 初始化
- 开始采集
- 停止采集
- 切换摄像头
- 释放实例
- 采集状态查询
- 获取 gl 上下文



```java
public interface KFIVideoCapture {
    ///< 视频采集初始化
    public void setup(Context context, KFVideoCaptureConfig config, KFVideoCaptureListener listener, EGLContext eglShareContext);
    ///< 释放采集实例
    public void release();
    ///< 开始采集
    public void startRunning();
    ///< 关闭采集
    public void stopRunning();
    ///< 是否正在采集
    public boolean isRunning();
    ///< 获取 OpenGL 上下文
    public EGLContext getEGLContext();
    ///< 切换摄像头
    public void switchCamera();
}
```


安卓提供了两套应用级相机框架：`camera1`、`camera2`。两者区别如下：

- `camera1`，最初的 camera 框架，通过 `android.hardware.Camera` 类提供功能接口。无安卓版本限制。
- `camera2`，Android 5.0 引入的 api，通过 `android.hardware.camera2` 包提供功能接口。更新 camera2 的原因是 camera1 过于简单，没法满足更加复杂的相机应用场景，为了提供应用层更多控制相机的权限，才推出 camera2。安卓版本限制：`requireApi >= 21`。


**2.1）KFVideoCaptureV1**

`KFVideoCaptureV1`：使用 camera1 的 Demo 采集实现类。实现接口 KFIVideoCapture。

包括以下模块：

- 初始化：setup。读取相机配置；创建采集线程，在采集线程发送相机指令；创建渲染线程和 GLContext，渲染线程刷新纹理。
- 开始采集：startRunning。初始化相机实例，配置采集参数；设置 SurfaceTexture 给 Camera: `mCamera.setPreviewTexture(mSurfaceTexture.getSurfaceTexture())`；开启相机预览。
- 回调采集数据：mSurfaceTextureListener。SurfaceTexture 接受 camera 采集纹理回调，在渲染线程拼装纹理数据返回给外层。


```java
public class KFVideoCaptureV1 implements KFIVideoCapture {
    public static final int KFVideoCaptureV1CameraDisableError = -3000;
    private static final String TAG = "KFVideoCaptureV1";

    private KFVideoCaptureListener mListener = null; ///< 回调
    private KFVideoCaptureConfig mConfig = null; ///< 配置
    private WeakReference<Context> mContext = null;
    private boolean mCameraIsRunning = false; ///< 是否正在采集

    private HandlerThread mCameraThread = null; ///< 采集线程
    private Handler mCameraHandler = null;

    private KFGLContext mGLContext = null; ///< GL 特效上下文
    private KFSurfaceTexture mSurfaceTexture = null; ///< Surface 纹理
    private KFGLFilter mOESConvert2DFilter; ///< 特效
    private HandlerThread mRenderThread = null; ///< 渲染线程
    private Handler mRenderHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程

    private Camera.CameraInfo mFrontCameraInfo = null; ///< 前置摄像头信息
    private int mFrontCameraId = -1;
    private Camera.CameraInfo mBackCameraInfo = null; ///< 后置摄像头信息
    private int mBackCameraId = -1;
    private Camera mCamera = null; ///< 当前摄像头实例（前置或者后置）

    public KFVideoCaptureV1() {

    }

    @Override
    public void setup(Context context, KFVideoCaptureConfig config, KFVideoCaptureListener listener, EGLContext eglShareContext) {
        mListener = listener;
        mConfig = config;
        mContext = new WeakReference<Context>(context);

        ///< 采集线程
        mCameraThread = new HandlerThread("KFCameraThread");
        mCameraThread.start();
        mCameraHandler = new Handler((mCameraThread.getLooper()));

        ///< 渲染线程
        mRenderThread = new HandlerThread("KFCameraRenderThread");
        mRenderThread.start();
        mRenderHandler = new Handler((mRenderThread.getLooper()));

        ///< OpenGL 上下文
        mGLContext = new KFGLContext(eglShareContext);
    }

    @Override
    public EGLContext getEGLContext() {
        return mGLContext.getContext();
    }

    @Override
    public boolean isRunning() {
        return mCameraIsRunning;
    }

    @Override
    public void release() {
        mCameraHandler.post(() -> {
            ///< 停止视频采集 清晰视频采集实例、OpenGL 上下文、线程等
            _stopRunning();
            mGLContext.bind();
            if(mSurfaceTexture != null){
                mSurfaceTexture.release();
                mSurfaceTexture = null;
            }
            if(mOESConvert2DFilter != null){
                mOESConvert2DFilter.release();
                mOESConvert2DFilter = null;
            }

            mGLContext.unbind();
            mGLContext.release();
            mGLContext = null;

            if(mCamera != null){
                mCamera.release();
                mCamera = null;
            }

            mCameraThread.quit();
            mRenderThread.quit();
        });
    }

    @Override
    public void startRunning() {
        mCameraHandler.post(() -> {
            ///< 检测视频采集权限
            if (ActivityCompat.checkSelfPermission(mContext.get(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                ActivityCompat.requestPermissions((Activity) mContext.get(), new String[] {Manifest.permission.CAMERA}, 1);
            }
            ///< 检测相机是否可用
            if(!_checkCameraService()){
                _callBackError(KFVideoCaptureV1CameraDisableError,"相机不可用");
                return;
            }

            ///< 开启视频采集
            _startRunning();
        });
    }

    @Override
    public void stopRunning() {
        mCameraHandler.post(() -> {
            _stopRunning();
        });
    }

    @Override
    public void switchCamera() {
        mCameraHandler.post(() -> {
            ///< 切换摄像头，先关闭相机调整方向再打开相机
            _stopRunning();
            mConfig.cameraFacing = mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT ? CameraCharacteristics.LENS_FACING_BACK : CameraCharacteristics.LENS_FACING_FRONT;
            _startRunning();
        });
    }

    private void _startRunning() {
        ///< 获取前后台摄像机信息
        if(mFrontCameraInfo == null || mBackCameraInfo == null){
            _initCameraInfo();
        }

        try {
            ///< 根据前后台摄像头 id 打开相机实例
            mCamera = Camera.open(_getCurrentCameraId());
            if(mCamera != null){
                ///< 设置相机各分辨率、帧率、方向
                Camera.Parameters parameters = mCamera.getParameters();
                Size previewSize = _getOptimalSize(mConfig.resolution.getWidth(), mConfig.resolution.getHeight());
                mConfig.resolution = new Size(previewSize.getHeight(),previewSize.getWidth());
                parameters.setPreviewSize(previewSize.getWidth(),previewSize.getHeight());
                Range<Integer> selectFpsRange = _chooseFpsRange();
                if(selectFpsRange.getUpper() > 0) {
                    parameters.setPreviewFpsRange(selectFpsRange.getLower(),selectFpsRange.getUpper());
                }
                mCamera.setParameters(parameters);
                mCamera.setDisplayOrientation(_getDisplayOrientation());
                ///< 创建 Surface 纹理
                if(mSurfaceTexture == null){
                    mGLContext.bind();
                    mSurfaceTexture = new KFSurfaceTexture(mSurfaceTextureListener);
                    mOESConvert2DFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,KFGLBase.oesFragmentShader);
                    mGLContext.unbind();
                }
                ///< 设置 SurfaceTexture 给 Camera，这样 Camera 自动将数据渲染到 SurfaceTexture
                mCamera.setPreviewTexture(mSurfaceTexture.getSurfaceTexture());
                ///< 开启预览
                mCamera.startPreview();
                mCameraIsRunning = true;
                if(mListener != null){
                    mMainHandler.post(()->{
                        ///< 回调相机打开
                        mListener.cameraOnOpened();
                    });
                }
            }
        } catch (RuntimeException | IOException e) {
            e.printStackTrace();
        }
    }

    private void _stopRunning() {
        if(mCamera != null){
            ///< 关闭相机采集
            mCamera.setPreviewCallback(null);
            mCamera.stopPreview();
            mCamera.release();
            mCamera = null;
            mCameraIsRunning = false;
            if(mListener != null){
                mMainHandler.post(()->{
                    ///< 回调相机关闭
                    mListener.cameraOnClosed();
                });
            }
        }
    }

    private int _getCurrentCameraId() {
        ///< 获取当前摄像机 id
        if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT) {
            return mFrontCameraId;
        } else if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_BACK) {
            return mBackCameraId;
        } else {
            throw new RuntimeException("No available camera id found.");
        }
    }

    private int _getDisplayOrientation() {
        ///< 获取摄像机需要旋转的方向
        int orientation = 0;
        if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT) {
            orientation = (_getCurrentCameraInfo().orientation) % 360;
            orientation = (360 - orientation) % 360;
        } else {
            orientation = (_getCurrentCameraInfo().orientation + 360) % 360;
        }
        return orientation;
    }

    private Camera.CameraInfo _getCurrentCameraInfo() {
        ///< 获取当前摄像机描述信息
        if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT) {
            return mFrontCameraInfo;
        } else if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_BACK) {
            return mBackCameraInfo;
        } else {
            throw new RuntimeException("No available camera id found.");
        }
    }

    private Size _getOptimalSize(int width, int height) {
        ///< 根据外层输入分辨率查找对应最合适的分辨率
        List<Camera.Size> sizeMap = mCamera.getParameters().getSupportedPreviewSizes();
        List<Size> sizeList = new ArrayList<>();
        for (Camera.Size option:sizeMap) {
            if (width > height) {
                if (option.width >= width && option.height >= height) {
                    sizeList.add(new Size(option.width,option.height));
                }
            } else {
                if (option.width >= height && option.height >= width) {
                    sizeList.add(new Size(option.width,option.height));
                }
            }
        }
        if (sizeList.size() > 0) {
            return Collections.min(sizeList, new Comparator<Size>() {
                @Override
                public int compare(Size o1, Size o2) {
                    return Long.signum(o1.getWidth() * o1.getHeight() - o2.getWidth() * o2.getHeight());
                }
            });
        }

        return new Size(0,0);
    }

    private Range<Integer> _chooseFpsRange() {
        ///< 根据外层设置帧率查找最合适的帧率
        List<int[]> fpsRange = mCamera.getParameters().getSupportedPreviewFpsRange();
        for(int[] range : fpsRange){
            if(range.length == 2 && range[1] >= mConfig.fps*1000 && range[0] <= mConfig.fps*1000){
                // return new Range<>(range[0],mConfig.fps*1000);
                return new Range<>(range[0],range[1]); ///< 仅支持列表中一项，不能像 camera2 一样指定
            }
        }

        return new Range<Integer>(0,0);
    }

    private void _initCameraInfo() {
        ///< 获取前置后置摄像头描述信息与 id
        int numberOfCameras = Camera.getNumberOfCameras();
        for (int cameraId = 0; cameraId < numberOfCameras; cameraId++) {
            Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
            Camera.getCameraInfo(cameraId, cameraInfo);
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                // 后置摄像头信息
                mBackCameraId = cameraId;
                mBackCameraInfo = cameraInfo;
            } else if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
                // 前置摄像头信息
                mFrontCameraId = cameraId;
                mFrontCameraInfo = cameraInfo;
            }
        }
    }

    private boolean _checkCameraService(){
        ///< 检测相机是否可用
        DevicePolicyManager dpm = (DevicePolicyManager)mContext.get().getSystemService(Context.DEVICE_POLICY_SERVICE);
        if (dpm.getCameraDisabled(null)) {
            return false;
        }
        return true;
    }

    private void _callBackError(int error, String errorMsg){
        ///< 错误回调
        if(mListener != null){
            mMainHandler.post(()->{
                mListener.cameraOnError(error,TAG + errorMsg);
            });
        }
    }

    private KFSurfaceTextureListener mSurfaceTextureListener = new KFSurfaceTextureListener() {
        @Override
        ///< SurfaceTexture 数据回调
        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
            mRenderHandler.post(()->{
                long timestamp = System.nanoTime();
                mGLContext.bind();
                ///< 刷新纹理数据至 SurfaceTexture
                mSurfaceTexture.getSurfaceTexture().updateTexImage();
                if(mListener != null){
                    ///< 拼装好纹理数据返回给外层
                    KFTextureFrame frame = new KFTextureFrame(mSurfaceTexture.getSurfaceTextureId(),mConfig.resolution,timestamp,true);
                    mSurfaceTexture.getSurfaceTexture().getTransformMatrix(frame.textureMatrix);
                    KFFrame convertFrame = mOESConvert2DFilter.render(frame);
                    mListener.onFrameAvailable(convertFrame);
                }
                mGLContext.unbind();
            });
        }
    };
}

```
   
**3.2）KFVideoCaptureV1**

`KFVideoCaptureV2`：使用 camera2 的 demo 采集实现类。

实现接口 KFIVideoCapture。模块设计和 KFVideoCaptureV1 基本一致，只是 camera api 调用有差异，不再赘述。

```java
public class KFVideoCaptureV2 implements KFIVideoCapture {
    public static final int KFVideoCaptureV2CameraDisableError = -3000;
    private static final String TAG = "KFVideoCaptureV2";
    private KFVideoCaptureListener mListener = null; ///< 回调
    private KFVideoCaptureConfig mConfig = null; ///< 采集配置
    private WeakReference<Context> mContext = null;

    private CameraManager mCameraManager = null; ///< 相机系统服务，用于管理和连接相机设备
    private String mCameraId; ///<摄像头id
    private CameraDevice mCameraDevice = null; ///< 相机设备类
    private HandlerThread mCameraThread = null; ///< 采集线程
    private Handler mCameraHandler = null;
    private CaptureRequest.Builder mCaptureRequestBuilder = null; ///< CaptureRequest 的构造器，使用 Builder 模式，设置更加方便
    private CaptureRequest mCaptureRequest = null; ///< 相机捕获图像的设置请求，包含传感器、镜头、闪光灯等
    private CameraCaptureSession mCameraCaptureSession = null; ///< 请求抓取相机图像帧的会话，会话的建立主要会建立起一个通道，源端是相机，另一端是 Target
    private boolean mCameraIsRunning = false;
    private Range<Integer>[] mFpsRange;

    private KFGLContext mGLContext = null;
    private KFSurfaceTexture mSurfaceTexture = null;
    private KFGLFilter mOESConvert2DFilter; ///< 特效
    private Surface mSurface = null;
    private HandlerThread mRenderThread = null;
    private Handler mRenderHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper());

    public KFVideoCaptureV2() {

    }

    @Override
    public void setup(Context context, KFVideoCaptureConfig config, KFVideoCaptureListener listener, EGLContext eglShareContext) {
        mListener = listener;
        mConfig = config;
        mContext = new WeakReference<Context>(context);

        ///< 相机采集线程
        mCameraThread = new HandlerThread("KFCameraThread");
        mCameraThread.start();
        mCameraHandler = new Handler((mCameraThread.getLooper()));

        ///< 渲染线程
        mRenderThread = new HandlerThread("KFCameraRenderThread");
        mRenderThread.start();
        mRenderHandler = new Handler((mRenderThread.getLooper()));

        mGLContext = new KFGLContext(eglShareContext);
    }

    @Override
    public EGLContext getEGLContext() {
        return mGLContext.getContext();
    }

    @Override
    public boolean isRunning() {
        return mCameraIsRunning;
    }

    @Override
    public void startRunning() {
        ///< 开启预览
        mCameraHandler.post(() -> {
            _startRunning();
        });
    }

    @Override
    public void stopRunning() {
        ///< 停止预览
        mCameraHandler.post(() -> {
            _stopRunning();
        });
    }

    @Override
    public void release() {
        mCameraHandler.post(() -> {
            ///< 关闭采集、释放 SurfaceTexture、OpenGL 上下文、线程等
            _stopRunning();
            mGLContext.bind();
            if(mSurfaceTexture != null){
                mSurfaceTexture.release();
                mSurfaceTexture = null;
            }

            if(mOESConvert2DFilter != null){
                mOESConvert2DFilter.release();
                mOESConvert2DFilter = null;
            }

            mGLContext.unbind();
            mGLContext.release();
            mGLContext = null;

            if(mSurface != null){
                mSurface.release();
                mSurface = null;
            }

            mCameraThread.quit();
            mRenderThread.quit();
        });
    }

    @Override
    public void switchCamera() {
        ///< 切换摄像头
        mCameraHandler.post(() -> {
            _stopRunning();
            mConfig.cameraFacing = mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT ? CameraCharacteristics.LENS_FACING_BACK : CameraCharacteristics.LENS_FACING_FRONT;
            _startRunning();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _startRunning() {
        ///< 获取相机系统服务
        if(mCameraManager == null){
            mCameraManager = (CameraManager) mContext.get().getSystemService(Context.CAMERA_SERVICE);
        }
        ///< 根据外层摄像头方向查找摄像头 id
        boolean selectSuccess = _chooseCamera();
        if (selectSuccess) {
            try {
                ///< 检测采集权限
                if (ActivityCompat.checkSelfPermission(mContext.get(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                    ActivityCompat.requestPermissions((Activity) mContext.get(), new String[] {Manifest.permission.CAMERA}, 1);
                }

                ///< 检测相机是否可用
                if(!_checkCameraService()){
                    _callBackError(KFVideoCaptureV2CameraDisableError, "相机不可用");
                    return;
                }

                ///< 打开相机设备
                mCameraManager.openCamera(mCameraId, mStateCallback, mCameraHandler);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }
    }

    private void _stopRunning() {
        ///< 停止采集
        if(mCameraCaptureSession != null) {
            mCameraCaptureSession.close();
            mCameraCaptureSession = null;
        }

        if(mCameraDevice != null){
            mCameraDevice.close();
            mCameraDevice = null;
        }
    }

    private KFSurfaceTextureListener mSurfaceTextureListener = new KFSurfaceTextureListener() {
        @Override
        //< SurfaceTexture 数据回调
        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
            mRenderHandler.post(() -> {
                long timestamp = System.nanoTime();
                mGLContext.bind();
                ///< 刷新纹理数据至 SurfaceTexture
                mSurfaceTexture.getSurfaceTexture().updateTexImage();
                if(mListener != null){
                    ///< 拼装好纹理数据返回给外层
                    KFTextureFrame frame = new KFTextureFrame(mSurfaceTexture.getSurfaceTextureId(),mConfig.resolution,timestamp,true);
                    mSurfaceTexture.getSurfaceTexture().getTransformMatrix(frame.textureMatrix);
                    KFFrame convertFrame = mOESConvert2DFilter.render(frame);
                    mListener.onFrameAvailable(convertFrame);
                }
                mGLContext.unbind();
            });
        }
    };

    private CameraCaptureSession.StateCallback mCaputreSessionCallback = new CameraCaptureSession.StateCallback() {
        @Override
        ///< 创建会话回调
        public void onConfigured(@NonNull CameraCaptureSession cameraCaptureSession) {
            ///< 创建CaptureRequest
            mCaptureRequest = mCaptureRequestBuilder.build();
            mCameraCaptureSession = cameraCaptureSession;
            try {
                ///< 通过连续重复的 Capture 实现预览功能，每次 Capture 会把预览画面显示到对应的 Surface 上
                mCameraCaptureSession.setRepeatingRequest(mCaptureRequest, null, null);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        ///< 创建会话出错回调
        public void onConfigureFailed(@NonNull CameraCaptureSession cameraCaptureSession) {
            _callBackError(1005,"onConfigureFailed");
        }
    };

    private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {
        @Override
        ///< 相机打开回调
        public void onOpened(@NonNull CameraDevice camera) {
            mCameraDevice = camera;
            try {
                ///< 通过相机设备创建构造器
                mCaptureRequestBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
                Range<Integer> selectFpsRange = _chooseFpsRange();
                ///< 设置帧率
                if(selectFpsRange.getUpper() > 0){
                    mCaptureRequestBuilder.set(CaptureRequest.CONTROL_AE_TARGET_FPS_RANGE,selectFpsRange);
                }
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }

            if(mListener != null){
                mMainHandler.post(()->{
                    mListener.cameraOnOpened();
                });
            }
            mCameraIsRunning = true;

            if(mSurfaceTexture == null){
                mGLContext.bind();
                mSurfaceTexture = new KFSurfaceTexture(mSurfaceTextureListener);
                mOESConvert2DFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,KFGLBase.oesFragmentShader);
                mGLContext.unbind();
                mSurface = new Surface(mSurfaceTexture.getSurfaceTexture());
            }

            if(mSurface != null) {
                ///< 设置目标输出 Surface
                mSurfaceTexture.getSurfaceTexture().setDefaultBufferSize(mConfig.resolution.getHeight(),mConfig.resolution.getWidth());
                mCaptureRequestBuilder.addTarget(mSurface);
                try {
                    ///< 创建通道会话
                    mCameraDevice.createCaptureSession(Arrays.asList(mSurface), mCaputreSessionCallback, mCameraHandler);
                } catch (CameraAccessException e) {
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void onDisconnected(@NonNull CameraDevice camera) {
            ///< 相机断开连接回调
            camera.close();
            mCameraDevice = null;
            mCameraIsRunning = false;
        }

        @Override
        public void onClosed(@NonNull CameraDevice camera) {
            ///< 相机关闭回调
            camera.close();
            mCameraDevice = null;
            if(mListener != null){
                mMainHandler.post(()->{
                    mListener.cameraOnClosed();
                });
            }
            mCameraIsRunning = false;
        }

        @Override
        public void onError(@NonNull CameraDevice camera, int error) {
            ///< 相机出错回调
            camera.close();
            mCameraDevice = null;
            _callBackError(error,"Camera onError");
            mCameraIsRunning = false;
        }
    };

    private boolean _chooseCamera() {
        try {
            ///< 根据外层配置方向选择合适的设备 id 与 FPS 区间
            final String[] ids = mCameraManager.getCameraIdList();
            for(String cameraId : ids) {
                CameraCharacteristics characteristics = mCameraManager.getCameraCharacteristics(cameraId);
                Integer facing = characteristics.get(CameraCharacteristics.LENS_FACING);
                if(facing == mConfig.cameraFacing){
                    mCameraId = cameraId;
                    mFpsRange = characteristics.get(CameraCharacteristics.CONTROL_AE_AVAILABLE_TARGET_FPS_RANGES);
                    StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
                    if (map != null) {
                        Size previewSize = _getOptimalSize(map.getOutputSizes(SurfaceTexture.class), mConfig.resolution.getWidth(), mConfig.resolution.getHeight());
                        // Range<Integer>[] fpsRanges = map.getHighSpeedVideoFpsRangesFor(previewSize); ///< high fps range
                        mConfig.resolution = new Size(previewSize.getHeight(),previewSize.getWidth());
                    }
                    return true;
                }
            }
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }

        return false;
    }

    private Size _getOptimalSize(Size[] sizeMap, int width, int height) {
        ///< 根据外层配置分辨率寻找合适的分辨率
        List<Size> sizeList = new ArrayList<>();
        for (Size option : sizeMap) {
            if (width > height) {
                if (option.getWidth() >= width && option.getHeight() >= height) {
                    sizeList.add(option);
                }
            } else {
                if (option.getWidth() >= height && option.getHeight() >= width) {
                    sizeList.add(option);
                }
            }
        }
        if (sizeList.size() > 0) {
            return Collections.min(sizeList, new Comparator<Size>() {
                @Override
                public int compare(Size o1, Size o2) {
                    return Long.signum(o1.getWidth() * o1.getHeight() - o2.getWidth() * o2.getHeight());
                }
            });
        }
        return sizeMap[0];
    }

    private boolean _checkCameraService(){
        ///< 检测相机是否可用
        DevicePolicyManager dpm = (DevicePolicyManager)mContext.get().getSystemService(Context.DEVICE_POLICY_SERVICE);
        if (dpm.getCameraDisabled(null)) {
            return false;
        }
        return true;
    }

    private void _callBackError(int error, String errorMsg){
        ///< 错误回调
        if(mListener != null){
            mMainHandler.post(()->{
                mListener.cameraOnError(error,TAG + errorMsg);
            });
        }
    }

    private Range<Integer> _chooseFpsRange() {
        ///< 根据外层配置的帧率寻找合适的帧率
        for(Range<Integer> range : mFpsRange){
            if(range.getUpper() >= mConfig.fps && range.getLower() <= mConfig.fps){
                return new Range<>(range.getLower(),mConfig.fps);
            }
        }

        return new Range<Integer>(0,0);
    }
}
```

### 2.2、视频渲染模块

**1）KFGLContext**

负责创建 OpenGL 环境，实现与 [RenderDemo（1）：用 OpenGL 画一个三角形](https://mp.weixin.qq.com/s/b-nFCBMf-oaayyG8a86mgw) 中一样，此处不再重复介绍。

**2）KFGLFilter**

KFGLFilter 是一个自定义滤镜，外部输入纹理，进行自定义效果渲染。

绘制流程和绘制三角形一致：加载编译 shader，链接到 shader 程序，设置顶点数据，绘制三角形。

Demo 中的 shader 只是最简单的纹理绘制，可以修改 shader 实现相机滤镜、美颜等效果。

```java
public class KFGLFilter {

    private boolean mIsCustomFBO = false; // 是否自定义帧缓存 部分渲染到指定 Surface 等其它场景会自定义
    private KFGLFrameBuffer mFrameBuffer = null; // 帧缓存
    private KFGLProgram mProgram = null; // 着色器容器
    private KFGLTextureAttributes mGLTextureAttributes = null; // 纹理格式描述

    private int mTextureUniform = -1; // 纹理下标
    private int mPostionMatrixUniform = -1; // 顶点矩阵下标
    private int mTextureMatrixUniform = -1; // 纹理矩阵下标
    private int mPositionAttribute = -1; // 顶点下标
    private int mTextureCoordinateAttribute = -1; // 纹理下标
    private FloatBuffer mSquareVerticesBuffer = null; // 顶点 buffer
    private FloatBuffer mTextureCoordinatesBuffer = null; // 纹理 buffer
    private FloatBuffer mCustomSquareVerticesBuffer = null; // 自定义顶点 buffer
    private FloatBuffer mCustomTextureCoordinatesBuffer = null; // 自定义纹理 buffer

    public KFGLFilter(boolean isCustomFBO,String vertexShader,String fragmentShader) {
        mIsCustomFBO = isCustomFBO;
        // 初始化着色器
        _setupProgram(vertexShader,fragmentShader);
    }

    public KFGLFilter(boolean isCustomFBO,String vertexShader,String fragmentShader, KFGLTextureAttributes textureAttributes) {
        mIsCustomFBO = isCustomFBO;
        mGLTextureAttributes = textureAttributes;
        // 初始化着色器
        _setupProgram(vertexShader,fragmentShader);
    }

    public KFGLFrameBuffer getOutputFrameBuffer() {
        return mFrameBuffer;
    }

    public KFFrame render(KFTextureFrame frame){
        if(frame == null){
            return frame;
        }

        KFTextureFrame resultFrame = new KFTextureFrame(frame);
        // 初始化帧缓存
        _setupFrameBuffer(frame.textureSize);

        // 绑定帧缓存
        if(mFrameBuffer != null){
            mFrameBuffer.bind();
        }

        if(mProgram != null){
            // 使用着色器
            mProgram.use();

            // 设置帧缓存背景色
            glClearColor(0,0,0,1);
            // 清空帧缓存颜色
            glClear(GLES20.GL_COLOR_BUFFER_BIT);

            // 激活纹理单元 1
            glActiveTexture(GLES20.GL_TEXTURE1);
            // 根据是否 OES 纹理绑定纹理 id
            if (frame.isOESTexture) {
                glBindTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, frame.textureId);
            } else {
                glBindTexture(GLES20.GL_TEXTURE_2D, frame.textureId);
            }
            // 传递纹理单元 1
            glUniform1i(mTextureUniform, 1);

            // 设置纹理矩阵
            if(mTextureMatrixUniform >= 0){
                glUniformMatrix4fv(mTextureMatrixUniform, 1, false, frame.textureMatrix, 0);
            }

            // 设置顶点矩阵
            if(mPostionMatrixUniform >= 0){
                glUniformMatrix4fv(mPostionMatrixUniform, 1, false, frame.positionMatrix, 0);
            }

            // 启用顶点着色器顶点坐标属性
            glEnableVertexAttribArray(mPositionAttribute);
            // 启用顶点着色器纹理坐标属性
            glEnableVertexAttribArray(mTextureCoordinateAttribute);

            // 根据自定义顶点缓存设置不同顶点坐标
            if(mCustomSquareVerticesBuffer != null){
                mCustomSquareVerticesBuffer.position(0);
                glVertexAttribPointer(mPositionAttribute, 2, GLES20.GL_FLOAT, false, 0, mCustomSquareVerticesBuffer);
            }else{
                mSquareVerticesBuffer.position(0);
                glVertexAttribPointer(mPositionAttribute, 2, GLES20.GL_FLOAT, false, 0, mSquareVerticesBuffer);
            }

            // 根据自定义纹理缓存设置不同纹理坐标
            if(mCustomTextureCoordinatesBuffer != null){
                mCustomTextureCoordinatesBuffer.position(0);
                glVertexAttribPointer(mTextureCoordinateAttribute, 2, GLES20.GL_FLOAT, false, 0, mCustomTextureCoordinatesBuffer);
            }else{
                mTextureCoordinatesBuffer.position(0);
                glVertexAttribPointer(mTextureCoordinateAttribute, 2, GLES20.GL_FLOAT, false, 0, mTextureCoordinatesBuffer);
            }

            // 真正的渲染
            glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, 4);

            // 解除绑定纹理
            if (frame.isOESTexture) {
                glBindTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, 0);
            } else {
                glBindTexture(GLES20.GL_TEXTURE_2D, 0);
            }

            // 关闭顶点着色器顶点属性
            glDisableVertexAttribArray(mPositionAttribute);
            // 关闭顶点着色器纹理属性
            glDisableVertexAttribArray(mTextureCoordinateAttribute);
        }

        // 解绑帧缓存
        if(mFrameBuffer != null){
            mFrameBuffer.unbind();
        }

        // 返回渲染后数据
        if(mFrameBuffer != null){
            resultFrame.textureId = mFrameBuffer.getTextureId();
            resultFrame.textureSize = mFrameBuffer.getSize();
            resultFrame.isOESTexture = false;
            resultFrame.textureMatrix = KFGLBase.KFIdentityMatrix();
            resultFrame.positionMatrix = KFGLBase.KFIdentityMatrix();
        }
        return resultFrame;
    }

    public void release() {
        // 释放帧缓存、着色器
        if(mFrameBuffer != null){
            mFrameBuffer.release();
            mFrameBuffer = null;
        }

        if(mProgram != null){
            mProgram.release();
            mProgram = null;
        }
    }

    public void setSquareVerticesBuffer(FloatBuffer squareVerticesBuffer) {
        mSquareVerticesBuffer = squareVerticesBuffer;
    }

    public void setTextureCoordinatesBuffer(FloatBuffer textureCoordinatesBuffer) {
        mCustomTextureCoordinatesBuffer = textureCoordinatesBuffer;
    }

    public void setIntegerUniformValue(String uniformName, int intValue){
        // 设置 int 类型 uniform 数据
        if(mProgram != null){
            int uniforamIndex = mProgram.getUniformLocation(uniformName);
            mProgram.use();
            glUniform1i(uniforamIndex, intValue);
        }
    }

    public void setFloatUniformValue(String uniformName, float floatValue){
        // 设置 float 类型 uniform 数据
        if(mProgram != null){
            int uniforamIndex = mProgram.getUniformLocation(uniformName);
            mProgram.use();
            glUniform1f(uniforamIndex, floatValue);
        }
    }

    private void _setupFrameBuffer(Size size) {
        if(mIsCustomFBO) {
            return;
        }

        // 初始化帧缓存与对应纹理
        if(mFrameBuffer == null || mFrameBuffer.getSize().getWidth() != size.getWidth() || mFrameBuffer.getSize().getHeight() != size.getHeight()){
            if(mFrameBuffer != null){
                mFrameBuffer.release();
                mFrameBuffer = null;
            }

            mFrameBuffer = new KFGLFrameBuffer(size,mGLTextureAttributes);
        }
    }

    private void _setupProgram(String vertexShader,String fragmentShader){
        // 根据 vs fs 初始化着色器容器
        if(mProgram == null){
            mProgram = new KFGLProgram(vertexShader,fragmentShader);
            mTextureUniform = mProgram.getUniformLocation("inputImageTexture");
            mPostionMatrixUniform = mProgram.getUniformLocation("mvpMatrix");
            mTextureMatrixUniform = mProgram.getUniformLocation("textureMatrix");
            mPositionAttribute = mProgram.getAttribLocation("position");
            mTextureCoordinateAttribute = mProgram.getAttribLocation("inputTextureCoordinate");

            final float squareVertices[] = {
                    -1.0f, -1.0f,
                    1.0f, -1.0f,
                    -1.0f,  1.0f,
                    1.0f,  1.0f,
            };

            ByteBuffer squareVerticesByteBuffer = ByteBuffer.allocateDirect(4 * squareVertices.length);
            squareVerticesByteBuffer.order(ByteOrder.nativeOrder());
            mSquareVerticesBuffer = squareVerticesByteBuffer.asFloatBuffer();
            mSquareVerticesBuffer.put(squareVertices);
            mSquareVerticesBuffer.position(0);

            final float textureCoordinates[] = {
                    0.0f, 0.0f,
                    1.0f, 0.0f,
                    0.0f, 1.0f,
                    1.0f, 1.0f,
            };
            ByteBuffer textureCoordinatesByteBuffer = ByteBuffer.allocateDirect(4 * textureCoordinates.length);
            textureCoordinatesByteBuffer.order(ByteOrder.nativeOrder());
            mTextureCoordinatesBuffer = textureCoordinatesByteBuffer.asFloatBuffer();
            mTextureCoordinatesBuffer.put(textureCoordinates);
            mTextureCoordinatesBuffer.position(0);
        }
    }
}

```

**3）KFRenderView**

KFRenderView 是渲染模块，实现如下：

- 初始化渲染视图：可选 TextureView 或 SurfaceView 作为实际的渲染视图，添加视图到父布局。
- 渲染：在相机采集纹理的回调里，承接外部输入纹理给 KFGLFilter，渲染到 View 的 Surface 上。
- 销毁：释放 GL 上下文，释放渲染时的帧缓存、着色器。


```java
public class KFRenderView extends ViewGroup {
    private KFGLContext mEGLContext = null; // OpenGL上下文
    private KFGLFilter mFilter = null; // 特效渲染到指定 Surface
    private EGLContext mShareContext = null; // 共享上下文
    private View mRenderView = null; // 渲染视图基类
    private int mSurfaceWidth = 0; // 渲染缓存宽
    private int mSurfaceHeight = 0; // 渲染缓存高
    private FloatBuffer mSquareVerticesBuffer = null; // 自定义顶点
    private KFRenderMode mRenderMode = KFRenderMode.KFRenderModeFill; // 自适应模式 黑边 比例填冲
    private boolean mSurfaceChanged = false; // 渲染缓存是否变更
    private Size mLastRenderSize = new Size(0,0); // 标记上次渲染 Size

    public enum KFRenderMode {
        KFRenderStretch,// 拉伸满-可能变形
        KFRenderModeFit,// 黑边
        KFRenderModeFill// 比例填充
    };

    public KFRenderView(Context context, EGLContext eglContext){
        super(context);
        mShareContext = eglContext; // 共享上下文
        _setupSquareVertices(); // 初始化顶点

        boolean isSurfaceView  = false; // TextureView 与 SurfaceView 开关
        if(isSurfaceView){
            mRenderView = new KFSurfaceView(context, mListener);
        }else{
            mRenderView = new KFTextureView(context, mListener);
        }

        this.addView(mRenderView); // 添加视图到父视图
    }

    public void release() {
        // 释放 GL 上下文、特效
        if(mEGLContext != null){
            mEGLContext.bind();
            if(mFilter != null){
                mFilter.release();
                mFilter = null;
            }
            mEGLContext.unbind();

            mEGLContext.release();
            mEGLContext = null;
        }
    }

    public void render(KFTextureFrame inputFrame){
        if(inputFrame == null){
            return;
        }

        //输入纹理使用自定义特效渲染到 View 的 Surface 上
        if(mEGLContext != null && mFilter != null){
            boolean frameResolutionChanged = inputFrame.textureSize.getWidth() != mLastRenderSize.getWidth() || inputFrame.textureSize.getHeight() != mLastRenderSize.getHeight();
            // 渲染缓存变更或者视图大小变更重新设置顶点
            if(mSurfaceChanged || frameResolutionChanged){
                _recalculateVertices(inputFrame.textureSize);
                mSurfaceChanged = false;
                mLastRenderSize = inputFrame.textureSize;
            }

            // 渲染到指定 Surface
            mEGLContext.bind();
            mFilter.setSquareVerticesBuffer(mSquareVerticesBuffer);
            GLES20.glViewport(0, 0, mSurfaceWidth, mSurfaceHeight);
            mFilter.render(inputFrame);
            mEGLContext.swapBuffers();
            mEGLContext.unbind();
        }
    }

    private KFRenderListener mListener = new KFRenderListener() {
        @Override
        // 渲染缓存创建
        public void surfaceCreate(@NonNull Surface surface) {
            mEGLContext = new KFGLContext(mShareContext,surface);
            // 初始化特效
            mEGLContext.bind();
            _setupFilter();
            mEGLContext.unbind();
        }

        @Override
        // 渲染缓存变更
        public void surfaceChanged(@NonNull Surface surface, int width, int height) {
            mSurfaceWidth = width;
            mSurfaceHeight = height;
            mSurfaceChanged = true;
            // 设置 GL 上下文 Surface
            mEGLContext.bind();
            mEGLContext.setSurface(surface);
            mEGLContext.unbind();
        }

        @Override
        public void surfaceDestroy(@NonNull Surface surface) {

        }
    };

    private void _setupFilter() {
        // 初始化特效
        if(mFilter == null){
            mFilter = new KFGLFilter(true, KFGLBase.defaultVertexShader,KFGLBase.defaultFragmentShader);
        }
    }

    private void _setupSquareVertices() {
        // 初始化顶点缓存
        final float squareVertices[] = {
                -1.0f, -1.0f,
                1.0f, -1.0f,
                -1.0f,  1.0f,
                1.0f,  1.0f,
        };

        ByteBuffer squareVerticesByteBuffer = ByteBuffer.allocateDirect(4 * squareVertices.length);
        squareVerticesByteBuffer.order(ByteOrder.nativeOrder());
        mSquareVerticesBuffer = squareVerticesByteBuffer.asFloatBuffer();
        mSquareVerticesBuffer.put(squareVertices);
        mSquareVerticesBuffer.position(0);
    }

    private void _recalculateVertices(Size inputImageSize){
        // 按照适应模式创建顶点
        if(mSurfaceWidth == 0 || mSurfaceHeight == 0){
            return;
        }

        Size renderSize = new Size(mSurfaceWidth,mSurfaceHeight);
        float heightScaling = 1, widthScaling = 1;
        Size insetSize = new Size(0,0);
        float inputAspectRatio = (float) inputImageSize.getWidth() / (float)inputImageSize.getHeight();
        float outputAspectRatio = (float)renderSize.getWidth() / (float)renderSize.getHeight();
        boolean isAutomaticHeight = inputAspectRatio <= outputAspectRatio ? false : true;

        if (isAutomaticHeight) {
            float insetSizeHeight = (float)inputImageSize.getHeight() / ((float)inputImageSize.getWidth() / (float)renderSize.getWidth());
            insetSize = new Size(renderSize.getWidth(),(int)insetSizeHeight);
        } else {
            float insetSizeWidth = (float)inputImageSize.getWidth() / ((float)inputImageSize.getHeight() / (float)renderSize.getHeight());
            insetSize = new Size((int)insetSizeWidth,renderSize.getHeight());
        }

        switch (mRenderMode) {
            case KFRenderStretch: {
                widthScaling = 1;
                heightScaling = 1;
            }; break;
            case KFRenderModeFit: {
                widthScaling = (float)insetSize.getWidth() / (float)renderSize.getWidth();
                heightScaling = (float)insetSize.getHeight() / (float)renderSize.getHeight();
            }; break;
            case KFRenderModeFill: {
                widthScaling = (float) renderSize.getHeight() / (float)insetSize.getHeight();
                heightScaling = (float)renderSize.getWidth() / (float)insetSize.getWidth();
            }; break;
        }

        final float squareVertices[] = {
                -1.0f, -1.0f,
                1.0f, -1.0f,
                -1.0f,  1.0f,
                1.0f,  1.0f,
        };

        final float customVertices[] = {
                -widthScaling, -heightScaling,
                widthScaling, -heightScaling,
                -widthScaling,  heightScaling,
                widthScaling,  heightScaling,
        };
        ByteBuffer squareVerticesByteBuffer = ByteBuffer.allocateDirect(4 * customVertices.length);
        squareVerticesByteBuffer.order(ByteOrder.nativeOrder());
        mSquareVerticesBuffer = squareVerticesByteBuffer.asFloatBuffer();
        mSquareVerticesBuffer.put(customVertices);
        mSquareVerticesBuffer.position(0);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        // 视图变更 Size
        this.mRenderView.layout(left,top,right,bottom);
    }
}
```

### 2.3、串联采集和渲染

MainActivity 中串联采集和渲染模块，实现相机图像实时预览功能。包括如下过程：

- 初始化采集事例：创建 KFIVideoCapture 实例并启动采集。
- 初始化渲染视图：创建 KFRenderView 并添加到 Demo 视图。
- 采集数据回调给渲染：KFIVideoCapture 注册监听 KFVideoCaptureListener，每帧采集触发回调 `onFrameAvailable(frame)`，将回调数据输入 KFRenderView 驱动渲染，实现实时预览。

```java
public class MainActivity extends AppCompatActivity {

    private KFIVideoCapture mCapture;
    private KFVideoCaptureConfig mCaptureConfig;
    private KFRenderView mRenderView;
    private KFGLContext mGLContext;

    private Button cameraButton;
    private Button playerButton;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        playerButton = findViewById(R.id.player_btn);
        cameraButton = findViewById(R.id.camera_btn);
        playerButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

            }
        });


        mGLContext = new KFGLContext(null);
        mRenderView = new KFRenderView(this,mGLContext.getContext());
        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);

        mCaptureConfig = new KFVideoCaptureConfig();
        mCaptureConfig.cameraFacing = LENS_FACING_FRONT;
        mCaptureConfig.resolution = new Size(720,1280);
        mCaptureConfig.fps = 30;
        boolean useCamera2 = false;
        if(useCamera2){
            mCapture = new KFVideoCaptureV2();
        }else{
            mCapture = new KFVideoCaptureV1();
        }
        mCapture.setup(this,mCaptureConfig,mVideoCaptureListener,mGLContext.getContext());
        mCapture.startRunning();
    }

    private KFVideoCaptureListener mVideoCaptureListener = new KFVideoCaptureListener() {
        @Override
        public void cameraOnOpened(){}

        @Override
        public void cameraOnClosed() {
        }

        @Override
        public void cameraOnError(int error,String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void onFrameAvailable(KFFrame frame) {
            mRenderView.render((KFTextureFrame) frame);
        }
    };
}
```

相关代码就介绍到这里。




