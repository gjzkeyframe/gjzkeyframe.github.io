---
title: 音视频面试题集锦第 26 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 04:48:08 +0800
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

下面是第 26 期面试题精选：

- **1、FFmpeg 中 avformat_open_input() 经历了什么步骤？**
- **2、FFmpeg 编码的流程是什么？**
- **3、PCM 数据操作的最小单元是多少字节？**
- **4、iOS 音频帧 CMSampleBufferRef 中的 kCMFormatDescriptionExtension_VerbatimISOSampleEntry 保存哪些信息，是否可以去掉？**

## 1、FFmpeg 中 avformat_open_input() 经历了什么步骤？

`avformat_open_input()` 用于打开输入媒体流与读取头部信息，包括本地文件、网络流、自定义缓冲区。关键流程：打开avio、探测输入流的封装格式。对应的释放方法为 `avformat_close_input()`。`avformat_open_input()` 的流程如下：

- **打开输入媒体流**：`avformat_open_input` 方法位于 `libavformat/utils.c`，打开输入媒体流的流程包括：分配 AVFormatContext、设置 options、初始化输入流、拷贝白名单与黑名单协议、读取 ID3V2 参数。
- **初始化输入媒体流**：核心方法是 `init_input()`，用于打开输入媒体流、探测封装格式。
- **探测输入格式，根据当前参数进行打分，最终根据分数最高的那个配置作为最终格式**：分为三步：`read_probe`、`av_match_ext`、`av_match_name`；每一步匹配结果有个 score 得分，取最高分数的作为 format；每步匹配有不同的分数，最高分数为 100，retry 为 25，extension 为 50，mimetype 为 75。
- **打开 avio**：`avio_open2` 方法会打开 `ffurl_open_whitelist()` 和 `fifo_fdopen()`。



![avformat_open_input 流程](assets/resource/av-interview-qa/avformat_open_input.png)
_avformat_open_input 流程_




## 2、FFmpeg 编码的流程是什么？

FFmpeg 编码的流程如图：



![FFmpeg 编码的流程](assets/resource/av-interview-qa/ffmpeg_encode.png)
_FFmpeg 编码的流程_

- `av_register_all()`：注册 FFmpeg 所有编解码器。
- `avformat_alloc_context()`：初始化输出码流的 AVFormatContext。
- `avio_open()`：打开输出文件。
- `av_new_stream()`：创建输出码流的 AVStream。
- `avcodec_find_encoder()`：查找编码器。
- `avcodec_open2()`：打开编码器。
- `avformat_write_header()`：写文件头（对于某些没有文件头的封装格式，不需要此函数，比如 MPEG2TS）。
- `avcodec_send_frame()`：编码核心接口新接口，发送一帧视频给编码器。即是 AVFrame（存储YUV像素数据）。
- `avcodec_receive_packet()`：编码核心接口新接口，接收编码器编码后的一帧视频，AVPacket（存储 H.264 等格式的码流数据）。
- `av_write_frame()`：将编码后的视频码流写入文件。
- `flush_encoder()`：输入的像素数据读取完成后调用此函数。用于输出编码器中剩余的 AVPacket。
- `av_write_trailer()`：写文件尾（对于某些没有文件头的封装格式，不需要此函数，比如 MPEG2TS）。




## 3、PCM 数据操作的最小单元是多少字节？

音频 PCM 数据，如果要进行编辑，那么其最小操作单元是：`声道数 * 位数 / 8 * 1 个采样点`。

比如：如果 PCM 是双声道、32 位，那么其最小操作单元是：2 * 32 / 8 * 1 = 8 个字节，如果不是按照 8 字节进行操作，一般就会是编辑后的 PCM 数据变成了白噪音。




## 4、iOS 音频帧 CMSampleBufferRef 中的 kCMFormatDescriptionExtension_VerbatimISOSampleEntry 保存哪些信息，是否可以去掉？

根据 Apple 官方文档：

```objc
@abstract	Preserves the original ISOSampleEntry data.
@discussion This extension is used to ensure that roundtrips from ISO Sample Entry (ie. AudioSampleEntry or VisualSampleEntry)
to CMFormatDescriptions back to ISO Sample Entry preserve the exact original sample descriptions.
IMPORTANT: If you make a modified clone of a CMFormatDescription, you must delete this extension from the clone, or your modifications could be lost.
```

其主要用于对应 AudioSampleEntry，这个是封装协议中的名字，MP4 协议中有此定义：avcC、mp4a Box。具体定义如下：

```c++
// BaseBox.h
// ...
// 其他 Box 的定义

class AvcBox : public BaseBox {
public:
unsigned char reserver[6] = {0}; // 2 保留位默认 0x00
unsigned short data_reference_ID = 0; // 2 引用参考 Dref Box
unsigned short code_stream_version = 0; // 2 一般都是默认值 0
unsigned char reserver2 [14] = {0};
unsigned short width = 0;
unsigned short heigth = 0;
unsigned int horizontal_resolution = 0; // 4 默认值一般都是 72dpi(前两个字节整数部分，后面两个字节小数部分)
unsigned int vertical_resolution = 0; // 4 默认值一般都是 72dpi(前两个字节整数部分，后面两个字节小数部分)
unsigned int reserver3 = 0;
unsigned short frame_count = 0; // 默认值一般是 0x00 01 每采样点图像的帧数，一般为 1，有些情况下，每个采样点有多帧
Byte compressorname[32] = {0};
unsigned short depth = 0; // 两个字节默认值是 0x00 0x18 即 24
unsigned short reserver4 = 0;

AvcBox(BoxHeader h);
AvcBox(BoxHeader h, Timebyte * d): BaseBox(h, d){};
void PrintDataInfo() override;
size_t GetDataOffset() const override {
		// Data属性大小
		return 78;
}

void AnalyzeBoxHeader(BoxHeader header, size_t offset) override;
};

class MP4aBox : public BaseBox {
public:
unsigned char reserver[6] = {0}; // 2 保留位默认 0x00
unsigned short data_reference_ID = 0; // 2 引用参考 Dref Box
unsigned char reserver2 [8] = {0};
unsigned short channelcount = 0;
unsigned short samplesize = 0;
unsigned char reserver3 [4] = {0};
unsigned int samplerate = 0;

TimeMP4a(BoxHeader h);
TimeMP4a(BoxHeader h, Timebyte * d): BaseBox(h, d){};
void PrintDataInfo() override;
size_t GetDataOffset() const override {
	// Data属性大小
	return 28;
}
void AnalyzeBoxHeader(BoxHeader header, size_t offset) override;
};
```

可见这里存放的信息是很重要的，是不能随意丢弃的，尤其是在自己实现 ATB 编码，然后 AVAssetWriter 进行 muxer 时，这个信息是必不可少，必要时需要自己去手动创建。

------

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![限时优惠，扫码加入](assets/img/keyframe-zsxq.png)


