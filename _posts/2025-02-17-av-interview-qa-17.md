---
title: 音视频面试题集锦第 17 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:58:28 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦, 音视频]
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



我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。



下面是第 17 期面试题精选：

- **1、聊聊 iOS CVPixelBufferRef 相关的细节？**
- **2、聊聊对音视频同步的理解？**





## 1、聊聊 iOS CVPixelBufferRef 相关的细节？


CVPixelBufferRef 像素缓冲区，是 iOS 平台进行视频编解码及图像处理相关最重要的数据结构之一。它的定义是 `typedef CVImageBufferRef CVPixelBufferRef`。CVPixelBuffer 是在 CVImageBuffer 的基础上实现了内存存储。并且，CVPixelBuffer 还可以实现 CPU 和 GPU 共享内存，为图像处理提供更高的效率。


**1）CVPixelBufferRef 数据格式类型**


常用的 CVPixelBufferRef 数据格式（kCVPixelFormatType）有：

- kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange
- kCVPixelFormatType_420YpCbCr8BiPlanarFullRange
- kCVPixelFormatType_32RGBA
- kCVPixelFormatType_32BGRA
- kCVPixelFormatType_OneComponent8

全部类型参考系统头文件：`CVPixelBuffer.h`：

```c
{
    kCVPixelFormatType_1Monochrome    = 0x00000001, /* 1 bit indexed */
    kCVPixelFormatType_2Indexed       = 0x00000002, /* 2 bit indexed */
    kCVPixelFormatType_4Indexed       = 0x00000004, /* 4 bit indexed */
    kCVPixelFormatType_8Indexed       = 0x00000008, /* 8 bit indexed */
    kCVPixelFormatType_1IndexedGray_WhiteIsZero = 0x00000021, /* 1 bit indexed gray, white is zero */
    kCVPixelFormatType_2IndexedGray_WhiteIsZero = 0x00000022, /* 2 bit indexed gray, white is zero */
    kCVPixelFormatType_4IndexedGray_WhiteIsZero = 0x00000024, /* 4 bit indexed gray, white is zero */
    kCVPixelFormatType_8IndexedGray_WhiteIsZero = 0x00000028, /* 8 bit indexed gray, white is zero */
    kCVPixelFormatType_16BE555        = 0x00000010, /* 16 bit BE RGB 555 */
    kCVPixelFormatType_16LE555        = 'L555',     /* 16 bit LE RGB 555 */
    kCVPixelFormatType_16LE5551       = '5551',     /* 16 bit LE RGB 5551 */
    kCVPixelFormatType_16BE565        = 'B565',     /* 16 bit BE RGB 565 */
    kCVPixelFormatType_16LE565        = 'L565',     /* 16 bit LE RGB 565 */
    kCVPixelFormatType_24RGB          = 0x00000018, /* 24 bit RGB */
    kCVPixelFormatType_24BGR          = '24BG',     /* 24 bit BGR */
    kCVPixelFormatType_32ARGB         = 0x00000020, /* 32 bit ARGB */
    kCVPixelFormatType_32BGRA         = 'BGRA',     /* 32 bit BGRA */
    kCVPixelFormatType_32ABGR         = 'ABGR',     /* 32 bit ABGR */
    kCVPixelFormatType_32RGBA         = 'RGBA',     /* 32 bit RGBA */
    kCVPixelFormatType_64ARGB         = 'b64a',     /* 64 bit ARGB, 16-bit big-endian samples */
    kCVPixelFormatType_64RGBALE       = 'l64r',     /* 64 bit RGBA, 16-bit little-endian full-range (0-65535) samples */
    kCVPixelFormatType_48RGB          = 'b48r',     /* 48 bit RGB, 16-bit big-endian samples */
    kCVPixelFormatType_32AlphaGray    = 'b32a',     /* 32 bit AlphaGray, 16-bit big-endian samples, black is zero */
    kCVPixelFormatType_16Gray         = 'b16g',     /* 16 bit Grayscale, 16-bit big-endian samples, black is zero */
    kCVPixelFormatType_30RGB          = 'R10k',     /* 30 bit RGB, 10-bit big-endian samples, 2 unused padding bits (at least significant end). */
    kCVPixelFormatType_422YpCbCr8     = '2vuy',     /* Component Y'CbCr 8-bit 4:2:2, ordered Cb Y'0 Cr Y'1 */
    kCVPixelFormatType_4444YpCbCrA8   = 'v408',     /* Component Y'CbCrA 8-bit 4:4:4:4, ordered Cb Y' Cr A */
    kCVPixelFormatType_4444YpCbCrA8R  = 'r408',     /* Component Y'CbCrA 8-bit 4:4:4:4, rendering format. full range alpha, zero biased YUV, ordered A Y' Cb Cr */
    kCVPixelFormatType_4444AYpCbCr8   = 'y408',     /* Component Y'CbCrA 8-bit 4:4:4:4, ordered A Y' Cb Cr, full range alpha, video range Y'CbCr. */
    kCVPixelFormatType_4444AYpCbCr16  = 'y416',     /* Component Y'CbCrA 16-bit 4:4:4:4, ordered A Y' Cb Cr, full range alpha, video range Y'CbCr, 16-bit little-endian samples. */
    kCVPixelFormatType_444YpCbCr8     = 'v308',     /* Component Y'CbCr 8-bit 4:4:4, ordered Cr Y' Cb, video range Y'CbCr */
    kCVPixelFormatType_422YpCbCr16    = 'v216',     /* Component Y'CbCr 10,12,14,16-bit 4:2:2 */
    kCVPixelFormatType_422YpCbCr10    = 'v210',     /* Component Y'CbCr 10-bit 4:2:2 */
    kCVPixelFormatType_444YpCbCr10    = 'v410',     /* Component Y'CbCr 10-bit 4:4:4 */
    kCVPixelFormatType_420YpCbCr8Planar = 'y420',   /* Planar Component Y'CbCr 8-bit 4:2:0.  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrPlanar struct */
    kCVPixelFormatType_420YpCbCr8PlanarFullRange    = 'f420',   /* Planar Component Y'CbCr 8-bit 4:2:0, full range.  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrPlanar struct */
    kCVPixelFormatType_422YpCbCr_4A_8BiPlanar = 'a2vy', /* First plane: Video-range Component Y'CbCr 8-bit 4:2:2, ordered Cb Y'0 Cr Y'1; second plane: alpha 8-bit 0-255 */
    kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange = '420v', /* Bi-Planar Component Y'CbCr 8-bit 4:2:0, video-range (luma=[16,235] chroma=[16,240]).  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrBiPlanar struct */
    kCVPixelFormatType_420YpCbCr8BiPlanarFullRange  = '420f', /* Bi-Planar Component Y'CbCr 8-bit 4:2:0, full-range (luma=[0,255] chroma=[1,255]).  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrBiPlanar struct */ 
    kCVPixelFormatType_422YpCbCr8BiPlanarVideoRange = '422v', /* Bi-Planar Component Y'CbCr 8-bit 4:2:2, video-range (luma=[16,235] chroma=[16,240]).  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrBiPlanar struct */
    kCVPixelFormatType_422YpCbCr8BiPlanarFullRange  = '422f', /* Bi-Planar Component Y'CbCr 8-bit 4:2:2, full-range (luma=[0,255] chroma=[1,255]).  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrBiPlanar struct */
    kCVPixelFormatType_444YpCbCr8BiPlanarVideoRange = '444v', /* Bi-Planar Component Y'CbCr 8-bit 4:4:4, video-range (luma=[16,235] chroma=[16,240]).  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrBiPlanar struct */
    kCVPixelFormatType_444YpCbCr8BiPlanarFullRange  = '444f', /* Bi-Planar Component Y'CbCr 8-bit 4:4:4, full-range (luma=[0,255] chroma=[1,255]).  baseAddr points to a big-endian CVPlanarPixelBufferInfo_YCbCrBiPlanar struct */
    kCVPixelFormatType_422YpCbCr8_yuvs = 'yuvs',     /* Component Y'CbCr 8-bit 4:2:2, ordered Y'0 Cb Y'1 Cr */
    kCVPixelFormatType_422YpCbCr8FullRange = 'yuvf', /* Component Y'CbCr 8-bit 4:2:2, full range, ordered Y'0 Cb Y'1 Cr */
    kCVPixelFormatType_OneComponent8  = 'L008',     /* 8 bit one component, black is zero */
    kCVPixelFormatType_TwoComponent8  = '2C08',     /* 8 bit two component, black is zero */
    kCVPixelFormatType_30RGBLEPackedWideGamut = 'w30r', /* little-endian RGB101010, 2 MSB are zero, wide-gamut (384-895) */
    kCVPixelFormatType_ARGB2101010LEPacked = 'l10r',     /* little-endian ARGB2101010 full-range ARGB */
    kCVPixelFormatType_40ARGBLEWideGamut      = 'w40a', /* little-endian ARGB10101010, each 10 bits in the MSBs of 16bits, wide-gamut (384-895, including alpha) */
    kCVPixelFormatType_40ARGBLEWideGamutPremultiplied = 'w40m', /* little-endian ARGB10101010, each 10 bits in the MSBs of 16bits, wide-gamut (384-895, including alpha). Alpha premultiplied */
    kCVPixelFormatType_OneComponent10      = 'L010',     /* 10 bit little-endian one component, stored as 10 MSBs of 16 bits, black is zero */
    kCVPixelFormatType_OneComponent12      = 'L012',     /* 12 bit little-endian one component, stored as 12 MSBs of 16 bits, black is zero */
    kCVPixelFormatType_OneComponent16      = 'L016',     /* 16 bit little-endian one component, black is zero */
    kCVPixelFormatType_TwoComponent16      = '2C16',     /* 16 bit little-endian two component, black is zero */
    kCVPixelFormatType_OneComponent16Half  = 'L00h',     /* 16 bit one component IEEE half-precision float, 16-bit little-endian samples */
    kCVPixelFormatType_OneComponent32Float = 'L00f',     /* 32 bit one component IEEE float, 32-bit little-endian samples */
    kCVPixelFormatType_TwoComponent16Half  = '2C0h',     /* 16 bit two component IEEE half-precision float, 16-bit little-endian samples */
    kCVPixelFormatType_TwoComponent32Float = '2C0f',     /* 32 bit two component IEEE float, 32-bit little-endian samples */
    kCVPixelFormatType_64RGBAHalf          = 'RGhA',     /* 64 bit RGBA IEEE half-precision float, 16-bit little-endian samples */
    kCVPixelFormatType_128RGBAFloat        = 'RGfA',     /* 128 bit RGBA IEEE float, 32-bit little-endian samples */
    kCVPixelFormatType_14Bayer_GRBG        = 'grb4',     /* Bayer 14-bit Little-Endian, packed in 16-bits, ordered G R G R... alternating with B G B G... */
    kCVPixelFormatType_14Bayer_RGGB        = 'rgg4',     /* Bayer 14-bit Little-Endian, packed in 16-bits, ordered R G R G... alternating with G B G B... */
    kCVPixelFormatType_14Bayer_BGGR        = 'bgg4',     /* Bayer 14-bit Little-Endian, packed in 16-bits, ordered B G B G... alternating with G R G R... */
    kCVPixelFormatType_14Bayer_GBRG        = 'gbr4',     /* Bayer 14-bit Little-Endian, packed in 16-bits, ordered G B G B... alternating with R G R G... */
    kCVPixelFormatType_DisparityFloat16     = 'hdis',     /* IEEE754-2008 binary16 (half float), describing the normalized shift when comparing two images. Units are 1/meters: ( pixelShift / (pixelFocalLength * baselineInMeters) ) */
    kCVPixelFormatType_DisparityFloat32     = 'fdis',     /* IEEE754-2008 binary32 float, describing the normalized shift when comparing two images. Units are 1/meters: ( pixelShift / (pixelFocalLength * baselineInMeters) ) */
    kCVPixelFormatType_DepthFloat16         = 'hdep',     /* IEEE754-2008 binary16 (half float), describing the depth (distance to an object) in meters */
    kCVPixelFormatType_DepthFloat32         = 'fdep',     /* IEEE754-2008 binary32 float, describing the depth (distance to an object) in meters */
    kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange = 'x420', /* 2 plane YCbCr10 4:2:0, each 10 bits in the MSBs of 16bits, video-range (luma=[64,940] chroma=[64,960]) */
    kCVPixelFormatType_422YpCbCr10BiPlanarVideoRange = 'x422', /* 2 plane YCbCr10 4:2:2, each 10 bits in the MSBs of 16bits, video-range (luma=[64,940] chroma=[64,960]) */
    kCVPixelFormatType_444YpCbCr10BiPlanarVideoRange = 'x444', /* 2 plane YCbCr10 4:4:4, each 10 bits in the MSBs of 16bits, video-range (luma=[64,940] chroma=[64,960]) */
    kCVPixelFormatType_420YpCbCr10BiPlanarFullRange  = 'xf20', /* 2 plane YCbCr10 4:2:0, each 10 bits in the MSBs of 16bits, full-range (Y range 0-1023) */
    kCVPixelFormatType_422YpCbCr10BiPlanarFullRange  = 'xf22', /* 2 plane YCbCr10 4:2:2, each 10 bits in the MSBs of 16bits, full-range (Y range 0-1023) */
    kCVPixelFormatType_444YpCbCr10BiPlanarFullRange  = 'xf44', /* 2 plane YCbCr10 4:4:4, each 10 bits in the MSBs of 16bits, full-range (Y range 0-1023) */
    kCVPixelFormatType_420YpCbCr8VideoRange_8A_TriPlanar   = 'v0a8', /* first and second planes as per 420YpCbCr8BiPlanarVideoRange (420v), alpha 8 bits in third plane full-range.  No CVPlanarPixelBufferInfo struct. */
    kCVPixelFormatType_16VersatileBayer    = 'bp16',   /* Single plane Bayer 16-bit little-endian sensor element ("sensel") samples from full-size decoding of ProRes RAW images; Bayer pattern (sensel ordering) and other raw conversion information is described via buffer attachments */
    kCVPixelFormatType_64RGBA_DownscaledProResRAW    = 'bp64',   /* Single plane 64-bit RGBA (16-bit little-endian samples) from downscaled decoding of ProRes RAW images; components--which may not be co-sited with one another--are sensel values and require raw conversion, information for which is described via buffer attachments */
    kCVPixelFormatType_422YpCbCr16BiPlanarVideoRange       = 'sv22', /* 2 plane YCbCr16 4:2:2, video-range (luma=[4096,60160] chroma=[4096,61440]) */
    kCVPixelFormatType_444YpCbCr16BiPlanarVideoRange       = 'sv44', /* 2 plane YCbCr16 4:4:4, video-range (luma=[4096,60160] chroma=[4096,61440]) */
    kCVPixelFormatType_444YpCbCr16VideoRange_16A_TriPlanar = 's4as', /* 3 plane video-range YCbCr16 4:4:4 with 16-bit full-range alpha (luma=[4096,60160] chroma=[4096,61440] alpha=[0,65535]).  No CVPlanarPixelBufferInfo struct. */
}
```

**2）CVPixelBufferRef 的内存管理和使用**

使用 CVPixelBufferRef 时要手动管理内存，常用到的引用计数相关方法如下：

```c
CVPixelBufferRetain / CFRetain
CVPixelBufferRelease / CFRelease
CFGetRetainCount
```

访问 CVPixelBufferRef 内存数据常用的 API 如下：

- [CVPixelBufferLockBaseAddress(...)](https://developer.apple.com/documentation/corevideo/1457128-cvpixelbufferlockbaseaddress?language=objc "CVPixelBufferLockBaseAddress(...)")：锁定 Pixel Buffer 的内存基地址。当使用 CPU 读取 Pixel 数据时，需要读取时锁定，读完解锁。如果在锁定时，带了 `kCVPixelBufferLock_ReadOnly` 的 lockFlags，解锁时也要带上。但是，如果使用 GPU 读取 Pixel 数据时，则没有必要锁定，反而会影响性能。
- [CVPixelBufferUnlockBaseAddress(...)](https://developer.apple.com/documentation/corevideo/1456843-cvpixelbufferunlockbaseaddress?language=objc "CVPixelBufferUnlockBaseAddress(...)")：解锁 Pixel Buffer 的内存基地址。
- [CVPixelBufferGetBaseAddress(...)](https://developer.apple.com/documentation/corevideo/1457115-cvpixelbuffergetbaseaddress?language=objc "CVPixelBufferLockBaseAddress(...)")：返回 Pixel Buffer 的内存基地址，但是根据 Buffer 的类型及创建场景的不同，返回的值的含义也有区别。对于 Chunky Buffers，返回的是坐标 (0, 0) 像素的内存地址；对于 Planar Buffers，返回的是对应的 CVPlanarComponentInfo 结构体的内存地址或者 NULL（如果不存在 CVPlanarComponentInfo 结构体的话），所以，对于 Planar Buffers，最好用 `CVPixelBufferGetBaseAddressOfPlane(...)` 和 `CVPixelBufferGetBytesPerRowOfPlane(...)` 来获取其中数据。获取 Pixel Buffer 的基地址时，需要先用 `CVPixelBufferLockBaseAddress(...)` 加锁。
- [CVPixelBufferGetHeight(...)](https://developer.apple.com/documentation/corevideo/1456666-cvpixelbuffergetheight?language=objc "CVPixelBufferGetHeight(...)")：返回 Buffer 的像素高度。
- [CVPixelBufferGetWidth(...)](https://developer.apple.com/documentation/corevideo/1457241-cvpixelbuffergetwidth?language=objc "CVPixelBufferGetWidth(...)")：返回 Buffer 的像素宽度。
- [CVPixelBufferIsPlanar(...)](https://developer.apple.com/documentation/corevideo/1456805-cvpixelbufferisplanar?language=objc "CVPixelBufferIsPlanar(...)")：判断 Pixel Buffer 是否是 Planar 类型。
- [CVPixelBufferGetBaseAddressOfPlane(...)](https://developer.apple.com/documentation/corevideo/1456821-cvpixelbuffergetbaseaddressofpla?language=objc "CVPixelBufferGetBaseAddressOfPlane(...)")：根据指定的 Plane Index 来获取对应的基地址。获取 Pixel Buffer 的基地址时，需要先用 `CVPixelBufferLockBaseAddress(...)` 加锁。怎么理解 Plane 呢？其实主要跟颜色模型有关，比如：存储 YUV420P 时，有 Y、U、V 这 3 个 Plane；存储 NV12 时，有 Y、UV 这 2 个 Plane；存储 RGBA 时，有 R、G、B、A 这 4 个 Plane；而在 Packed 存储模式中，因为所有分量的像素是交织存储的，所以只有 1 个 Plane。
- [CVPixelBufferGetPlaneCount(...)](https://developer.apple.com/documentation/corevideo/1456976-cvpixelbuffergetplanecount?language=objc "CVPixelBufferGetPlaneCount(...)")：返回 Pixel Buffer 中 Plane 的数量。
- [CVPixelBufferGetBytesPerRowOfPlane(...)](https://developer.apple.com/documentation/corevideo/1456711-cvpixelbuffergetbytesperrowofpla?language=objc "CVPixelBufferGetBytesPerRowOfPlane(...)")：返回指定 Index 的 Plane 的每行字节数。
- [CVPixelBufferGetHeightOfPlane(...)](https://developer.apple.com/documentation/corevideo/1456698-cvpixelbuffergetheightofplane?language=objc "CVPixelBufferGetHeightOfPlane(...)")：返回指定 Index 的 Plane 的高度。非 Planar 类型，返回 0。
- [CVPixelBufferGetWidthOfPlane(...)](https://developer.apple.com/documentation/corevideo/1456830-cvpixelbuffergetwidthofplane?language=objc "CVPixelBufferGetWidthOfPlane(...)")：返回指定 Index 的 Plane 的宽度。非 Planar 类型，返回 0。

下面是一个获取 pixelBuffer（nv12、480x720）的 byte size 的示例代码：

```c
int pixelWidth = (int)CVPixelBufferGetWidth(pixelBuffer);   // 480
int pixelHeight = (int)CVPixelBufferGetHeight(pixelBuffer); // 720
                
CVPixelBufferLockBaseAddress(pixelBuffer, 0); 

__unused uint8_t *y_frame = (unsigned char *)CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, 0);
__unused uint8_t *uv_frame = (unsigned char *)CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, 1);

size_t plane1_stride = CVPixelBufferGetBytesPerRowOfPlane(pixelBuffer, 0); // 512
size_t plane2_stride = CVPixelBufferGetBytesPerRowOfPlane(pixelBuffer, 1); // 512
size_t y_height = CVPixelBufferGetHeightOfPlane(pixelBuffer, 0);  // 720
size_t plane1_size = plane1_stride * y_height;
size_t uv_height = CVPixelBufferGetHeightOfPlane(pixelBuffer, 1); // 360
size_t plane2_size = plane2_stride * uv_height;
size_t frame_size = plane1_size + plane2_size;

CVPixelBufferUnlockBaseAddress(pixelBuffer, 0);
```


下面是将一个 pixelBuffer 转换成 UIImage 的示例代码：

```c
+ (UIImage*)imageFromPixelBuffer:(CVPixelBufferRef)pixelBuffer {
        CIImage* ciImage = [CIImage imageWithCVPixelBuffer:pixelBuffer];
        CIContext* context = [CIContext contextWithOptions:@{kCIContextUseSoftwareRenderer : @(YES)}];
        CGRect rect = CGRectMake(0, 0, CVPixelBufferGetWidth(pixelBuffer), CVPixelBufferGetHeight(pixelBuffer));
        CGImageRef videoImage = [context createCGImage:ciImage fromRect:rect];
        UIImage* image = [UIImage imageWithCGImage:videoImage];
        CGImageRelease(videoImage);
        return image;
}
```


**3）CVPixelBufferRef 的一些使用注意项**

- 访问 CVPixelBufferRef 内存时，需要用 `CVPixelBufferLockBaseAddress` 和 `CVPixelBufferUnlockBaseAddress` 包起来进行加锁，否则可能取不到，返回 NULL。
- Apple 在兼容一些特殊情况时会对 CVPixelBufferRef 的内存进行扩充，一般是在 width 上，也就是每一行增加 8-32 个字节，用 0 来填充，yuv 的 0 转换为 rgb 正好是绿色，如果处理不当时就会出现绿边。正确的获取数据的方式是：通过 `CVPixelBufferGetWidth` 获取有效像素的宽度，通过 `CVPixelBufferGetBytesPerRowOfPlane`、`CVPixelBufferGetBytesPerRow` 获取有效像素宽度和扩充的宽度。
- 合理使用 CVPixelBufferRef 实现 CPU 与 GPU 共享内存。iPhone 的 CPU 对于处理视频来说能力是非常有限的，所以在 Apple 开发中，如果要进行视频处理，比如滤镜、美颜等，都会用到设备的 GPU 能力，也就是会用到 OpenGL ES 的 API，而 CPU 和 GPU 之间的数据传递效率十分低下，尤其是从 GPU 回传数据到 CPU，更是缓慢。鉴于此，Apple 就有了 GPU 和 CPU 的共享内存机制，从而避免数据的拷贝。满足某种条件的 CVPixelBufferRef 本身就是共享内存，这个条件就 CVPixelBufferRef 具有 `kCVPixelBufferIOSurfacePropertiesKey` 属性，从 Camera 采集出来和从 VideoToolBox 硬解出来的 pixelBuffer 是具有这个属性，也就是这些 pixelBuffer 可以在 CPU 和 GPU 之间共享。
- 当处理纹理时，可以使用 CVOpenGLESTextureCacheRef 纹理缓冲池。使用纹理缓冲池可以让我们每次生产的 texture 都是从缓存获取的，从而省掉反复创建 texture 的开销，这个机制目前不支持模拟器，只支持真机。下面是相关 API 解释和示例：

```c
// 1、系统将 texture 用 CVOpenGLESTextureRef 进行封装

CVOpenGLESTextureRef textureRef;
GLuint textureId = CVOpenGLESTextureGetName(textureRef); // texture id
GLenum textureEnum = CVOpenGLESTextureGetTarget(textureRef); // GL_TEXTURE_2D
// glBindTexture(textureEnum , textureId);


// 2、CVOpenGLESTextureCacheRef 是对多个 CVOpenGLESTextureRef 的管理

// 创建
CVOpenGLESTextureCacheRef _textureCache;
CVReturn err = CVOpenGLESTextureCacheCreate(kCFAllocatorDefault, NULL, _eaglContext, NULL, &_textureCache);
if (err != kCVReturnSuccess) NSLog(@"CVOpenGLESTextureCacheCreate fail %d", result);

// 清理
// 当每次调用 CVOpenGLESTextureCacheCreateTextureFromImage 这个方法获取 textureRef 时，纹理缓存会自动刷新当前未使用的资源，也可以显式调用 flush 方法来刷新缓存
CVOpenGLESTextureCacheFlush(_textureCache, 0); // 第 2 个参数是保留位


// 3、通过 CVOpenGLESTextureCacheCreateTextureFromImage 获取 CVOpenGLESTextureRef

CVOpenGLESTextureCacheCreateTextureFromImage(
    <#CFAllocatorRef  _Nullable allocator#>, // kCFAllocatorDefault 用默认就好
    <#CVOpenGLESTextureCacheRef  _Nonnull textureCache#>, // texture cache
    <#CVImageBufferRef  _Nonnull sourceImage#>, // renderTarget
    <#CFDictionaryRef  _Nullable textureAttributes#>, // NULL
    <#GLenum target#>, // GL_TEXTURE_2D
    <#GLint internalFormat#>, // opengl format，相当于 glTexImage2D() 函数第三个参数，希望把纹理储存为何种格式
    <#GLsizei width#>,
    <#GLsizei height#>,
    <#GLenum format#>, // native iOS format，相当于 glTexImage2D() 函数倒数第三个参数，这里即renderTarget(pixelBuffer) 的像素格式，这里是 iOS 系统默认的 BGRA 数据格式
    <#GLenum type#>, // 相当于glTexImage2D() 函数第二个参数
    <#size_t planeIndex#>, // 对于 planner 存储方式的像素数据，这里填写对应的索引。非 planner 格式写 0 即可，非 planner 时系统会忽略此参数
    <#CVOpenGLESTextureRef _Nullable * _Nonnull textureOut#>); // 生成的纹理

// glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, size.width, size.height, 0, GL_RGBA, GL_UNSIGNED_BYTE, imageData);    
```







## 2、聊聊对音视频同步的理解？



音视频对齐方式有三种：

- 以音频时钟为基准
- 以视频时钟为基准
- 以第三方时钟为基准

由于音频播放都是硬件来驱动的，相对比较稳定，另外音频 pts 通常是单调递增的，所以一般是按照音视时钟为准。

以一个 44.1KHz 的 AAC 音频流和 24 FPS 的视频流为例，理想情况下，音视频完全同步，音视频播放过程如下图所示：

![音视频同步](assets/resource/av-interview-qa/av_clock.jpg)

但实际情况下，如果用上面那种简单的方式，慢慢的就会出现音视频不同步的情况，主要原因是同步时，时间粒度太大了，难以精准控制。所以需要引入一个参考时钟（要求参考时钟上的时间是恒定线性递增的）来提高音频时钟的时间粒度，比如：系统时间，进而进行精准时钟对齐。最后以音频时钟为准，视频放快了就减慢播放速度，播放快了就丢帧或加快播放的速度。




---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_





