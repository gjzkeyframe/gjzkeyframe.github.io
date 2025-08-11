---
title: 音视频面试题集锦第 5 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 01:58:08 +0800
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

下面是 2023.08 月音视频面试题集锦的几条干货精选：


- **1、点播的倍速播放要如何实现？**
- **2、视频编辑中如何实现视频倒放？**
- **3、播放器解码后的帧缓冲区一般设置多大合适？**
- **4、如何监控视频播放黑屏、花屏、绿屏等异常？**






## 1、点播的倍速播放要如何实现？


点播的倍速播放分为视频处理和音频处理部分。

**1）视频处理**

对应视频数据的处理，核心逻辑就是按照倍速重新计算各视频帧的 pts 时间戳。

比如，对一个视频做 2 倍速播放，假设原来各视频帧的 pts 依次是 `0, 30, 60, 90 ...`，倍速处理及将它们除以 2 变成 `0, 15, 30, 45 ...`。这样处理后，视频的帧率和总时长相应的也发生了变化，帧率变为原来的 2 倍，总时长变为原来的 1/2。

但是，如果对视频进行高倍速播放，比如 10 倍速，这时候如果只处理 pts，原视频的时间戳除以 10 变成 `0, 3, 6, 9 ...`，这时候 3ms 一帧，帧率达到了 333fps，已经超过了屏幕硬件的刷新率，根本渲染不过来。所以对于高倍速播放：第一步，我们像上面一样在处理完 pts；第二步，我们还需要设定最大帧率的限制，并按照这个最大帧率来进行丢帧。

假如我们设定最大帧率是 60fps，这时候我们每 17ms 只需要一帧，上面的 `0, 3, 6, 9, 12, 15, 18 ...` 经过丢帧和帧率处理可能就变成了 `0, 17(18→17), 34(33→34) ...`。

高倍速播放还有另外的问题：解码性能是否跟得上、网络视频的下载速度是否跟得上等等。对于这些问题，我们可能还需要其他方案来解决，比如：在客户端在解码前就要丢弃非参考帧，对不需要解码的帧直接丢弃等等；在服务端对高倍速视频进行预处理，提前做好时间戳和丢帧处理，当用户切换高倍速时，帮用户切换资源即可。

**2）音频处理**

音频是每秒几 K 到几十 K 的采样点，对于音频数据的处理，如果只是跟视频一样，简单的处理 pts 时间戳会出现噪音、杂音，体验很差。音频一般需要进行重采样处理。比如，原来的音频是 48K 的采样率，播放设置了使用 48K 的采样率进行音频渲染，这时候要对音频做 2 倍速播放，可以将音频数据每秒 48K 个采样点重采样降低到 24K 个（把音频数据的采样率处理为 24K），当播放还是使用 48K 的采样率来播放时，每秒需要 48K 个采样点，这时候就需要 2s 的数据，这时候音频的播放速度就变成了原来的 2 倍。使用这样的方式来实现音频倍速，可以解决只简单处理 pts 带来的噪音、杂音问题，但是音频的播放会变调：快播时，声调会变高，听起来尖细；慢播时，声调会变低，听起来低沉。

要实现音频倍速变速不变调，可以使用第三方库 SoundTouch 来实现。相关代码可以研究一下：

- [SoundTouch 官网](https://www.surina.net/soundtouch/ "SoundTouch 官网")
- [B 站 fork 的 SoundTouch 代码](https://github.com/bilibili/soundtouch "B 站 fork 的 SoundTouch 代码")




<!--  

视频倍速播放
https://www.bilibili.com/video/BV1AX4y1i73v/

音频倍速播放
https://www.bilibili.com/video/BV1hh4y1775k/

-->



## 2、视频编辑中如何实现视频倒放？

如果用最直接的思路去实现视频倒放，那就是把视频中的每一帧图像都解码出来逆排序一下，然后将原视频的 pts 时间戳一一对应的关联上逆排序后的每一帧，再重新编码就可以了。

这个思路在实际实现时会有几个问题：解码后的视频帧放在内存或磁盘可能都还挺大的；完整的解码、逆排序、编码一整个视频可能耗时较长。

对于这些问题我们可以采取分而治之、并行提效的思路。因为视频本身可以按照 GOP 单元独立编解码，所以我们可以把视频的每一个 GOP 单元取出来分别做解码、逆排序、编码，最后再把处理后的所有 GOP 重新 remux 封装起来即可。

其中逆排序过程中，对于一个 GOP 的各帧处理流程大致是这样的：比如一个 GOP 的各视频帧及对应的 pts 分别是 `1(0), 2(30), 3(60), 4(90)`，那逆排序后就是 `4(0), 3(30), 2(60), 1(90)`。










## 3、播放器解码后的帧缓冲区一般设置多大合适？

对于播放器来说，在渲染之前一般会有一个解码后的帧缓冲区用于后续渲染。

使用 FFmpeg 软解码、Android MediaCodec 硬解码时，给编码器的数据是按照 dts 顺序输入的，解码器输出数据是按照 pts 输出的。但是使用 iOS VideoToolbox 硬解码时，解码器输出数据并没有按照 pts 顺序，而是解一帧出一帧，需要我们自己排序。

这样在使用 iOS VideoToolbox 硬解码时，还可以在这个缓冲区还可以用来对解码后的帧按 pts 做重排来保证正确的渲染顺序。

这个重排主要是针对 B 帧，因为 B 帧可能会依赖后面的帧才能完成解码，如果解码后的帧缓冲区太小，可能导致按照渲染顺序本该渲染的 B 帧由于依赖帧还未解码而出现播放卡顿。但是解码后的帧缓冲区也不是越大越好，因为解码后的视频帧数据是比较大的，会占用不少内存，缓冲区过大会造成播放器的内存占用过大。

那么这个缓冲区应该设置多大比较合适呢？这个其实取决于解码器需要的重排窗口大小，解码后的帧缓冲区大小只要不超过这个重排窗口尺寸即可。要计算重排窗口的大小通常可能会用到下面这几个参数：

- `max_ref_frames`
- `max_num_reorder_frames`

下面是一些开源项目里的实现方案：

**1）IJKPlayer**

IJKPlayer 计算重排窗口大小的实现方案是在保证最小为 2、最大为 5 的限制下来使用 `sps.max_ref_frames` 作为缓冲区大小。

```c
fmt_desc->max_ref_frames = FFMAX(fmt_desc->max_ref_frames, 2);
fmt_desc->max_ref_frames = FFMIN(fmt_desc->max_ref_frames, 5);
```

相关代码见：

- [IJKVideoToolBoxAsync.m](https://github.com/bilibili/ijkplayer/blob/30eb9441945da795079492041a791c121d2b8206/ios/IJKMediaPlayer/IJKMediaPlayer/ijkmedia/ijkplayer/ios/pipeline/IJKVideoToolBoxAsync.m#L1153C1-L1155C1 "IJKVideoToolBoxAsync.m")
- [h264_sps_parser.h](https://github.com/bilibili/ijkplayer/blob/cced91e3ae3730f5c63f3605b00d25eafcf5b97b/ios/IJKMediaPlayer/IJKMediaPlayer/ijkmedia/ijkplayer/ios/pipeline/h264_sps_parser.h#L267 "IJKVideoToolBoxAsync.m")


**2）VLC**

VLC 计算重排窗口大小的实现方式如下：

- 1）根据 AVC E.2.1 标准尝试取值 `max_num_reorder_frames`；
- 2）判断如果是特定的 profile，就不需要排序，设置 `max_num_reorder_frames` 为 0 并返回，表示不用重排；
- 3）如果 profile 都不符合，则根据 `sps` 中的 `level` 限制计算 `max_dpb_frames` 并返回，`max_dpb_frames` 默认为 16。

这里的 `dpb` 是 `decoded picture buffer` 的缩写。


```c
static uint8_t h264_get_max_dpb_frames( const h264_sequence_parameter_set_t *p_sps )
{
    const h264_level_limits_t *limits = h264_get_level_limits( p_sps );
    if( limits )
    {
        unsigned i_frame_height_in_mbs = ( p_sps->pic_height_in_map_units_minus1 + 1 ) *
                                         ( 2 - p_sps->frame_mbs_only_flag );
        unsigned i_den = ( p_sps->pic_width_in_mbs_minus1 + 1 ) * i_frame_height_in_mbs;
        uint8_t i_max_dpb_frames = limits->i_max_dpb_mbs / i_den;
        if( i_max_dpb_frames < 16 )
            return i_max_dpb_frames;
    }
    return 16;
}

bool h264_get_dpb_values( const h264_sequence_parameter_set_t *p_sps,
                          uint8_t *pi_depth, unsigned *pi_delay )
{
    uint8_t i_max_num_reorder_frames = p_sps->vui.i_max_num_reorder_frames;
    if( !p_sps->vui.b_bitstream_restriction_flag )
    {
        switch( p_sps->i_profile ) /* E-2.1 */
        {
            case PROFILE_H264_BASELINE:
                i_max_num_reorder_frames = 0; /* only I & P */
                break;
            case PROFILE_H264_CAVLC_INTRA:
            case PROFILE_H264_SVC_HIGH:
            case PROFILE_H264_HIGH:
            case PROFILE_H264_HIGH_10:
            case PROFILE_H264_HIGH_422:
            case PROFILE_H264_HIGH_444_PREDICTIVE:
                if( p_sps->i_constraint_set_flags & H264_CONSTRAINT_SET_FLAG(3) )
                {
                    i_max_num_reorder_frames = 0; /* all IDR */
                    break;
                }
                /* fallthrough */
            default:
                i_max_num_reorder_frames = h264_get_max_dpb_frames( p_sps );
                break;
        }
    }

    *pi_depth = i_max_num_reorder_frames;
    *pi_delay = 0;

    return true;
}
```

相关代码见：

- [h264_nal.c](https://github.com/videolan/vlc/blob/d7ff28e96eb2cb64c5b1a502443a24229532a449/modules/packetizer/h264_nal.c#L735 "h264_nal.c")



**3）Chrome**

Chrome 计算重排窗口大小的实现方式如下：

- 1）判断 `sps` 的 `pic_order_cnt_type` 是否为 2，如果是，则直接返回 0，表示不用重排；
- 2）尝试取 `max_dpb_frames`，默认值为 16；
- 3）根据 AVC E.2.1 标准尝试取值 `max_num_reorder_frames` 并返回；
- 4）判断如果是特定的 profile，就直接返回 0，表示不用重排；
- 5）如果上述流程都没返回，则直接返回 `max_dpb_frames`。


```c
// TODO(sandersd): Share this computation with the VAAPI decoder.
int32_t ComputeH264ReorderWindow(const H264SPS* sps) {
  // When |pic_order_cnt_type| == 2, decode order always matches presentation
  // order.
  // TODO(sandersd): For |pic_order_cnt_type| == 1, analyze the delta cycle to
  // find the minimum required reorder window.
  if (sps->pic_order_cnt_type == 2)
    return 0;

  int max_dpb_mbs = H264LevelToMaxDpbMbs(sps->GetIndicatedLevel());
  int max_dpb_frames =
      max_dpb_mbs / ((sps->pic_width_in_mbs_minus1 + 1) *
                     (sps->pic_height_in_map_units_minus1 + 1));
  max_dpb_frames = std::clamp(max_dpb_frames, 0, 16);

  // See AVC spec section E.2.1 definition of |max_num_reorder_frames|.
  if (sps->vui_parameters_present_flag && sps->bitstream_restriction_flag) {
    return std::min(sps->max_num_reorder_frames, max_dpb_frames);
  } else if (sps->constraint_set3_flag) {
    if (sps->profile_idc == 44 || sps->profile_idc == 86 ||
        sps->profile_idc == 100 || sps->profile_idc == 110 ||
        sps->profile_idc == 122 || sps->profile_idc == 244) {
      return 0;
    }
  }
  return max_dpb_frames;
}

#if BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
int32_t ComputeHEVCReorderWindow(const H265VPS* vps) {
  int32_t vps_max_sub_layers_minus1 = vps->vps_max_sub_layers_minus1;
  return vps->vps_max_num_reorder_pics[vps_max_sub_layers_minus1];
}
#endif  // BUILDFLAG(ENABLE_HEVC_PARSER_AND_HW_DECODER)
```


相关代码见：

- [vt_video_decode_accelerator_mac.cc](https://source.chromium.org/chromium/chromium/src/+/main:media/gpu/mac/vt_video_decode_accelerator_mac.cc "vt_video_decode_accelerator_mac.cc")




参考：[如何获取 VideoToolbox 的 Reorder Size](https://lvpengwei.com/blog/2020/03/07/ru-he-huo-qu-videotoolboxde-reorder-size/ "如何获取 VideoToolbox 的 Reorder Size")

<!-- 对 H.264 来说这个缓冲区最大设置为 5 帧，对于 H.265 则最大设置为 7 帧，就是符合规范可以保证平滑播放的。为什么呢？
 -->





## 4、如何监控视频播放黑屏、花屏、绿屏等异常？

视频播放时如果遇到黑屏、花屏、绿屏通常会伴有解码器的报错或异常信息，我们可以上报这些异常信息来实现对这些情况的监控。

但是也有一些情况，即使出现黑屏、花屏、绿屏的情况了，解码器也没有报错或异常，这时候就需要我们对解码后的画面进行检测来识别这些问题。一般可以这样：

- 用传统的图像处理算法来识别。对于黑屏、绿屏可以用传统图像处理算法来进行识别，但这里也会有一些误识别的问题，比如视频本身就有些亮度较低、正常全黑帧、正常全绿帧的情况，可能也会被识别为异常。
- 训练 AI 模型类识别。对于花屏，可以训练 AI 模型来进行识别。训练过程可以通过人工构造丢帧视频的方法来生成花屏样本，同时筛选出无花屏问题的正常样本，基于这两类样本来做二分类模型的训练。





<!--  
视频不良画质检测
https://www.nxrte.com/jishu/15262.html
-->





---


更多的音视频知识、面试题、技术方案干货可以进群来看：

![扫描加入](assets/img/keyframe-zsxq.png)















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

