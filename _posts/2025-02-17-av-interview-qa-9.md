---
title: 音视频面试题集锦第 9 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 02:28:08 +0800
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

下面是 2023.10 月音视频面试题集锦的几条干货精选：

- **1、iOS 如何实现 HDR 转 SDR？**
- **2、iOS 如何支持封装 FMP4 格式？**
- **3、FFmpeg 如何支持封装 FMP4 格式？**
- **4、转码速度优化的几点建议？**
- **5、Seek 优化的几点建议？**

## 1、iOS 如何实现 HDR 转 SDR？

`HDR` 与 `SDR` 的基础知识以及如何使用 `FFmpeg` 实现转换可以参考我们之前的文章：[如何正确将 HDR 视频转换成 SDR 视频](https://mp.weixin.qq.com/s/rzxGv-KwNgrgI_XlBcaUKw)，下面我们主要介绍一下在 iOS 平台使用系统库来的完成 HDR 转 SDR 的具体方案。

**1）iOS 如何判断是 HDR 视频**

```objc
- (BOOL)isHDR:(CMVideoFormatDescriptionRef)description {
    CFDictionaryRef descriptionExtensions = CMFormatDescriptionGetExtensions(description);
    if(descriptionExtensions){
        NSDictionary * ocDescriptionExtensions = (__bridge  NSDictionary*) descriptionExtensions;
        NSString* bufferYCbCrMatrix = ocDescriptionExtensions[@"CVImageBufferYCbCrMatrix"];
        return [bufferYCbCrMatrix isEqualToString:(__bridge NSString*)kCVImageBufferYCbCrMatrix_ITU_R_2020];
    }
    return NO;
}
```

**2）iOS 使用 AVVideoComposition 将 HDR 转换为 SDR**

使用 `AVVideoComposition` 参数设置，可以支持系统 AVPlayer、AVAssetReader、AVAssetImageGenerator，将 `AVVideoComposition` 设置给 `AVComposition` 做为输入参数即可。

```objc
- (AVMutableVideoComposition*) buildVideoCompositon {
    AVMutableVideoComposition *videoComposition = [AVMutableVideoComposition videoComposition];
    videoComposition.frameDuration = CMTimeMake(1, 30);
    videoComposition.renderSize = CGSizeMake(720,1280);
    if(isHDR){
        videoComposition.colorPrimaries = AVVideoColorPrimaries_ITU_R_709_2;//默认转换为 BT709
        videoComposition.colorTransferFunction = AVVideoTransferFunction_ITU_R_709_2;
        videoComposition.colorYCbCrMatrix = AVVideoYCbCrMatrix_ITU_R_601_4;
    }
    return videoComposition;
}
```

**3）iOS 使用 VideoToolBox HDR 转换为 SDR**

使用 `VideoToolBox` 参数设置，可以支持系统硬件解码输出为 `SDR` 的 `CVPixelBuffer`。

```objc
- (void)vtbHDR2SDR {
    CFMutableDictionaryRef pixelTransferProperty = CFDictionaryCreateMutable(
           kCFAllocatorDefault, 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
    CFDictionarySetValue(pixelTransferProperty,
                                @"DestinationColorPrimaries",
                                kCVImageBufferColorPrimaries_ITU_R_709_2);
    CFDictionarySetValue(pixelTransferProperty,
                                @"DestinationTransferFunction",
                                kCVImageBufferTransferFunction_ITU_R_709_2);
    CFDictionarySetValue(pixelTransferProperty,
                                @"DestinationYCbCrMatrix",
                                kCVImageBufferYCbCrMatrix_ITU_R_709_2);
    VTSessionSetProperty(session, kVTDecompressionPropertyKey_PixelTransferProperties,
                         pixelTransferProperty);
    CFRelease(pixelTransferProperty);
}
```

## 2、iOS 如何支持封装 FMP4 格式？

**1）FMP4 与 MP4 很相似，区别如下**

- `FMP4` 不需要一个 `moov Box` 来进行 `Initialization`，`FMP4` 的 `moov Box` 只包含了一些 Track 信息。
- `FMP4` 的视频/音频 metadata 信息与数据都存在一个个 `moof`、`mdat` 中，它是一个流式的封装格式。

![FMP4](assets/resource/av-interview-qa/task-interview-qa-collection_fmp4.jpg)

**2）FMP4 应用场景如下**

- 流式转码与上传 
- 流式播放 （`Dash` 支持，内部数据为 `FMP4`）

**3）iOS 封装 FMP4 如何实现**

`AVAssetWriter` 设置以下参数即可，系统会按照设置的切片时长回调 `didOutputSegmentData` 返回数据，仅 iOS 14 系统以上才可以。

```objc
- (AVAssetWriter*)createFMP4Writer {
    AVAssetWriter *writer = [[AVAssetWriter alloc] initWithContentType:[UTType typeWithIdentifier:AVFileTypeMPEG4]];
    writer.outputFileTypeProfile = AVFileTypeProfileMPEG4AppleHLS;
    writer.preferredOutputSegmentInterval = CMTimeMake(10.0, 1);//10.0自定义切片时长
    writer.initialSegmentStartTime = kCMTimeZero;
    writer.delegate = self;
    return writer;
}

- (void)assetWriter:(AVAssetWriter *)writer didOutputSegmentData:(NSData *)segmentData segmentType:(AVAssetSegmentType)segmentType segmentReport:(nullable AVAssetSegmentReport *)segmentReport {
    //存储segmentData 即可
}
```


## 3、FFmpeg 如何支持封装 FMP4 格式？

`FMP4` 数据格式参考第 2 题，`FFmpeg` 实现 `FMP4` 需要通过 `Muxer` 配置参数，设置 `avio_alloc_context` 回调数据，具体分片通过控制 `av_write_frame(formatContext, NULL)` 调用时机执行相关策略。

```c
void muxerInit {
    av_register_all();
    uint8_t databuf[5 * 1024 * 1024] = {};
    AVFormatContext *formatContext = avformat_alloc_context();
    formatContext->oformat = av_guess_format("mp4", NULL, NULL);
    formatContext->pb = avio_alloc_context(databuf, sizeof(databuf), AVIO_FLAG_WRITE, NULL, NULL, fmp4Write, NULL); // 设置输出回调
    formatContext->opaque = formatContext->pb->opaque = this;
    formatContext->flags |= AVFMT_FLAG_BITEXACT;
    
    AVDictionary*  dict = NULL;
    av_dict_set(&dict, "movflags", "frag_custom+empty_moov+dash+frag_discont", 0); // 开启FMP4
    avformat_write_header(formatContext, &dict);
}

static int fmp4Write(void* opaque, uint8_t* buf, int size) {
    // 存储数据
}

void processInputPacket(AVPacket *pkt){
    av_write_frame(formatContext, NULL); //通过关键帧以及到指定时长触发此方法进行切片
}
```

## 4、转码速度优化的几点建议？

**1）解码**

- 优先使用系统硬件解码，软件解码仅兜底使用。
- 解码方式优先使用异步。
- 解码器可以创建复用池。
- Android 解码优先考虑 `Surface` 解码方式。

**2）多线程并发**

- 转码分为解码与编码，通常编码更加耗时，这样解码线程可保持几帧 `Cache` 数据供编码获取。
- 同一个视频可以分片转码为 `FMP4` ，最终拼接为一个 `FMP4`，并发数目设置参考 2-5。

**3）其它**

- 码率不高的视频无需转码。
- 对于帧率过高、码率过高视频可降低相关参数，提高转码速度。
- 视频转码与发布可结合优化策略，例如一边分片转码一边上传。
- 针对不同场景进行指定策略处理，例如添加了音乐仅转码音频，仅转换封装格式使用 `Remuxer`。


## 5、Seek 优化的几点建议？

**1）解码**

- 优先使用系统硬件解码，软件解码仅兜底使用。
- 解码方式优先使用异步。
- 解码器可以创建复用池。
- Android 解码优先考虑 `Surface` 。
- 对于 Seek 过程中的帧可以不输出数据，例如 iOS `kVTDecodeFrame_DoNotOutputFrame`。

**2）其它**

- 对于 Seek 过程中的帧可以丢弃非参考帧。
- 优先判断 Seek 帧是否命中缓存。
- 优先找到 Seek 帧最近的关键帧。
- 对于向右侧 Seek 优先判断是否与当前解码帧同一个 `GOP`，同一个 `GOP` 无需 `Flush`，继续解码即可。
- 对于 Seek 位置精准度可以给一点空隙，例如 100ms 内偏差用户无感，这样可以利用缓存优势以及最近关键帧。
- 对于 Seek 超级频繁可以选择丢弃某些帧。


---


**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_