---
title: 音视频面试题集锦第 13 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:38:08 +0800
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


下面是第 13 期面试题精选：


- **1、AVPlayer 中如何实现视频片段加速预览播放？**
- **2、如何高效获取一个视频的关键帧序列？**
- **3、SPS 和 PPS 在 extradata 中的作用是什么？**
- **4、I 帧和 IDR 帧有什么区别？在什么情况下 I 帧不是 IDR 帧？**


## 1、AVPlayer 中如何实现视频片段加速预览播放？

在编辑场景用 AVPlayer 来实现预览播放器时，对视频中某一段内容进行加速播放的实现代码如下：

```objc

// 创建 AVMutableComposition 对象
AVMutableComposition *composition = [AVMutableComposition composition];
// 将视频文件加载到 AVURLAsset 对象中
NSURL *videoURL = [[NSBundle mainBundle] URLForResource:@"your_video" withExtension:@"mp4"];
AVURLAsset *videoAsset = [AVURLAsset URLAssetWithURL:videoURL options:nil];
// 将视频的前 3 秒进行加速处理
CMTime startTime = kCMTimeZero;
CMTime duration = CMTimeMake(3, 1); // 加速的时间范围为前 3 秒
CMTimeRange timeRange = CMTimeRangeMake(startTime, duration);
[composition scaleTimeRange:timeRange toDuration:CMTimeMake(1, 1)]; 
// 将时间范围加速到 1 秒
// 创建 AVPlayerItem 对象并将组合后的视频添加到其中
AVPlayerItem *playerItem = [AVPlayerItem playerItemWithAsset:composition];
// 创建 AVPlayer 对象并将 AVPlayerItem 对象添加到其中
AVPlayer *player = [AVPlayer playerWithPlayerItem:playerItem];
```


## 2、如何高效获取一个视频的关键帧序列？ 

获取一个视频的关键帧序列，基于 Android 平台 API 实现：

```java
MediaExtractor extractor = new MediaExtractor();
extractor.setDataSource(getVideoPath());

int trackIndex = MediaExtractorUtil.selectVideoTrack(extractor);
extractor.selectTrack(trackIndex);

List<Long> keyframeTimestampsMS = new ArrayList<Long>();

while (extractor.getSampleTime() != -1) {
    long sampleTime = extractor.getSampleTime();

    if ((extractor.getSampleFlags() & MediaExtractor.SAMPLE_FLAG_SYNC) > 0) {
        keyframeTimestampsMS.add(sampleTime / 1000);
    }
    // 此处表示 extractor seek 的间隔为 1000 微妙
    extractor.seekTo(sampleTime + 1000, MediaExtractor.SEEK_TO_NEXT_SYNC);
}
```

获取一个视频的关键帧序列，基于 FFmpeg 实现：


```C++
#include "libavcodec/avcodec.h"  
#include "libavutil/ff_time.h"  
#include "libavformat/avformat.h"  
#include "libavformat/avc.h"

AVFormatContext* formatCtx = avformat_alloc_context();  
avformat_open_input(&formatCtx, path.c_str(), NULL, NULL);  
int videoIndex = -1;  
  
for (int i = 0; i < formatCtx->nb_streams; i++) {  
    AVStream* stream = formatCtx->streams[i];  
    AVMediaType type = stream->codecpar->codec_type;  
    if (type == AVMEDIA_TYPE_VIDEO) {  
        videoIndex = i;  
        break;  
    }
}  
if (videoIndex > 0) {
    AVStream* videoStream = formatCtx->streams[videoIndex];  
    AVInputFormat* iformat = formatCtx->iformat;  
    if (strcmp(iformat->name, "mov,mp4,m4a,3gp,3g2,mj2") == 0) {  
        std::vector<int64_t> keyframe_time_list_tmp;  
        MOVStreamContext* sc = (MOVStreamContext*) videoStream->priv_data;  
        for (int videoIndex = 0; videoIndex < videoStream->nb_index_entries; videoIndex++) {  
            AVIndexEntry indexEntry = videoStream->index_entries[videoIndex];  
            if (indexEntry.flags & AVINDEX_KEYFRAME) {  
                MOVStts cttsData = {0};  
                if (sc && sc->ctts_count == videoStream->nb_index_entries) {  
                    cttsData = sc->ctts_data[videoIndex];  
                } 
                double doublePts = (indexEntry.timestamp + sc->dts_shift + cttsData.duration) * av_q2d(videoStream->time_base) * 1000.0;  
                int64_t ptsTime = ceil(doublePts);  
                keyframe_time_list_tmp.push_back(ptsTime);  
            }    
        }  
    }
}
```


## 3、SPS 和 PPS 在 extradata 中的作用是什么？

SPS（Sequence Parameter Set）和 PPS（Picture Parameter Set）是 H.264 视频编码中的两种重要参数集。它们包含了视频序列的特性和参数信息，对于解码器来说非常重要。

SPS 包含了视频序列的全局参数，如分辨率、帧率、颜色空间等。PPS 则包含了与特定图像相关的参数，如切片组的配置、参考帧的使用等。

在 extradata 中，SPS 和 PPS 的作用是为解码器提供视频序列的配置信息，以确保解码器能够正确地解释和处理视频数据。通过提供这些参数集，解码器能够准确地还原视频序列的特性，从而实现高质量的视频解码。

  

## 4、I 帧和 IDR 帧有什么区别？在什么情况下 I 帧不是 IDR 帧？

I 帧：I 帧是视频序列中的关键帧，它是一个完整的图像帧，类似于 JPEG 或 BMP 图像文件。I 帧不依赖于其他帧，因此可以独立解码和显示。在视频序列中，I 帧通常用于随机访问点，也作为其他帧解码的参考。

IDR 帧：IDR 帧是一种特殊的 I 帧，它具有刷新解码器缓冲区的功能。当解码器接收到 IDR 帧时，它会清除之前的解码状态，确保从该帧开始解码，从而避免错误传播。IDR 帧通常用于视频序列的随机访问点，以及在视频传输或存储中用于错误恢复。

因此 IDR 帧一定是 I 帧，但是 I 帧则不一定是 IDR 帧。在遇到 OpenGOP 的情况下，就会出现 I 帧为非 IDR 帧的情况。

![OpenGOP](assets/resource/av-interview-qa/opengop.wep)

如上图所示右数第一个 I 帧就是一个非 IDR 的 I 帧，前一个 GOP 中的 B 帧依赖了当前 GOP 的 I 帧。所以右数第一个 I 帧接受时，不能刷新解码器，否则上一个 GOP 中的 B 帧无法被成功解码，可能会出现花屏或者报错。





---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_



