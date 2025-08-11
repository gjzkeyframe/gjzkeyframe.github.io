---
title: 音视频面试题集锦第 10 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:08:08 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦,  面试, 音视频]
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

我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。

下面是 2023.11 月音视频面试题集锦的几条干货精选：


- **1、介绍一下 FFmpeg 中关于 timebase 的基础知识与应用？**
- **2、如何识别一个视频是 HDR 视频？**
- **3、如何通过优化播放器来优化音乐播放体验，比如提升音质或音效？**
- **4、介绍一下 SIMD 以及它在音视频处理中的应用？**




## 1、介绍一下 FFmpeg 中关于 timebase 的基础知识与应用？

**1）timebase 定义**

在 FFmpeg 中，`time_base` 是一个关键概念，它用于表示时间单位。在处理音频或视频流时，`time_base` 可以根据不同的采样频率或帧率来定义。`timebase` 在 FFmpeg 的定义是一个 `AVRational` 结构体：

```c
typedef struct AVRational{
    int num; ///< numerator  
    int den; ///< denominator  
} AVRational;
```


**2）timebase 的使用**

在某些情况下，`time_base` 是根据采样频率来定义的。例如：对于视频采样频率为 90KHz（90000Hz）的情况，`time_base` 就相当于 1/90000 秒。另一种定义 `time_base` 的方式是根据帧率。例如：对于视频帧率为 24fps 的情况，`time_base` 就相当于 1/24 秒。在 FFmpeg 的分层结构中，原始数据层、编解码层和封装层都有对应的 `time_base`。原始数据层和封装层都通过 `AVStream` 进行处理，而编解码层则对应 `AVCodec`。


**3）封装层 timebase，视频流/音频流 timebase 和现实时间戳的的关系和转换**

封装层 tbn、视频 tbc 和音频 tbc 可以各不相同，相互不影响。现实时间基我们一般选用 1us 即 (1/1000000)s。因为每一层用的时间基不同，在函数参数传递上只会使用时间基前面的倍数值，`timebase` 是统一的，因此时间在不同的时间基上面需要做一层转换。 例如：现实时间 1s 转换到音频流时间实现为 `1000000 * (1/1000000) = 44100 * (1/44100)`，那么现实时间 1000000 在音频流时间值则为 44100。举一个开发中的实例：如果想 seek 视频到现实时间的 X ms。

```c
int64_t seekTime = (int64_t)(( X / 1000 )  / av_q2d(videoStream->time_base)); 
av_seek_frame(videoFormatCtx_, video_index_, seekTime, AVSEEK_FLAG_BACKWARD);
```

因为 `av_seek_frame` 是在视频流层面，时间基与现实时间不同，需要转换并将转换后的值作为参数才能得到正确的结果。

**4）转换函数解析**


```c
double av_q2d(AVRational a) //将AVRational 对象转换为小数，便于转换
// 将一个时间戳a从时基bq转换到时基cq下
int64_t av_rescale_q(int64_t a, AVRational bq, AVRational cq)
```

例如，将视频流的一帧 `pts(a * atbr)` 转换到封装层打包成 `AVPacket`，封装层 `timebase` 为 tbn，此时需要转换 `int64 t = av_rescale_q_rnd(a， atbr， tbn);`。




## 2、如何识别一个视频是 HDR 视频？

iOS 判断一个视频是否是 HDR 视频的方法：判断是否带有 HDR 特征的 track 即可，如下：

```c
NSArray<AVAssetTrack *> *hdrTracks = 
[asset tracksWithMediaCharacteristic:AVMediaCharacteristicContainsHDRVideo];
if (hdrTracks.count > 0){
   return YES;
}
```

Android 需要我们自己解析出 `colortransforfunction和ccolorStandard`，如下：

```c
@RequiresApi(api = Build.VERSION_CODES.R)
public static boolean isHDR(MediaMetadataRetriever mediaMetadataRetriever)
        throws NumberFormatException {
    String colorTransferString =
            mediaMetadataRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_COLOR_TRANSFER);
    Log.e("isHDR", colorTransferString);
    String colorStandardString =
            mediaMetadataRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_COLOR_STANDARD);
    Log.e("isHDR", colorStandardString);
    int colorTransfer = Integer.parseInt(colorTransferString);
    int colorStandard = Integer.parseInt(colorStandardString);
    // This check needs to match the isHDR check in
    // frameworks/av/media/libstagefright/FrameDecoder.cpp.
    return (colorTransfer == MediaFormat.COLOR_TRANSFER_HLG
            || colorTransfer == MediaFormat.COLOR_TRANSFER_ST2084)
            && colorStandard == MediaFormat.COLOR_STANDARD_BT2020;
}
```

## 3、如何通过优化播放器来优化音乐播放体验，比如提升音质或音效？


在播放侧可以使用自动增益控制算法（AGC）来提升音效。AGC 算法通过自动调整音频信号的增益，使其保持在一定的范围内，这种算法可以避免因音频信号的幅度变化而引起的声音过大或过小的问题，保证了音频信号的稳定性和可听性，目前有开源的实现例如 `webrtcagc`，可以把算法移植到自己的项目中。



## 4、介绍一下 SIMD 以及它在音视频处理中的应用？


SIMD（Single Instruction Multiple Data）是一种并行计算的技术，它允许在单个指令中同时处理多个数据元素。SIMD 指令集通常由处理器提供，用于加速向量化计算，从而提高程序的性能。

下面是一个 SIMD 的示例：向量化乘法

假设有两个数组 A 和 B，我们想要将它们的对应元素相乘，并将结果存储在另一个数组 C 中，使用 SIMD 指令，可以一次处理多个元素，提高计算效率。

```c
// 使用 SIMD 指令进行向量化乘法
#include <immintrin.h>

void vectorMultiply(float* A, float* B, float* C, int size) {
    for (int i = 0; i < size; i += 8) {
        __m256 a = _mm256_load_ps(A + i); // 加载 8 个单精度浮点数到向量寄存器 A
        __m256 b = _mm256_load_ps(B + i); // 加载 8 个单精度浮点数到向量寄存器 B
        __m256 result = _mm256_mul_ps(a, b); // 执行向量乘法
        _mm256_store_ps(C + i, result); // 存储结果到数组 C
    }
}
```

在实际应用中，还可以使用 SIMD 指令进行其他操作，如减法、除法、逻辑运算等，以及应用于不同的数据类型，如整数、双精度浮点数等。通过合理地使用 SIMD 优化，可以显著提高程序的性能。

在音视频开发中，SIMD 也有不少的应用场景。比如：

1）在音频处理中，SIMD 可以用于实时音频效果处理，如均衡器、压缩器、混响器等，通过同时处理多个音频样本，可以提高音频处理的效率和实时性。

2）在视频处理中，SIMD 可以用于加速图像处理算法，如图像滤波、边缘检测、图像压缩等，通过同时处理多个像素，可以提高图像处理的速度和质量。

3）在视频编码中，SIMD 可以用于加速压缩和解压算法，如 H.264、H.265 编码器一些实现中，可以通过并行处理视频数据来提高视频编解码的效率和性能。

总之，SIMD 在音视频开发中的合理应用可以提高数据处理速度，降低功耗。



---


**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_





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

