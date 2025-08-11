---
title: iOS 不用 libyuv 也能高效实现 RGB/YUV 数据转换
description: 介绍 iOS 使用 vImage 实现 RGB/YUV 数据转换的技术方案和实现源码。
author: Keyframe
date: 2025-02-25 08:08:08 +0800
categories: [音视频实战经验]
tags: [音视频实战经验, 音视频, 视频编辑, RGB, YUV, 编解码]
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

做音视频或图像处理方向的开发同学一般都或多或少接触过 [libyuv](https://chromium.googlesource.com/libyuv/libyuv "libyuv") 这个开源库，我们在音视频开发中处理 `YUV`、`RGB` 等格式的数据转换、旋转、缩放时常常用到它。libyuv 基于 C 语言实现，可以在 Windows、Linux、Mac OS、Android、iOS 等多平台使用，并且做了指令集加速优化，有比较好的性能表现。

其实苹果也提供了一套名为 `vImage` 的原生 API 来支持高性能的图像数据处理。`vImage` 是 `Accelerate` 库的一部分，使用传统 C 语言实现，侧重于高性能的图像处理，很多接口需要自己手动进行内存管理，我们基于 `vImage` 也可以实现高性能的 `RGB` 与 `YUV` 数据的转换。


## 1、BGRA 转换为 ARGB

下面的示例代码实现了 iOS `BGRA` 转换 `ARGB`，也打印了 `vImage` 与 `libyuv` 的处理耗时，接口与 `libyuv` 保持一致，反复测试发现性能基本一致。 

如果需要更改转换格式为 `RGBA` 等其它顺序，调整 `permuteMap` 内顺序即可。

```objc
+ (void)testBGRAToARGB:(CVPixelBufferRef)srcPixelBuffer dstPixelBuffer:(CVPixelBufferRef)dstPixelBuffer {
    CVPixelBufferLockBaseAddress(srcPixelBuffer, kCVPixelBufferLock_ReadOnly);
    CVPixelBufferLockBaseAddress(dstPixelBuffer, kCVPixelBufferLock_ReadOnly);
    
    NSTimeInterval time1 = [[NSDate date] timeIntervalSince1970] * 1000;
    
    // 使用 vImage 实现数据转换：
    [self.class BGRAToARGB:CVPixelBufferGetBaseAddress(srcPixelBuffer) src_stride:CVPixelBufferGetBytesPerRow(srcPixelBuffer) dst:CVPixelBufferGetBaseAddress(dstPixelBuffer) dst_stride:CVPixelBufferGetBytesPerRow(dstPixelBuffer) width:CVPixelBufferGetHeight(srcPixelBuffer) height:CVPixelBufferGetWidth(srcPixelBuffer)];
    
    NSTimeInterval time2 = [[NSDate date] timeIntervalSince1970] * 1000;

    // 使用 libyuv 实现数据转换：
    RGBAToARGB(CVPixelBufferGetBaseAddress(srcPixelBuffer),CVPixelBufferGetBytesPerRow(srcPixelBuffer),CVPixelBufferGetBaseAddress(dstPixelBuffer),CVPixelBufferGetBytesPerRow(dstPixelBuffer),CVPixelBufferGetHeight(srcPixelBuffer),CVPixelBufferGetWidth(srcPixelBuffer));

    NSTimeInterval time3 = [[NSDate date] timeIntervalSince1970] * 1000;
    NSLog(@"BGRAToARGB duration: vImage=%.2f, yuv=%.2f", time2 - time1, time3 - time2);
    
    CVPixelBufferUnlockBaseAddress(dstPixelBuffer, kCVPixelBufferLock_ReadOnly);
    CVPixelBufferUnlockBaseAddress(srcPixelBuffer, kCVPixelBufferLock_ReadOnly);
}


+ (int)BGRAToARGB:(const uint8_t *)src src_stride:(int)src_stride
              dst:(uint8_t *)dst dst_stride:(int)dst_stride width:(int)width height:(int)height {
    if (width == 0 || height == 0 || !src || src_stride == 0 || dst_stride == 0 || !dst) {
        return -1;
    }
    
    vImage_Buffer srcBufferInfo = {
       .width = width,
       .height = height,
       .rowBytes = src_stride,
       .data = src
    };
    
    vImage_Buffer dstBufferInfo = {
       .width = width,
       .height = height,
       .rowBytes = dst_stride,
       .data = dst
    };

    const uint8_t permuteMap[4] = {3, 2, 1, 0}; // transfer BGRA to ARGB
    return vImagePermuteChannels_ARGB8888(&srcBufferInfo, &dstBufferInfo, permuteMap, kvImageNoFlags);
}
```

下面是我们进行几次测试得到的耗时：

```
BGRAToARGB duration: vImage=0.11, yuv=0.13
BGRAToARGB duration: vImage=0.10, yuv=0.10
BGRAToARGB duration: vImage=0.13, yuv=0.09
BGRAToARGB duration: vImage=0.09, yuv=0.10
BGRAToARGB duration: vImage=0.12, yuv=0.13
BGRAToARGB duration: vImage=0.10, yuv=0.10
BGRAToARGB duration: vImage=0.09, yuv=0.08
```

**示例代码中的关键参数解析：**

- `permuteMap` 中四个参数值代表了源格式四个通道的 Index，四个参数的顺序则代表了原格式四个通道在目标格式中的顺序。
- `vImagePermuteChannels_ARGB8888` 参数是原始格式与目标格式对应 `vImage_Buffer` 及 `permuteMap`，每个 `vImage_Buffer` 设置 宽、高、行字节数、地址指针。
- `CVPixelBuffer` 获取地址需要先调用 `CVPixelBufferLockBaseAddress` 加锁，使用完成再调用 `CVPixelBufferUnlockBaseAddress` 解锁。

## 2、YUV(NV12) 转换为 BGRA

在 iOS 中要将 `NV12` 格式的 CVPixelBuffer 保存为 UIImage 时，例如视频抽帧等需求，需要先将 `NV12` 转换 `BGRA`。下面的示例代码实现并对比了 `vImage` 与 `libyuv` 性能，接口与 `libyuv` 保持一致，反复测试发现性能基本一致。

如果原数据为 `I420`，可更改转换方法为 `vImageConvert_420Yp8_Cb8_Cr8ToARGB8888`。

```objc
+ (void)testNV12ToBGRA:(CVPixelBufferRef)srcPixelBuffer dstPixelBuffer:(CVPixelBufferRef)dstPixelBuffer {
    vImage_Flags flags = kvImageNoFlags;
    static vImage_YpCbCrToARGB info;
    static dispatch_once_t onceTokenYUVToBGRA;
    dispatch_once(&onceTokenYUVToBGRA, ^{
        // vImage_YpCbCrPixelRange pixelRange = (vImage_YpCbCrPixelRange){16, 128, 235, 240, 255, 0, 255, 1};
        vImage_YpCbCrPixelRange pixelRange = {0, 128, 255, 255, 255, 1, 255, 0}; //fullrange - fullrange
        
        vImageConvert_YpCbCrToARGB_GenerateConversion(kvImage_YpCbCrToARGBMatrix_ITU_R_709_2, &pixelRange,
                                                      &info, kvImage420Yp8_CbCr8,
                                                      kvImageARGB8888, flags);
    });
    
    CVPixelBufferLockBaseAddress(srcPixelBuffer, kCVPixelBufferLock_ReadOnly);
    CVPixelBufferLockBaseAddress(dstPixelBuffer, kCVPixelBufferLock_ReadOnly);
    
    NSTimeInterval time1 = [[NSDate date] timeIntervalSince1970] * 1000;

    // 使用 vImage 实现数据转换：
    [self.class NV12ToBGRA:CVPixelBufferGetBaseAddressOfPlane(srcPixelBuffer, 0) src_stride_y:CVPixelBufferGetBytesPerRowOfPlane(srcPixelBuffer, 0) src_uv:CVPixelBufferGetBaseAddressOfPlane(srcPixelBuffer, 1) src_stride_uv:CVPixelBufferGetBytesPerRowOfPlane(srcPixelBuffer, 1) dst:CVPixelBufferGetBaseAddress(dstPixelBuffer) dst_stride:CVPixelBufferGetBytesPerRow(dstPixelBuffer) width:CVPixelBufferGetWidthOfPlane(srcPixelBuffer, 0) height:CVPixelBufferGetHeightOfPlane(srcPixelBuffer, 0) info:info];
    
    NSTimeInterval time2 = [[NSDate date] timeIntervalSince1970] * 1000;

    // 使用 libyuv 实现数据转换：
    NV12ToARGB(CVPixelBufferGetBaseAddressOfPlane(srcPixelBuffer, 0), CVPixelBufferGetBytesPerRowOfPlane(srcPixelBuffer, 0), CVPixelBufferGetBaseAddressOfPlane(srcPixelBuffer, 1), CVPixelBufferGetBytesPerRowOfPlane(srcPixelBuffer, 1), CVPixelBufferGetBaseAddress(dstPixelBuffer), CVPixelBufferGetBytesPerRow(dstPixelBuffer), CVPixelBufferGetWidthOfPlane(srcPixelBuffer, 0), CVPixelBufferGetHeightOfPlane(srcPixelBuffer, 0));

    NSTimeInterval time3 = [[NSDate date] timeIntervalSince1970] * 1000;
    NSLog(@"NV12ToBGRA duration: vImage=%.2f, yuv=%.2f", time2 - time1, time3 - time2);
    
    CVPixelBufferUnlockBaseAddress(dstPixelBuffer, kCVPixelBufferLock_ReadOnly);
    CVPixelBufferUnlockBaseAddress(srcPixelBuffer, kCVPixelBufferLock_ReadOnly);
}

+ (int)NV12ToBGRA:(uint8_t *)src_y src_stride_y:(int)src_stride_y src_uv:(uint8_t*)src_uv src_stride_uv:(int)src_stride_uv dst:(const uint8_t *)dst dst_stride:(int)dst_stride
            width:(int)width height:(int)height info:(vImage_YpCbCrToARGB)info {
    if (!src_y || src_stride_y == 0 || !src_uv || src_stride_uv == 0 || !dst || dst_stride == 0 || width == 0 || height == 0) {
        return -1;
    }
    
    vImage_Flags flags = kvImageNoFlags;
    
    vImage_Buffer srcYBufferInfo = {
       .width = width,
       .height = height,
       .rowBytes = src_stride_y,
       .data = src_y};
    
    vImage_Buffer srcUVBufferInfo = {
       .width = (width  + 1) >> 1,
       .height = (height + 1) >> 1,
       .rowBytes = src_stride_uv,
       .data = src_uv};
    
    vImage_Buffer dstBufferInfo = {
       .width = width,
       .height = height,
       .rowBytes = dst_stride,
       .data = dst};
    
    const uint8_t permuteMap[4] = {3, 2, 1, 0}; // transfer ARGB to BGRA
    return vImageConvert_420Yp8_CbCr8ToARGB8888(&srcYBufferInfo, &srcUVBufferInfo, &dstBufferInfo, &info, 
                                                permuteMap, 255, flags);
}
```

下面是我们进行几次测试得到的耗时：


```
NV12ToBGRA duration: vImage=0.16, yuv=0.13
NV12ToBGRA duration: vImage=0.13, yuv=0.12
NV12ToBGRA duration: vImage=0.14, yuv=0.13
NV12ToBGRA duration: vImage=0.13, yuv=0.13
NV12ToBGRA duration: vImage=0.15, yuv=0.11
NV12ToBGRA duration: vImage=0.12, yuv=0.11
NV12ToBGRA duration: vImage=0.12, yuv=0.11
NV12ToBGRA duration: vImage=0.14, yuv=0.11
NV12ToBGRA duration: vImage=0.13, yuv=0.10
```

**示例代码中的关键参数解析：**

- `vImageConvert_YpCbCrToARGB_GenerateConversion` 需要设置 `601`、`709`，以及 `FullRange`、`VideoRange`。`kvImage420Yp8_CbCr8` 代表 `NV12`。
- `permuteMap` 中四个参数值代表了源格式四个通道的 Index，四个参数的顺序则代表了原格式四个通道在目标格式中的顺序。
- `vImageConvert_420Yp8_CbCr8ToARGB8888` 函数默认转换为 `ARGB`，但目标转换为 `BGRA`，所以需调整设置 `permuteMap`。
- `CVPixelBuffer` 获取地址需要先调用 `CVPixelBufferLockBaseAddress` 加锁，使用完成再调用 `CVPixelBufferUnlockBaseAddress` 解锁。
- `srcUVBufferInfo` 宽高针对于奇数会向上取整，对齐 `libyuv`。


## 3、BGRA 转换为 YUV(NV12) 

当 UIImage 转换为 `NV12` 格式的 CVPixelBuffer时，例如图片与视频混排需要将图片转换为 `NV12` 进行编码，需要先将 `BGRA` 转换 `NV12`。下面的示例代码实现并对比了 `vImage` 与 `libyuv` 性能，接口与 `libyuv` 保持一致，反复测试发现性能基本一致。

如果需要转换为 `I420`，可更改转换方法为 `vImageConvert_ARGB8888To420Yp8_Cb8_Cr8`。

```objc
+ (void)testBGRAToNV12:(CVPixelBufferRef)srcPixelBuffer dstPixelBuffer:(CVPixelBufferRef)dstPixelBuffer{
    vImage_Flags flags = kvImageNoFlags;
    static vImage_ARGBToYpCbCr info;
    static dispatch_once_t onceTokenBGRAToNV12;
    dispatch_once(&onceTokenBGRAToNV12, ^{
        // vImage_YpCbCrPixelRange pixelRange = (vImage_YpCbCrPixelRange){16, 128, 235, 240, 255, 0, 255, 1};
        vImage_YpCbCrPixelRange pixelRange = (vImage_YpCbCrPixelRange){0, 128, 255, 255, 255, 1, 255, 0};
        vImageConvert_ARGBToYpCbCr_GenerateConversion(
                                                      kvImage_ARGBToYpCbCrMatrix_ITU_R_709_2,
                                                      &pixelRange,
                                                      &info,
                                                      kvImageARGB8888,
                                                      kvImage420Yp8_CbCr8,
                                                      0);
    });
    
    CVPixelBufferLockBaseAddress(srcPixelBuffer, kCVPixelBufferLock_ReadOnly);
    CVPixelBufferLockBaseAddress(dstPixelBuffer, kCVPixelBufferLock_ReadOnly);
    
    NSTimeInterval time1 = [[NSDate date] timeIntervalSince1970] * 1000;

    // 使用 vImage 实现数据转换：
    [self.class BGRAToNV12:CVPixelBufferGetBaseAddress(srcPixelBuffer) src_stride:CVPixelBufferGetBytesPerRow(srcPixelBuffer) dst_y:CVPixelBufferGetBaseAddressOfPlane(dstPixelBuffer, 0) dst_stride_y:CVPixelBufferGetBytesPerRowOfPlane(dstPixelBuffer, 0) dst_uv:CVPixelBufferGetBaseAddressOfPlane(dstPixelBuffer, 1) dst_stride_uv:CVPixelBufferGetBytesPerRowOfPlane(dstPixelBuffer, 1) width:CVPixelBufferGetWidthOfPlane(srcPixelBuffer, 0) height:CVPixelBufferGetHeightOfPlane(srcPixelBuffer, 0) info:info];

    NSTimeInterval time2 = [[NSDate date] timeIntervalSince1970] * 1000;

    // 使用 libyuv 实现数据转换：
    ARGBToNV12(CVPixelBufferGetBaseAddress(srcPixelBuffer), CVPixelBufferGetBytesPerRow(srcPixelBuffer), CVPixelBufferGetBaseAddressOfPlane(dstPixelBuffer, 0), CVPixelBufferGetBytesPerRowOfPlane(dstPixelBuffer, 0), CVPixelBufferGetBaseAddressOfPlane(dstPixelBuffer, 1), CVPixelBufferGetBytesPerRowOfPlane(dstPixelBuffer, 1), CVPixelBufferGetWidthOfPlane(srcPixelBuffer, 0), CVPixelBufferGetHeightOfPlane(srcPixelBuffer, 0));

    NSTimeInterval time3 = [[NSDate date] timeIntervalSince1970] * 1000;
    NSLog(@"BGRAToNV12 duration: vImage=%.2f, yuv=%.2f", time2 - time1, time3 - time2);

    CVPixelBufferUnlockBaseAddress(dstPixelBuffer, kCVPixelBufferLock_ReadOnly);
    CVPixelBufferUnlockBaseAddress(srcPixelBuffer, kCVPixelBufferLock_ReadOnly);
}

+ (int)BGRAToNV12:(const uint8_t*)src src_stride:(int)src_stride
            dst_y:(uint8_t*)dst_y dst_stride_y:(int)dst_stride_y dst_uv:(uint8_t*)dst_uv dst_stride_uv:(int)dst_stride_uv width:(int)width height:(int)height info:(vImage_ARGBToYpCbCr)info{
    if(!src || src_stride == 0 || !dst_y || dst_stride_y == 0 || !dst_uv || dst_stride_uv == 0 || width == 0 || height == 0){
        return -1;
    }
    
    vImage_Buffer srcBufferInfo = {
       .width = width,
       .height = height,
       .rowBytes = src_stride,
       .data = src};
    
    vImage_Buffer yBufferInfo = {
       .width = width,
       .height = height,
       .rowBytes = dst_stride_y,
       .data = dst_y};
    
    vImage_Buffer uvBufferInfo = {
       .width = (width  + 1) >> 1,
       .height = (height + 1) >> 1,
       .rowBytes = dst_stride_uv,
       .data = dst_uv};

    const uint8_t permuteMap[4] = { 3, 2, 1, 0}; // transfer ARGB to BGRA
    return vImageConvert_ARGB8888To420Yp8_CbCr8(
       &srcBufferInfo,
       &yBufferInfo,
       &uvBufferInfo,
       &info,
       permuteMap,
       kvImageDoNotTile);
}
```

下面是我们进行几次测试得到的耗时：

```
BGRAToNV12 duration: vImage=0.16, yuv=0.12
BGRAToNV12 duration: vImage=0.10, yuv=0.12
BGRAToNV12 duration: vImage=0.10, yuv=0.12
BGRAToNV12 duration: vImage=0.10, yuv=0.19
BGRAToNV12 duration: vImage=0.10, yuv=0.12
BGRAToNV12 duration: vImage=0.16, yuv=0.19
BGRAToNV12 duration: vImage=0.10, yuv=0.12
BGRAToNV12 duration: vImage=0.10, yuv=0.12
```

**示例代码中的关键参数解析：**

- `vImageConvert_ARGBToYpCbCr_GenerateConversion ` 需要设置 `601`、`709`，以及 `FullRange`、`VideoRange`。`kvImage420Yp8_CbCr8 ` 代表 `NV12`。
- `permuteMap` 中四个参数值代表了源格式四个通道的 Index，四个参数的顺序则代表了原格式四个通道在目标格式中的顺序。
- `vImageConvert_ARGB8888To420Yp8_CbCr8` 函数默认转换为 `ARGB`，但目标转换为 `BGRA`，所以需调整设置 `permuteMap`。
- `CVPixelBuffer` 获取地址需要先调用 `CVPixelBufferLockBaseAddress` 加锁，使用完成再调用 `CVPixelBufferUnlockBaseAddress` 解锁。
- `uvBufferInfo ` 宽高针对于奇数会向上取整，对齐 `libyuv`。





---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

