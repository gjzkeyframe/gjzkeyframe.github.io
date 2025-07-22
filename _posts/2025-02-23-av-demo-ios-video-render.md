---
title: iOS AVDemo（13）：视频渲染，代码开源并提供解析
description: 介绍 iOS 视频渲染流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 09:18:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, 视频, 视频渲染, 渲染]
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

iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频实用工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目，我们将通过拆解`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并实现 Demo 来向大家介绍如何在 iOS/Android 平台上手音视频开发。

这里是第十三篇：**iOS 视频渲染 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频采集装模块；
- 2）实现一个视频渲染模块；
- 3）串联视频采集和渲染模块，将采集的视频数据输入给渲染模块进行渲染；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。



在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』，这是一个收费的社群服务，目前还有少量优惠券可用。
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_
 
 


## 1、视频采集模块

在这个 Demo 中，视频采集模块 `KFVideoCapture` 的实现与 [《iOS 视频采集 Demo》](https://mp.weixin.qq.com/s/CJAhkk9BmhMOXgD2pl_rjg) 中一样，这里就不再重复介绍了，其接口如下：


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


## 2、视频渲染模块

在之前的[《iOS 视频采集 Demo》](https://mp.weixin.qq.com/s/CJAhkk9BmhMOXgD2pl_rjg)那篇中，我们采集后的视频数据是通过系统封装好的 `AVCaptureVideoPreviewLayer` 来做预览渲染的。这篇我们来介绍一下使用 `MetalKit` 来实现渲染。


首先，我们在 `KFShaderType.h` 中定义一些渲染过程需要用到的数据结构。


`KFShaderType.h`

```objc
#ifndef KFShaderType_h
#define KFShaderType_h

#include <simd/simd.h>

// 存储数据的自定义结构，用于桥接 OC 和 Metal 代码（顶点）。
typedef struct {
    // 顶点坐标，4 维向量。
    vector_float4 position;
    // 纹理坐标。
    vector_float2 textureCoordinate;
} KFVertex;

// 存储数据的自定义结构，用于桥接 OC 和 Metal 代码（顶点）。
typedef struct {
    // YUV 矩阵。
    matrix_float3x3 matrix;
    // 是否为 full range。
    bool fullRange;
} KFConvertMatrix;

// 自定义枚举，用于桥接 OC 和 Metal 代码（顶点）。
// 顶点的桥接枚举值 KFVertexInputIndexVertices。
typedef enum KFVertexInputIndex {
    KFVertexInputIndexVertices = 0,
} KFVertexInputIndex;

// 自定义枚举，用于桥接 OC 和 Metal 代码（片元）。
// YUV 矩阵的桥接枚举值 KFFragmentInputIndexMatrix。
typedef enum KFFragmentBufferIndex {
    KFFragmentInputIndexMatrix = 0,
} KFMetalFragmentBufferIndex;

// 自定义枚举，用于桥接 OC 和 Metal 代码（片元）。
// YUV 数据的桥接枚举值 KFFragmentTextureIndexTextureY、KFFragmentTextureIndexTextureUV。
typedef enum KFFragmentYUVTextureIndex {
    KFFragmentTextureIndexTextureY = 0,
    KFFragmentTextureIndexTextureUV = 1,
} KFFragmentYUVTextureIndex;

// 自定义枚举，用于桥接 OC 和 Metal 代码（片元）。
// RGBA 数据的桥接枚举值 KFFragmentTextureIndexTextureRGB。
typedef enum KFFragmentRGBTextureIndex {
    KFFragmentTextureIndexTextureRGB = 0,
} KFFragmentRGBTextureIndex;

#endif /* KFMetalShaderType_h */
```

然后，我们在 `render.metal` 中写 Metal 渲染代码。它类似 OpenGL 的 shader。


`render.metal`


```objc
#include <metal_stdlib>
#include "KFShaderType.h"

using namespace metal;

// 定义了一个类型为 RasterizerData 的结构体，里面有一个 float4 向量和 float2 向量。
typedef struct {
    // float4：4 维向量；
    // clipSpacePosition：参数名，表示顶点；
    // [[position]]：position 是顶点修饰符，这是苹果内置的语法，不能改变，表示顶点信息。
    float4 clipSpacePosition [[position]];
    // float2：2 维向量；
    // textureCoordinate：参数名，这里表示纹理。
    float2 textureCoordinate;
} RasterizerData;

// 顶点函数通过一个自定义的结构体，返回对应的数据；顶点函数的输入参数也可以是自定义结构体。

// 顶点函数
// vertex：函数修饰符，表示顶点函数；
// RasterizerData：返回值类型；
// vertexShader：函数名；
// [[vertex_id]]：vertex_id 是顶点 id 修饰符，苹果内置的语法不可改变；
// [[buffer(YYImageVertexInputIndexVertexs)]]：buffer 是缓存数据修饰符，苹果内置的语法不可改变，YYImageVertexInputIndexVertexs 是索引；
// constant：是变量类型修饰符，表示存储在 device 区域。
vertex RasterizerData vertexShader(uint vertexID [[vertex_id]],
                                   constant KFVertex *vertexArray [[buffer(KFVertexInputIndexVertices)]]) {
    RasterizerData out;
    out.clipSpacePosition = vertexArray[vertexID].position;
    out.textureCoordinate = vertexArray[vertexID].textureCoordinate;
    
    return out;
}

// 片元函数
// fragment：函数修饰符，表示片元函数；
// float4：返回值类型，返回 RGBA；
// fragmentImageShader：函数名；
// RasterizerData：参数类型；
// input：变量名；
// [[stage_in]：stage_in 表示这个数据来自光栅化，光栅化是顶点处理之后的步骤，业务层无法修改。
// texture2d：类型表示纹理；
// textureY：表示 Y 通道；
// textureUV：表示 UV 通道；
// [[texture(index)]]：纹理修饰符；可以加索引：[[texture(0)]] 对应纹理 0，[[texture(1)]] 对应纹理 1；
// KFFragmentTextureIndexTextureY、KFFragmentTextureIndexTextureUV：表示纹理索引。
fragment float4 yuvSamplingShader(RasterizerData input [[stage_in]],
                                  texture2d<float> textureY [[texture(KFFragmentTextureIndexTextureY)]],
                                  texture2d<float> textureUV [[texture(KFFragmentTextureIndexTextureUV)]],
                                  constant KFConvertMatrix *convertMatrix [[buffer(KFFragmentInputIndexMatrix)]]) {
    constexpr sampler textureSampler (mag_filter::linear, min_filter::linear);
    float3 yuv = float3(textureY.sample(textureSampler, input.textureCoordinate).r, textureUV.sample(textureSampler, input.textureCoordinate).rg);
    
    if (convertMatrix->fullRange) { // full range.
        yuv.x = textureY.sample(textureSampler, input.textureCoordinate).r;
    } else { // video range.
        yuv.x = textureY.sample(textureSampler, input.textureCoordinate).r - (16.0 / 255.0);
    }
    yuv.yz = textureUV.sample(textureSampler, input.textureCoordinate).rg - 0.5;
    
    float3 rgb = convertMatrix->matrix * yuv;
  
    return float4(rgb,1.0);
}

// 片元函数
// fragment：函数修饰符，表示片元函数；
// float4：返回值类型，返回 RGBA；
// fragmentImageShader：函数名；
// RasterizerData：参数类型；
// input：变量名；
// [[stage_in]：stage_in 表示这个数据来自光栅化，光栅化是顶点处理之后的步骤，业务层无法修改。
// texture2d：类型表示纹理；
// colorTexture：代表 RGBA 数据；
// [[texture(index)]]：纹理修饰符；可以加索引：[[texture(0)]] 对应纹理 0，[[texture(1)]] 对应纹理 1；
// KFFragmentTextureIndexTextureRGB：表示纹理索引。
fragment float4 rgbSamplingShader(RasterizerData input [[stage_in]],
                                  texture2d<half> colorTexture [[texture(KFFragmentTextureIndexTextureRGB)]]) {
    constexpr sampler textureSampler (mag_filter::linear, min_filter::linear);
    
    half4 colorSample = colorTexture.sample(textureSampler, input.textureCoordinate);
  
    return float4(colorSample);
}
```


接下来，就是封装渲染 Metal 渲染视图 `KFMetalView` 了，它接受 `CVPixelBufferRef` 作为参数来进行渲染。


`KFMetalView.h`

```objc
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

// 渲染画面填充模式。
typedef NS_ENUM(NSInteger, KFMetalViewContentMode) {
    // 自动填充满，可能会变形。
    KFMetalViewContentModeStretch = 0,
    // 按比例适配，可能会有黑边。
    KFMetalViewContentModeFit = 1,
    // 根据比例裁剪后填充满。
    KFMetalViewContentModeFill = 2
};

@interface KFMetalView : UIView
@property (nonatomic, assign) KFMetalViewContentMode fillMode; // 画面填充模式。
- (void)renderPixelBuffer:(CVPixelBufferRef)pixelBuffer; // 渲染。
@end

NS_ASSUME_NONNULL_END
```


`KFMetalView.m`

```objc
#import "KFMetalView.h"
#import <MetalKit/MetalKit.h>
#import <AVFoundation/AVFoundation.h>
#import <MetalPerformanceShaders/MetalPerformanceShaders.h>
#import "KFShaderType.h"

// 颜色空间转换矩阵，BT.601 Video Range。
static const matrix_float3x3 kFColorMatrix601VideoRange = (matrix_float3x3) {
    (simd_float3) {1.164,  1.164,  1.164},
    (simd_float3) {0.0,    -0.392,  2.017},
    (simd_float3) {1.596,  -0.813,   0.0},
};

// 颜色空间转换矩阵，BT.601 Full Range。
static const matrix_float3x3 kFColorMatrix601FullRange = (matrix_float3x3) {
    (simd_float3) {1.0,    1.0,    1.0},
    (simd_float3) {0.0,    -0.343, 1.765},
    (simd_float3) {1.4,    -0.711, 0.0},
};

// 颜色空间转换矩阵，BT.709 Video Range。
static const matrix_float3x3 kFColorMatrix709VideoRange = (matrix_float3x3) {
    (simd_float3) {1.164,  1.164, 1.164},
    (simd_float3) {0.0,   -0.213, 2.112},
    (simd_float3) {1.793, -0.533,   0.0},
};

// 颜色空间转换矩阵，BT.709 Full Range。
static const matrix_float3x3 kFColorMatrix709FullRange = (matrix_float3x3) {
    (simd_float3) { 1.0,    1.0,    1.0},
    (simd_float3) {0.0,    -0.187, 1.856},
    (simd_float3) {1.575,    -0.468, 0.0},
};

@interface KFMetalView () <MTKViewDelegate>
@property (nonatomic, assign) CVPixelBufferRef pixelBuffer; // 外层输入的最后一帧数据。
@property (nonatomic, strong) dispatch_semaphore_t semaphore; // 处理 PixelBuffer 锁，防止外层输入线程与渲染线程同时操作 Crash。
@property (nonatomic, assign) CVMetalTextureCacheRef textureCache; // 纹理缓存，根据 pixelbuffer 获取纹理。
@property (nonatomic, strong) MTKView *mtkView; // Metal 渲染的 view。
@property (nonatomic, assign) vector_uint2 viewportSize; // 视口大小。
@property (nonatomic, strong) id<MTLRenderPipelineState> pipelineState; // 渲染管道，管理顶点函数和片元函数。
@property (nonatomic, strong) id<MTLCommandQueue> commandQueue; // 渲染指令队列。
@property (nonatomic, strong) id<MTLBuffer> vertices; // 顶点缓存对象。
@property (nonatomic, assign) NSUInteger numVertices; // 顶点数量。
@property (nonatomic, strong) id<MTLBuffer> yuvMatrix; // YUV 数据矩阵对象。
@property (nonatomic, assign) BOOL updateFillMode; // 填充模式变更标记。
@property (nonatomic, assign) CGSize pixelBufferSize; // pixelBuffer 数据尺寸。
@property (nonatomic, assign) CGSize currentViewSize; // 当前视图大小。
@property (nonatomic, strong) dispatch_queue_t renderQueue; // 渲染线程。
@end

@implementation KFMetalView
#pragma mark - LifeCycle
- (instancetype)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    if (self) {
        _currentViewSize = frame.size;
        _fillMode = KFMetalViewContentModeFit;
        _updateFillMode = YES;
        //  创建 Metal 渲染视图且添加到当前视图。
        self.mtkView = [[MTKView alloc] initWithFrame:self.bounds];
        self.mtkView.device = MTLCreateSystemDefaultDevice();
        self.mtkView.backgroundColor = [UIColor clearColor];
        [self addSubview:self.mtkView];
        self.mtkView.delegate = self;
        self.mtkView.framebufferOnly = YES;
        self.viewportSize = (vector_uint2) {self.mtkView.drawableSize.width, self.mtkView.drawableSize.height};
        
        // 创建渲染线程。
        _semaphore = dispatch_semaphore_create(1);
        _renderQueue = dispatch_queue_create("com.KeyFrameKit.metalView.renderQueue", DISPATCH_QUEUE_SERIAL);
        
        // 创建纹理缓存。
        CVMetalTextureCacheCreate(NULL, NULL, self.mtkView.device, NULL, &_textureCache);
    }
    
    return self;
}

- (void)layoutSubviews {
    // 视图自动调整布局，同步至 Metal 视图。
    [super layoutSubviews];
    self.mtkView.frame = self.bounds;
    _currentViewSize = self.bounds.size;
}

- (void)dealloc {
    // 释放最后一帧数据、纹理缓存。
    dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
    if (_pixelBuffer) {
        CFRelease(_pixelBuffer);
        _pixelBuffer = NULL;
    }
    
    if (_textureCache) {
        CVMetalTextureCacheFlush(_textureCache, 0);
        CFRelease(_textureCache);
        _textureCache = NULL;
    }
    dispatch_semaphore_signal(_semaphore);
    [self.mtkView releaseDrawables];
}

#pragma mark - Public Method
- (void)renderPixelBuffer:(CVPixelBufferRef)pixelBuffer {
    if (!pixelBuffer) {
        return;
    }
    // 外层输入 BGRA、YUV 数据。
    dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
    if (_pixelBuffer) {
        CFRelease(_pixelBuffer);
        _pixelBuffer = NULL;
    }
    _pixelBuffer = pixelBuffer;
    _pixelBufferSize = CGSizeMake(CVPixelBufferGetWidth(pixelBuffer), CVPixelBufferGetHeight(pixelBuffer));
    CFRetain(pixelBuffer);
    dispatch_semaphore_signal(_semaphore);
}

- (void)setFillMode:(KFMetalViewContentMode)fillMode {
    // 更改视图填充模式。
    _fillMode = fillMode;
    _updateFillMode = YES;
}

#pragma mark - Private Method
-(void)_setupPipeline:(BOOL)isYUV {
    // 根据本地 shader 文件初始化渲染管道与渲染指令队列。
    id<MTLLibrary> defaultLibrary = [self.mtkView.device newDefaultLibrary];
    id<MTLFunction> vertexFunction = [defaultLibrary newFunctionWithName:@"vertexShader"];
    id<MTLFunction> fragmentFunction = [defaultLibrary newFunctionWithName:isYUV ? @"yuvSamplingShader" : @"rgbSamplingShader"];
    
    MTLRenderPipelineDescriptor *pipelineStateDescriptor = [[MTLRenderPipelineDescriptor alloc] init];
    pipelineStateDescriptor.vertexFunction = vertexFunction;
    pipelineStateDescriptor.fragmentFunction = fragmentFunction;
    pipelineStateDescriptor.colorAttachments[0].pixelFormat = self.mtkView.colorPixelFormat;
    self.pipelineState = [self.mtkView.device newRenderPipelineStateWithDescriptor:pipelineStateDescriptor error:NULL];
    self.commandQueue = [self.mtkView.device newCommandQueue];
}

- (void)_setupYUVMatrix:(BOOL)isFullRange colorSpace:(CFTypeRef)colorSpace{
    // 初始化 YUV 矩阵，判断 pixelBuffer 的颜色格式是 601 还是 709，创建对应的矩阵。
    KFConvertMatrix matrix;
    if (colorSpace == kCVImageBufferYCbCrMatrix_ITU_R_601_4) {
        matrix.matrix = isFullRange ? kFColorMatrix601FullRange : kFColorMatrix601VideoRange;
    }else if (colorSpace == kCVImageBufferYCbCrMatrix_ITU_R_709_2) {
        matrix.matrix = isFullRange ? kFColorMatrix709FullRange : kFColorMatrix709VideoRange;
    }
    matrix.fullRange = isFullRange;
    self.yuvMatrix = [self.mtkView.device newBufferWithBytes:&matrix
                                                          length:sizeof(KFConvertMatrix)
                                                         options:MTLResourceStorageModeShared];
}

- (void)_updaterVertices {
    // 根据填充模式计算顶点数据。
    float heightScaling = 1.0;
    float widthScaling = 1.0;
    
    if (!CGSizeEqualToSize(_currentViewSize, CGSizeZero) && !CGSizeEqualToSize(_pixelBufferSize, CGSizeZero)) {
        CGRect insetRect = AVMakeRectWithAspectRatioInsideRect(_pixelBufferSize, CGRectMake(0, 0, _currentViewSize.width, _currentViewSize.height));
        
        switch (_fillMode) {
            case KFMetalViewContentModeStretch: {
                widthScaling = 1.0;
                heightScaling = 1.0;
                break;
            }
            case KFMetalViewContentModeFit: {
                widthScaling = insetRect.size.width / _currentViewSize.width;
                heightScaling = insetRect.size.height / _currentViewSize.height;
                break;
            }
            case KFMetalViewContentModeFill: {
                widthScaling = _currentViewSize.height / insetRect.size.height;
                heightScaling = _currentViewSize.width / insetRect.size.width;
                break;
            }
        }
    }
    
    KFVertex quadVertices[] =
    {
        { { -widthScaling, -heightScaling, 0.0, 1.0 },  { 0.f, 1.f } },
        { { widthScaling,  -heightScaling, 0.0, 1.0 },  { 1.f, 1.f } },
        { { -widthScaling, heightScaling,  0.0, 1.0 },  { 0.f, 0.f } },
        { {  widthScaling, heightScaling,  0.0, 1.0 },  { 1.f, 0.f } },
    };
    // MTLResourceStorageModeShared 属性可共享的，表示可以被顶点或者片元函数或者其他函数使用。
    self.vertices = [self.mtkView.device newBufferWithBytes:quadVertices
                                                 length:sizeof(quadVertices)
                                                options:MTLResourceStorageModeShared];
    // 获取顶点数量。
    self.numVertices = sizeof(quadVertices) / sizeof(KFVertex);
}

- (BOOL)_pixelBufferIsFullRange:(CVPixelBufferRef)pixelBuffer {
    // 判断 YUV 数据是否为 full range。
    CFDictionaryRef cfDicAttributes = CVPixelBufferCopyCreationAttributes(pixelBuffer);
    NSDictionary *dicAttributes = (__bridge_transfer NSDictionary*)cfDicAttributes;
    if (dicAttributes && [dicAttributes objectForKey:@"PixelFormatDescription"]) {
        NSDictionary *pixelFormatDescription = [dicAttributes objectForKey:@"PixelFormatDescription"];
        if (pixelFormatDescription && [pixelFormatDescription objectForKey:(__bridge NSString*)kCVPixelFormatComponentRange]) {
            NSString *componentRange = [pixelFormatDescription objectForKey:(__bridge NSString*)kCVPixelFormatComponentRange];
            return [componentRange isEqualToString:(__bridge NSString*)kCVPixelFormatComponentRange_FullRange];
        }
    }
    return NO;
}

- (void)_drawInMTKView:(MTKView*)view {
    // 渲染数据。
    dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
    if (_pixelBuffer) {
        // 为当前渲染的每个渲染传递创建一个新的命令缓冲区。
        id<MTLCommandBuffer> commandBuffer = [self.commandQueue commandBuffer];
        // 获取渲染命令编码器 MTLRenderCommandEncoder 的描述符。
        // currentRenderPassDescriptor 描述符包含 currentDrawable 的纹理、视图的深度、模板和 sample 缓冲区和清晰的值。
        // MTLRenderPassDescriptor 描述一系列 attachments 的值，类似 OpenGL 的 FrameBuffer；同时也用来创建 MTLRenderCommandEncoder。
        MTLRenderPassDescriptor *renderPassDescriptor = view.currentRenderPassDescriptor;
        if (renderPassDescriptor) {
            // 根据描述创建 x 渲染命令编码器。
            id<MTLRenderCommandEncoder> renderEncoder = [commandBuffer renderCommandEncoderWithDescriptor:renderPassDescriptor];
            // 设置绘制区域。
            [renderEncoder setViewport:(MTLViewport) {0.0, 0.0, self.viewportSize.x, self.viewportSize.y, -1.0, 1.0 }];
            BOOL isRenderYUV = CVPixelBufferGetPlaneCount(_pixelBuffer) > 1;
            
            // 根据是否为 YUV 初始化渲染管道。
            if (!self.pipelineState) {
                [self _setupPipeline:isRenderYUV];
            }
            // 设置渲染管道。
            [renderEncoder setRenderPipelineState:self.pipelineState];
            
            // 更新填充模式。
            if (_updateFillMode) {
                [self _updaterVertices];
                _updateFillMode = NO;
            }
            // 传递顶点缓存。
            [renderEncoder setVertexBuffer:self.vertices
                                    offset:0
                                   atIndex:KFVertexInputIndexVertices];
            CVPixelBufferRef pixelBuffer = _pixelBuffer;
            
            if (isRenderYUV) {
                // 获取 y、uv 纹理。
                id<MTLTexture> textureY = nil;
                id<MTLTexture> textureUV = nil;
                {
                    size_t width = CVPixelBufferGetWidthOfPlane(pixelBuffer, 0);
                    size_t height = CVPixelBufferGetHeightOfPlane(pixelBuffer, 0);
                    MTLPixelFormat pixelFormat = MTLPixelFormatR8Unorm;
                    
                    CVMetalTextureRef texture = NULL;
                    CVReturn status = CVMetalTextureCacheCreateTextureFromImage(kCFAllocatorDefault, _textureCache, pixelBuffer, NULL, pixelFormat, width, height, 0, &texture);
                    if (status == kCVReturnSuccess) {
                        textureY = CVMetalTextureGetTexture(texture);
                        CFRelease(texture);
                    }
                }
                
                {
                    size_t width = CVPixelBufferGetWidthOfPlane(pixelBuffer, 1);
                    size_t height = CVPixelBufferGetHeightOfPlane(pixelBuffer, 1);
                    MTLPixelFormat pixelFormat = MTLPixelFormatRG8Unorm;
                    
                    CVMetalTextureRef texture = NULL;
                    CVReturn status = CVMetalTextureCacheCreateTextureFromImage(kCFAllocatorDefault, _textureCache, pixelBuffer, NULL, pixelFormat, width, height, 1, &texture);
                    if (status == kCVReturnSuccess) {
                        textureUV = CVMetalTextureGetTexture(texture);
                        CFRelease(texture);
                    }
                }
                
                // 传递纹理。
                if (textureY != nil && textureUV != nil) {
                    [renderEncoder setFragmentTexture:textureY
                                              atIndex:KFFragmentTextureIndexTextureY];
                    [renderEncoder setFragmentTexture:textureUV
                                              atIndex:KFFragmentTextureIndexTextureUV];
                }
                
                // 初始化 YUV 矩阵。
                if (!self.yuvMatrix) {
                    CFTypeRef matrixKey = CVBufferCopyAttachment(pixelBuffer, kCVImageBufferYCbCrMatrixKey, NULL);
                    [self _setupYUVMatrix:[self _pixelBufferIsFullRange:pixelBuffer] colorSpace:matrixKey];
                    CFRelease(matrixKey);
                }
                // 传递 YUV 矩阵。
                [renderEncoder setFragmentBuffer:self.yuvMatrix
                                          offset:0
                                         atIndex:KFFragmentInputIndexMatrix];
            } else {
                // 生成 rgba 纹理。
                id<MTLTexture> textureRGB = nil;
                size_t width = CVPixelBufferGetWidth(pixelBuffer);
                size_t height = CVPixelBufferGetHeight(pixelBuffer);
                MTLPixelFormat pixelFormat = MTLPixelFormatBGRA8Unorm;
                
                CVMetalTextureRef texture = NULL;
                CVReturn status = CVMetalTextureCacheCreateTextureFromImage(kCFAllocatorDefault, _textureCache, pixelBuffer, NULL, pixelFormat, width, height, 0, &texture);
                if (status == kCVReturnSuccess) {
                    textureRGB = CVMetalTextureGetTexture(texture);
                    CFRelease(texture);
                }
                
                // 传递纹理。
                if (textureRGB) {
                    [renderEncoder setFragmentTexture:textureRGB
                                              atIndex:KFFragmentTextureIndexTextureRGB];
                }
            }
            
            // 绘制。
            [renderEncoder drawPrimitives:MTLPrimitiveTypeTriangleStrip
                              vertexStart:0
                              vertexCount:self.numVertices];
            
            // 命令结束。
            [renderEncoder endEncoding];
            
            // 显示。
            [commandBuffer presentDrawable:view.currentDrawable];
            
            // 提交。
            [commandBuffer commit];
        }
        
        CFRelease(_pixelBuffer);
        _pixelBuffer = NULL;
    }
    dispatch_semaphore_signal(_semaphore);
}

#pragma mark - MTKViewDelegate
- (void)mtkView:(nonnull MTKView *)view drawableSizeWillChange:(CGSize)size {
    self.viewportSize = (vector_uint2) {size.width, size.height};
}

- (void)drawInMTKView:(nonnull MTKView *)view {
    // Metal 视图回调，有数据情况下渲染视图。
    __weak typeof(self) weakSelf = self;
    dispatch_async(_renderQueue, ^{
        [weakSelf _drawInMTKView:view];
    });
}

@end
```

更具体细节见上述代码及其注释。


## 3、采集视频数据并渲染



我们在一个 ViewController 中来实现对采集的视频数据进行渲染播放。


`KFVideoRenderViewController.m`


```objc
#import "KFVideoRenderViewController.h"
#import "KFVideoCapture.h"
#import "KFMetalView.h"

@interface KFVideoRenderViewController ()
@property (nonatomic, strong) KFVideoCaptureConfig *videoCaptureConfig;
@property (nonatomic, strong) KFVideoCapture *videoCapture;
@property (nonatomic, strong) KFMetalView *metalView;
@end

@implementation KFVideoRenderViewController
#pragma mark - Property
- (KFVideoCaptureConfig *)videoCaptureConfig {
    if (!_videoCaptureConfig) {
        _videoCaptureConfig = [[KFVideoCaptureConfig alloc] init];
    }
    
    return _videoCaptureConfig;
}

- (KFVideoCapture *)videoCapture {
    if (!_videoCapture) {
        _videoCapture = [[KFVideoCapture alloc] initWithConfig:self.videoCaptureConfig];
        __weak typeof(self) weakSelf = self;
        _videoCapture.sampleBufferOutputCallBack = ^(CMSampleBufferRef sampleBuffer) {
             // 视频采集数据回调。将采集回来的数据给渲染模块渲染。
            [weakSelf.metalView renderPixelBuffer:CMSampleBufferGetImageBuffer(sampleBuffer)];
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
    self.metalView.frame = self.view.bounds;
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
    _metalView = [[KFMetalView alloc] initWithFrame:self.view.bounds];
    _metalView.fillMode = KFMetalViewContentModeFill;
    [self.view addSubview:self.metalView];
}

@end
```

上面是 `KFVideoRenderViewController` 的实现，主要分为以下几个部分：

- 1）在页面加载完成后，启动采集模块。
	- 在 `-requestAccessForVideo` 方法中实现。
- 2）做好渲染模块 `KFMetalView` 的布局。
	- 在 `-setupUI` 方法中实现。
- 3）在采集模块的回调中将采集的视频数据给渲染模块渲染。
	- 在 `KFVideoCapture` 的 `sampleBufferOutputCallBack` 回调中实现。

更具体细节见上述代码及其注释。




