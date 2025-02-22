---
title: FFmpeg 工具：常用的 FFmpeg 命令合集
description: 介绍 ffmpeg 常用命令介绍、ffplay 常用命令介绍、ffprobe 常用命令。
author: Keyframe
date: 2025-02-22 09:56:08 +0800
categories: [音视频工具]
tags: [音视频工具, 音视频, FFmpeg]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }

（本文基本逻辑：ffmpeg 常用命令介绍 → ffplay 常用命令介绍 → ffprobe 常用命令介绍）


从事音视频开发的程序员几乎都应该知道或使用过 FFmpeg。FFmpeg 是一个开源软件，采用 LGPL 或 GPL 许可证（需要注意这里的开源协议，它具有『传染性』，会要求它的使用方也开源）。**我们可以使用 FFmpeg 来进行多种格式音频和视频的录制、转换、流处理功能。**


FFmpeg 由多个组件组成，包含了命令行应用程序以及一系列函数库：

- 命令行应用程序：
	- **ffmpeg**：用于音视频的编解码、格式转换以及音视频流的内容处理。
	- **ffplay**：基于 SDL 与 ffmpeg 库实现的一个播放器。
	- **ffprobe**：音视频分析工具。
- 函数库：
	- **libavcodec**：编解码库。
	- **libavformat**：音视频容器格式以及所支持的协议的封装和解析。
	- **libavutil**：提供了一些公共函数，工具库。
	- **libavfilter**：音视频的滤镜库，如视频加水印、音频变声等。
	- **libavdevice**：支持众多设备数据的输入与输出，如读取摄像头数据、屏幕录制。
	- **libswresample, libavresample**：提供音频的重采样工具库。
	- **libswscale**：提供对视频图像进行色彩转换、缩放以及像素格式转换，如图像的 YUV 转换。
	- **libpostproc**：多媒体后处理器。



如果你使用 Mac 设备，在 Mac 上安装 FFmpeg 可以用 **Homebrew**：

```
$ brew install ffmpeg
```

至于 Homebrew 的安装，以及使用它安装 ffmpeg 的相关细节，这里就不做过多探讨了。


本文主要介绍 FFmpeg 命令行应用程序的使用，这是我们在音视频开发中必不可少的工具。




## 1、ffmpeg 命令行工具

ffmpeg 是一个音视频编解码、格式转换以及音视频流内容处理的工具。



### 1.1、基础能力

通过下列命令可以查看当前 ffmpeg 工具所支持的能力：


```
// 获取帮助
$ ffmpeg -help

// 支持的格式
$ ffmpeg -formats

// 支持的解码
$ ffmpeg -decoders

// 支持的编码
$ ffmpeg -encoders

// 支持的协议
$ ffmpeg -protocols
```



### 1.2、转封装


可以使用下列命令来转封装：

```
$ ffmpeg -i <输入文件路径> -c copy -f <输出封装格式> <输入文件路径>
```


**1）转 MP4**


MP4 是当下短视频最常使用的封装格式，关于 MP4 格式更详细的介绍，参见[《MP4 格式》](https://mp.weixin.qq.com/)。

FFmpeg 封装 MP4 常用参数：

![](assets/resource/av-tool/ffmpeg-tool-1.png)


示例：将 FLV 的文件转封装成 MP4 并将 moov box 移动到文件头部。

```
$ ffmpeg -i input.flv -c copy -f mp4 -movflags faststart output.mp4
```



**2）转 FLV**

FLV 是当下实时直播最常使用的封装格式，关于 FLV 格式更详细的介绍，参见[《FLV 格式》](https://mp.weixin.qq.com/)。

FFmpeg 封装 FLV 常用参数：

![](assets/resource/av-tool/ffmpeg-tool-2.png)



示例：将 MP4 的文件转封装成 FLV。

```
$ ffmpeg -i input.mp4 -c copy -f flv output.flv
```

FLV 封装中可以支持的音频编码和视频编码是有限的，在转封装的时候，如果音频或视频不符合标准时，会封装不了而报错。一般，我们可以在转封装的时候同时将音频和视频转码成 FLV 支持的格式。

示例：将 MP4 的文件转封装成 FLV 并确保音频转码为 AAC。

```
$ ffmpeg -i input.mp4 -vcodec copy -acodec aac -f flv output.flv
```




**3）转 HLS**


HLS 是当下直播回放和部分实时直播场景最常使用的协议，它对应的媒体格式是 M3U8 + TS，关于 HLS 更详细的介绍，参见[《HLS 协议》](https://mp.weixin.qq.com/)、[《M3U8 格式》](https://mp.weixin.qq.com/)、[《TS 格式》](https://mp.weixin.qq.com/)。

FFmpeg 封装 HLS 常用参数：

![](assets/resource/av-tool/ffmpeg-tool-3.png)



示例：将 MP4 的文件转封装成 HLS 直播。

```
$ ffmpeg -re -i input.mp4 -c copy -f hls -bsf:v h264_mp4toannexb output.m3u8
```

因为默认是 HLS 直播，所以生成的 M3U8 文件内容会随着切片的产生而更新。这里多了一个 `-bsf:v h264_mp4toannexb` 参数，它的作用是将 MP4 中的 H.264 数据转换为 H.264 AnnexB 标准的编码，AnnexB 标准的编码常见于实时传输流中。如果源文件为 FLV、TS 等可作为直播传输流的视频，则不需要这个参数。

- `re`：表示以本地帧率读数据。
- `bsf`：表示 Binary Stream Filter。


**4）音视频流抽取**

FFmpeg 除了转封装、转码之外，还可以提取音频流和视频流。

示例：从 MP4 文件中提取 AAC 音频流。

```
$ ffmpeg -i input.mp4 -vn -acodec copy output.aac
```

- `vn`：表示不包含视频。


示例：从 MP4 文件中提取 H.264 视频流。

```
$ ffmpeg -i input.mp4 -an -vcodec copy output.h264
```

- `an`：表示不包含音频。


示例：从 MP4 文件中提取 H.265 视频流。

```
$ ffmpeg -i input.mp4 -an -vcodec copy -bsf hevc_mp4toannexb -f hevc output.hevc
```



### 1.3、转码

FFmpeg 一般使用 libx264 来进行软编码。下面是 x264 相关的编码参数：


![](assets/resource/av-tool/ffmpeg-tool-4.png)


**1）Preset**

示例：设置 preset 预设参数为 ultrafast 进行转码。

```
$ ffmpeg -i input.mp4 -vcodec libx264 -preset ultrafast -b:v 2000k output.mp4
```

- `b:v`：表示视频输出码率。


**2）Profile**

示例：设置 profile 为 high 进行转码。

```
$ ffmpeg -i input.mp4 -vcodec libx264 -profile:v high -level 3.1 -s 720x1280 -an -y -t 10 output_high.ts
```

- `y`：表示覆盖输出文件。
- `s`：表示输出分辨率。


使用 main profile 和 high profile 编码出来的视频是可以包含 B 帧的，转码完后，可以看一下：

```
$ ffprobe -v quiet -show_frames -select_streams v output_high.ts | grep "pict_type=B" | wc -l
```


**3）GOP**


示例：设置 GOP 为 50 帧，并且场景切换时不插入关键帧。

```
$ ffmpeg -i input.mp4 -c:v libx264 -g 50 -sc_threshold 0 -t 60 -y output.mp4
```


- `g`：以帧为单位设置 GOP 大小。
- `sc_threshold`：设定是否在场景切换时插入关键帧。0 表示不插入，1 表示插入。


**4）B 帧**


由于设置 x264 的参数比较多，所以 FFmpeg 开放了 x264opts 来设置 x264 内部的私有参数。

示例：设置 GOP 为 50 帧，并且场景切换时不插入关键帧，且不出现 B 帧。


```
$ ffmpeg -i input.mp4 -c:v libx264 -x264opts "bframes=0" -g 50 -sc_threshold 0 -t 60 -y output.mp4
```



示例：设置 GOP 为 50 帧，并且场景切换时不插入关键帧，且 2 个 P 帧之间存放 3 个 B 帧。


```
$ ffmpeg -i input.mp4 -c:v libx264 -x264opts "bframes=3:b-adapt=0" -g 50 -sc_threshold 0 -t 60 -y output.mp4
```


**5）码率**

编码时能够设置 VBR、CBR 编码模式，VBR 表示可变码率，CBR 表示恒定码率。


示例：

```
$ ffmpeg -i input.mp4 -c:v libx264 -x264opts "bframes=10:b-adapt=0" -b:v 1000k -maxrate 1000k -minrate 1000k -bufsize 50k -nal-hrd cbr -g 50 -sc_threshold 0 -t 60 -y output.ts
```

上面的命令比较复杂，分别做了这些事：

- `-x264opts "bframes=10:b-adapt=0"`：设置 B 帧个数为 2 个 P 帧之间包含 10 个 B 帧。
- `-b:v 1000k`：设置视频平均码率为 1000kbps。
- `-maxrate 1000k`：设置视频最大码率为 1000kbps。
- `-minrate 1000k`：设置视频最小码率为 1000kbps。
- `-bufsize 50k`：设置编码的 buffer 大小为 50KB。
- `-nal-hrd cbr`：设置 H.264 的编码 HRD 信号形式为 CBR。
- `-g 50`：设置每 50 帧一个 GOP。
- `-sc_threshold 0`：设置场景切换不插入关键帧。



### 1.4、流媒体


**1）发布 RTMP 流**

RTMP 是当下实时直播最常使用的推流协议，关于 RTMP 协议更详细的介绍，参见[《RTMP 协议》](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)。

FFmpeg 操作 RTMP 直播流使用的参数：

![](assets/resource/av-tool/ffmpeg-tool-5.png)


示例：本地 MP4 视频文件转封装为 FLV 后推流至指定 RTMP 流媒体服务器。

```
$ ffmpeg -re -i input.mp4 -c copy -f flv rtmp://localhost:1935/live/room
```


**2）录制 RTMP 流**

示例：RTMP 媒体流保存为 FLV 视频文件。

```
$ ffmpeg -i rtmp://localhost:1935/live/room -c copy output.flv
```

**3）录制 HTTP 流**

在流媒体服务中，HTTP 服务是最为常见的，尤其是点播。直播也是可以的，包括 HTTP-FLV、HTTP-TS、HLS。

FFmpeg 操作 HTTP 使用的参数：

![](assets/resource/av-tool/ffmpeg-tool-6.png)


示例：拉取并录制 FLV 直播流。

```
$ ffmpeg -i http://www.abc.com/live.flv -c copy -f flv output.flv
```


示例：拉取 TS 直播流流录制为 FLV。

```
$ ffmpeg -i http://www.abc.com/live.ts -c copy -f flv output.flv
```


示例：拉取 HLS 直播流流录制为 FLV。

```
$ ffmpeg -i http://www.abc.com/live.m3u8 -c copy -f flv output.flv
```




<!-- 

### 1.5、滤镜



### 1.6、采集 

-->





## 2、ffplay 命令行工具

ffplay 是基于 SDL 与 ffmpeg 库实现的一个播放器，可以使用它来播放原始的 YUV/PCM 数据、编码后的 H.264/H.265 等数据，封装好的 MP4/M4A 等数据，或是流媒体数据。



**1）播放原始声音数据**

```
$ ffplay -f <格式名> -ac <声道数> -ar <采样率> -i <文件路径>
```


其中，`-f` 表示 PCM 格式，可以用 `ffmpeg -formats | grep PCM` 命令查看当前支持的格式。



示例：

```
$ ffplay -f f32le -ac 1 -ar 48000 -i input.pcm
```


**2）播放原始图像数据**

```
$ ffplay -f <文件格式> -pixel_format <像素格式> -video_size <视频尺寸> -i <文件路径>
```

其中，`-pixel_format` 表示像素格式，可以用 `ffplay -pix_fmts` 命令开查看当前支持的格式。

示例：

```
$ ffplay -f rawvideo -pixel_format yuv420p -video_size 1280x720 -i input.yuv
```



**3）播放编码数据**

使用 ffplay 播放编码后的视频或音频文件如下所示：

```
$ ffplay -i <文件路径>
```

示例：

```
$ ffplay -i input.h264
```



**4）播放封装数据**

使用 ffplay 播放封装好的视频或音频文件如下所示：

```
$ ffplay -i <文件路径>
```

示例：

```
$ ffplay -i input.mp4
```

不过，这里还有一些可能会用到的功能可以关注一下：

**4.1）播放控制**

在播放音频或视频时，使用下列键盘按键可以进行播放控制：

- `w`，切换播放模式，比如在音频波形图、音频频谱图、视频画面之间切换。
- `s`，步进模式，每按一次就播放下一帧图像。
- `right`，快进 10 s。
- `left`，快退 10 s。
- `up`，快退 1 min。
- `down`，快退 1 min。
- `space`，暂停。
- `esc`，退出。

**4.2）循环播放**

通过 `-loop` 指定循环次数。

```
$ ffplay -loop <循环播放次数> -i <文件路径>
```


**4.3）播放某一路音频或视频**

通过 `-ast` 和 `-vst` 分别指定音频流和视频流编号。

```
$ ffplay -ast <音频流编号> -i <文件路径>
$ ffplay -vst <视频流编号> -i <文件路径>
```

如果不存在对应编号的音频流或视频流，则静音或没有画面。



**4.4）设置音视频同步方式**

通过 `-sync` 指定音视频同步方式。


```
$ ffplay -sync <同步方式> -i <文件路径>
```

其中同步方式有 3 种，包括：

- `audio`，以音频时钟为基准。
- `video`，以视频时钟为基准。
- `ext`，已外部时钟为基准。





## 3、ffprobe 命令行工具

ffprobe 是 FFmpeg 源码编译后生成的一个可执行程序。ffprobe 是一个很强大的多媒体分析工具，它可以从媒体文件或媒体流中获得音视频及媒体容器的参数信息。



**1）查看媒体封装信息**

使用 `-show_format` 来查看媒体封装信息。

```
$ ffprobe -show_format <文件路径>
```

下面是输出信息示例及字段含义说明：

```
[FORMAT]
filename=http://www.example.com/1.flv
nb_streams=2
nb_programs=0
format_name=flv
format_long_name=FLV (Flash Video)
start_time=4088.213000
duration=0.000000
size=N/A
bit_rate=N/A
probe_score=100
TAG:fileSize=0
TAG:audiochannels=2
TAG:encoder=xxx
[/FORMAT]
```

- `filename`：文件名。
- `nb_streams`：封装的流的数量，对应 `AVFormatContext->nb_streams`。
- `nb_programs`：对应 `AVFormatContext->nb_programs`。
- `format_name`：封装格式，对应 `AVFormatContext->iformat->name`。
- `format_long_name`：封装格式完整名，对应 `AVFormatContext->iformat->long_name`。
- `start_time`：开始时间，对应 `AVFormatContext->start_time`，基于 `AV_TIME_BASE_Q`，单位为秒。
- `duration`：时长，对应 `AVFormatContext->duration`，基于 `AV_TIME_BASE_Q`，单位为秒。
- `size`：大小，对应 `avio_size(AVFormatContext->pb)`，单位字节。
- `bit_rate`：码率，对应 `AVFormatContext->bit_rate`。
- `probe_score`：表示输入媒体文件的格式与其实际数据格式的匹配度，匹配度高则得分高（比如：1.mp4 确实是 mp4 格式），匹配度低则得分低（比如：1.mp4 其实是 wav 的格式）。对应 `AVFormatContext->probe_score`。
- `TAG:*`：TAG 是从 metadata dump 处理的信息。







**2）查看媒体流信息**


使用 `-show_streams` 来查看媒体流信息。


```
$ ffprobe -show_streams <文件路径>
```

下面是输出信息示例及字段含义说明：


```
[STREAM]
index=0
codec_name=h264
codec_long_name=H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10
profile=High
codec_type=video
codec_time_base=1/30
codec_tag_string=[0][0][0][0]
codec_tag=0x0000
width=720
height=1280
coded_width=720
coded_height=1280
has_b_frames=1
sample_aspect_ratio=N/A
display_aspect_ratio=N/A
pix_fmt=yuv420p
level=31
color_range=unknown
color_space=unknown
color_transfer=unknown
color_primaries=unknown
chroma_location=left
field_order=progressive
timecode=N/A
refs=1
is_avc=true
nal_length_size=4
id=N/A
r_frame_rate=15/1
avg_frame_rate=15/1
time_base=1/1000
start_pts=1030
start_time=1.030000
duration_ts=N/A
duration=N/A
bit_rate=N/A
max_bit_rate=N/A
bits_per_raw_sample=8
nb_frames=N/A
nb_read_frames=N/A
nb_read_packets=N/A
DISPOSITION:default=0
DISPOSITION:dub=0
DISPOSITION:original=0
DISPOSITION:comment=0
DISPOSITION:lyrics=0
DISPOSITION:karaoke=0
DISPOSITION:forced=0
DISPOSITION:hearing_impaired=0
DISPOSITION:visual_impaired=0
DISPOSITION:clean_effects=0
DISPOSITION:attached_pic=0
DISPOSITION:timed_thumbnails=0
[/STREAM]
[STREAM]
index=1
codec_name=aac
codec_long_name=AAC (Advanced Audio Coding)
profile=LC
codec_type=audio
codec_time_base=1/48000
codec_tag_string=[0][0][0][0]
codec_tag=0x0000
sample_fmt=fltp
sample_rate=48000
channels=2
channel_layout=stereo
bits_per_sample=0
id=N/A
r_frame_rate=0/0
avg_frame_rate=0/0
time_base=1/1000
start_pts=55
start_time=0.055000
duration_ts=N/A
duration=N/A
bit_rate=N/A
max_bit_rate=N/A
bits_per_raw_sample=N/A
nb_frames=N/A
nb_read_frames=N/A
nb_read_packets=N/A
DISPOSITION:default=0
DISPOSITION:dub=0
DISPOSITION:original=0
DISPOSITION:comment=0
DISPOSITION:lyrics=0
DISPOSITION:karaoke=0
DISPOSITION:forced=0
DISPOSITION:hearing_impaired=0
DISPOSITION:visual_impaired=0
DISPOSITION:clean_effects=0
DISPOSITION:attached_pic=0
DISPOSITION:timed_thumbnails=0
[/STREAM]
```

视频流：

- `index`：当前流的索引号，对应 `AVStream->index`。
- `codec_name`：解码器名称，对应 `AVCodecDescriptor *cd = avcodec_descriptor_get(AVStream->codecpar->codec_id); cd->name`。
- `codec_long_name`：解码器全名，对应 `cd->long_name`。
- `profile`：编码等级，通过 `avcodec_profile_name(AVStream->codecpar->codec_id, AVStream->codecpar->profile)` 获得。
- `codec_type`：流类型，即 `av_get_media_type_string(AVStream->codecpar->codec_type)`。
- `codec_time_base`：编码的时间戳计算基础单位，对应 `AVStream->codec->time_base`。
- `codec_tag_string`：编码器标签描述，对应 `av_fourcc2str(AVStream->codecpar->codec_tag)`。
- `codec_tag`：对应 `AVStream->codecpar->codec_tag`。
- `width`：有效区域的宽度，对应 `AVStream->codecpar->width`。
- `height`：有效区域的高度，对应 `AVStream->codecpar->height`。
- `coded_width`：视频帧宽度，可能与上面的宽度不同，因为有一些编码器要求帧的宽或高是某个数的倍数，所以如果输入的视频帧的宽或高不符合对应的规则时，则需要做填充，这里的 `coded_width` 就是填充后的宽度，在解码时需要用到这个参数来做对应的裁剪。对应 `AVStream->codec->coded_width`。
- `coded_height`：视频帧高度，可能与上面的高度不同，对应 `AVStream->codec->coded_height`。
- `has_b_frames`：是否包含 B 帧。
- `sample_aspect_ratio`：简称 SAR，指的是图像采集时，横向采集点数与纵向采集点数的比例。FFmpeg提供了多个 SAR：`AVStream->sample_aspect_ratio`、`AVStream->codecpar->sample_aspect_ratio`、`AVFrame->sample_aspect_ratio`，通过 `av_guess_sample_aspect_ratio` 获取最终的 SAR。
- `display_aspect_ratio`：简称 DAR，指的是真正展示的图像宽高比，在渲染视频时，必须根据这个比例进行缩放。通过 `av_reduce` 计算得到，`PAR * SAR = DAR`，其中 PAR 是 Pixel Aspect Ratio，表示单个像素的宽高比，大多数情况像素宽高比为 1:1，也就是一个正方形像素，如果不是 1:1，则该像素可以理解为长方形像素。
- `pix_fmt`：像素格式，对应 `av_get_pix_fmt_name(AVStream->codecpar->format)`。
- `level`：编码参数，对应`AVStream->codecpar->level`。
- `color_range`：额外的色彩空间特征，对应 `av_color_range_name(AVStream->codecpar->color_range)`，`AVCOL_RANGE_MPEG` 对应 TV，`AVCOL_RANGE_JPEG` 对应 PC。
- `color_space`：YUV 彩色空间类型，对应 `av_color_space_name(AVStream->codecpar->color_space)`。
- `color_transfer`：颜色传输特性，对应 `av_color_transfer_name(AVStream->codecpar->color_trc)`。
- `color_primaries`：对应 `av_color_primaries_name(AVStream->codecpar->color_primaries)`。
- `chroma_location`：色度样品的位置，对应 `av_chroma_location_name(AVStream->codecpar->chroma_location)`。
- `field_order`：交错视频中字段的顺序，对应 `AVStream->codecpar->field_order`。
- `timecode`：通过 `av_timecode_make_mpeg_tc_string` 处理 `AVStream->codec->timecode_frame_start` 获得。
- `refs`：参考帧数量，即 `AVStream->codec->refs`。
- `is_avc`：是否 AVC。
- `nal_length_size`：表示用几个字节表示 NALU 的长度。
- `id`：
- `r_frame_rate`：当前流的基本帧率，这个值仅是一个猜测，对应于 `AVStream->r_frame_rate`。
- `avg_frame_rate`：平均帧率，对应于 `AVStream->avg_frame_rate`。
- `time_base`：AVStream 的时间基准，即 `AVStream->time_base`。
- `start_pts`：流开始的 PTS 时间戳，基于 `time_base`，即 `AVStream->start_time`。
- `start_time`：转换 `start_pts * time_base` 之后的开始时间，单位秒。
- `duration_ts`：流时长，基于 `time_base`，即 `AVStream->duration`。
- `duration`：转换 `duration_ts * time_base` 之后的时长，单位秒。
- `bit_rate`：码率，即 `AVStream->codecpar->bit_rate`。
- `max_bit_rate`：最大码率，即 `AVStream->codec->rc_max_rate`。
- `bits_per_raw_sample`：每个采样或像素的比特数，即 `AVStream->codec->bits_per_raw_sample`。
- `nb_frames`：视频流中的帧数，即 `AVStream->nb_frames`。
- `nb_read_frames`：略。
- `nb_read_packets`：略。
- `TAG:*`：对应 `AVStream->metadata` 中的信息。
	- `TAG:rotate`：逆时针的旋转角度（相当于正常视频的逆时针旋转角度）。
- `side_data`：在视频流中，有时候我们还会看到 `side_data` 数据，对应 `AVStream->side_data`，示例如下：

```
[SIDE_DATA]
// side_data 数据类型，Display Matrix 表示一个 3*3 的矩阵，这个矩阵需要应用到解码后的视频帧上，才能正确展示：
side_data_type=Display Matrix
displaymatrix=
00000000:		 0	65536			 0
00000001:	-65536		0			 0
00000002:		 0		0	1073741824
// 顺时针旋转 90 度还原视频
rotation=-90
[/SIDE_DATA]
```


音频流：

- `sample_fmt`：采样格式，通过 `av_get_sample_fmt_name(AVStream->codecpar->format)` 获取。
- `sample_rate`：采样率，即 `AVStream->codecpar->sample_rate`。
- `channels`：声道数，即 `AVStream->codecpar->channels`。
- `channel_layout`：声道类型，与 `channels` 是相对应，通过 `av_bprint_channel_layout` 获取，比如：mono 表示单声道，stereo 表示多声道。





**3）查看媒体数据包信息**

使用 `-show_streams` 来查看媒体数据包信息。


```
$ ffprobe -show_packets <文件路径>
```

下面是输出信息示例及字段含义说明：

```
[PACKET]
codec_type=audio
stream_index=0
pts=1690083
pts_time=1690.083000
dts=1690083
dts_time=1690.083000
duration=23
duration_time=0.023000
convergence_duration=N/A
convergence_duration_time=N/A
size=470
pos=2757652
flags=K_
[/PACKET]
[PACKET]
codec_type=video
stream_index=1
pts=1690232
pts_time=1690.232000
dts=1690099
dts_time=1690.099000
duration=33
duration_time=0.033000
convergence_duration=N/A
convergence_duration_time=N/A
size=11253
pos=2758139
flags=__
[/PACKET]
```


- `codec_type`：帧类型。audio 表示音频帧，video 表示视频帧。
- `stream_index`：当前帧所属流的索引，对应于 `AVStream->index`。
- `pts`：帧的展示时间戳，即 `AVPacket->pts`，基于 `AVStream->time_base` 时间基准。
- `pts_time`：转换 `pts * time_base` 之后的时长，单位秒。
- `dts`：帧的解码时间戳，即 `AVPacket->dts`，基于 `AVStream->time_base` 时间基准。
- `dts_time`：转换 `dts * time_base` 之后的时长，单位秒。
- `duration`：当前帧的时长，等于下一帧的 pts 减去当前帧 pts，即 `AVPacket->duration`，基于 `AVStream->time_base` 时间基准。
- `duration_time`：转换 `duration * time_base` 之后的时长，单位秒。
- `convergence_duration`：略。
- `convergence_duration_time`：略。
- `size`：当前帧的大小。
- `pos`：当前帧的位置，等于上一帧的 pos 加上当前帧的 size。
- `flags`：略。




**4）查看媒体帧信息**

使用 `-show_frames` 来查看媒体帧信息。

```
$ ffprobe -show_frames <文件路径>
```

下面是输出信息示例及字段含义说明：

```
[FRAME]
media_type=video
stream_index=1
key_frame=0
pkt_pts=2084699
pkt_pts_time=2084.699000
pkt_dts=2084699
pkt_dts_time=2084.699000
best_effort_timestamp=2084699
best_effort_timestamp_time=2084.699000
pkt_duration=33
pkt_duration_time=0.033000
pkt_pos=3751477
pkt_size=2665
width=720
height=1280
pix_fmt=yuv420p
sample_aspect_ratio=N/A
pict_type=B
coded_picture_number=334
display_picture_number=0
interlaced_frame=0
top_field_first=0
repeat_pict=0
color_range=unknown
color_space=unknown
color_primaries=unknown
color_transfer=unknown
chroma_location=left
[/FRAME]
[FRAME]
media_type=audio
stream_index=0
key_frame=1
pkt_pts=2084707
pkt_pts_time=2084.707000
pkt_dts=2084707
pkt_dts_time=2084.707000
best_effort_timestamp=2084707
best_effort_timestamp_time=2084.707000
pkt_duration=23
pkt_duration_time=0.023000
pkt_pos=3775354
pkt_size=472
sample_fmt=fltp
nb_samples=1024
channels=2
channel_layout=stereo
[/FRAME]
```


视频帧：

- `media_type=`：帧类型，即 `av_get_media_type_string(AVStream->codecpar->codec_type)`。
- `stream_index`：当前帧所属流的索引，对应于 `AVStream->index`。
- `key_frame`：是否关键帧（IDR)。
- `pkt_pts`：帧的展示时间戳，即 `AVFrame->pts`，基于 `AVStream->time_base` 时间基准。
- `pkt_pts_time`：转换 `pkt_pts * time_base` 之后的时长，单位秒。
- `pkt_dts`：帧的解码时间戳，即 `AVFrame->dts`，基于 `AVStream->time_base` 时间基准。
- `pkt_dts_time`：转换 `pkt_dts * time_base` 之后的时长，单位秒。
- `best_effort_timestamp`：帧时间戳，基本与 pts 相同，如果当前 pts 存在不合理值，会尝试进行一系列校准来得到这个更合理的值，对应 `AVFrame->best_effort_timestamp`，基于 `AVStream->time_base` 时间基准。
- `best_effort_timestamp_time`：转换 `best_effort_timestamp * time_base` 之后的时长，单位秒。
- `pkt_duration`：对应的 AVPacket 的帧时长，即 `AVFrame->pkt_duration`，基于 `AVStream->time_base` 时间基准。
- `pkt_duration_time`：转换 `pkt_duration * time_base` 之后的时长，单位秒。
- `pkt_pos`：从最后一个已输入解码器的 AVPacket 重新排序的位置，即 `AVFrame->pkt_pos`。
- `pkt_size`：对应的 AVPacket 的帧大小，即 `AVFrame->pkt_size`。
- `width`：旋转之前的帧宽度，即 `AVFrame->width`。
- `height`：旋转之前的帧高度，即 `AVFrame->height`。
- `pix_fmt`：像素格式，对应 `av_get_pix_fmt_name(AVFrame->format)`。
- `sample_aspect_ratio`：简称 SAR，指的是图像采集时，横向采集点数与纵向采集点数的比例。FFmpeg提供了多个 SAR：`AVStream->sample_aspect_ratio`、`AVStream->codecpar->sample_aspect_ratio`、`AVFrame->sample_aspect_ratio`，通过 `av_guess_sample_aspect_ratio` 获取最终的 SAR。
- `pict_type`：视频帧的图片类型，即 `av_get_picture_type_char(frame->pict_type)`。
- `coded_picture_number`：帧在比特流中的编号，即 `AVFrame->coded_picture_number`。
- `display_picture_number`：帧的显示编号，即 `AVFrame->display_picture_number`。
- `interlaced_frame`：视频帧内容是否是交错的，即 `AVFrame->interlaced_frame`。
- `top_field_first`：若视频帧内容是交错的，表示首先展示的顶部域，即 `AVFrame->top_field_first`。
- `repeat_pict`：当解码时，这个信号表明视频帧必须延迟多少。`extra_delay = repeat_pict / (2*fps)`，即 `AVFrame->repeat_pict`。
- `color_range`：额外的色彩空间特征，即 `av_color_range_name(AVFrame->color_range)`，`AVCOL_RANGE_MPEG` 对应 TV，`AVCOL_RANGE_JPEG` 对应 PC。
- `color_space`：YUV 彩色空间类型，即 `av_color_space_name(AVFrame->colorspace)`。
- `color_primaries`：即 `av_color_primaries_name(AVFrame->color_primaries)`。
- `color_transfer`：颜色传输特性，即 `av_color_transfer_name(AVFrame->color_trc)`。
- `chroma_location`：色度样品的位置，即 `av_chroma_location_name(AVFrame->chroma_location)`。


音频帧：

- `sample_fmt`：采样格式，通过 `av_get_sample_fmt_name(AVFrame->format)` 获取。
- `sample_rate`：采样率，即 `AVFrame->sample_rate`。
- `channels`：声道数，即 `AVFrame->channels`。
- `channel_layout`：声道类型，与 `channels` 是相对应，通过 `av_bprint_channel_layout` 获取，比如：mono 表示单声道，stereo 表示多声道。






<!--  

## 4、使用 Xcode 调试 FFmpeg


参考：

- [使用 Xcode 断点调试 ffmpeg](https://www.jianshu.com/p/226c19aa6e42)


## 5、实践示例

1、统计 B 帧的数量：

```
ffprobe -show_frames input.mp4 | grep "pict_type=B" | wc -l
```

I 帧、P 帧以此类推。


2、FFmpeg 将一个 moov 后置的 MP4 处理为前置：

```
ffmpeg -i slow_play.mp4 -movflags faststart fast_play.mp4
```

3、通过 show_frames 和 select_streams 组合查看视频帧信息：


```
ffprobe -of csv -show_frames -select_streams v http://www.example.com/1.flv > video_frames_info.csv
```


4、使用 ffplay 直接播放 yuv 和 pcm 文件：

播放 yuv 需要指定 yuv 格式和分辨率：

```
ffplay -i 1101.yuv -pix_fmt yuv420p(nv12) -s 640x480
```

播放 pcm 需要指定声道数、采样率和音频格式：

```
ffplay -ar 44100 -channels 1 -f s16le -i 2020.pcm
```



5、使用 ffplay 播放音频的频谱图和波形图：

频谱图：

```
ffplay -f lavfi 'amovie=test.aac, asplit [a][out1];[a] showspectrum=mode=separate:color=intensity:slide=1:scale=cbrt [out0]'
```

波形图：

```
ffplay -f lavfi 'amovie=test.aac,asplit[out0],showwaves[out1]'
```
-->


## 小结

通过上文的介绍，我们了解了 ffmpeg、ffplay、ffprobe 等常用的命令用法，这对我们平时的音视频开发工作非常有用。


## 本文参考

- [《FFmpeg 从入门到精通》](https://book.douban.com/subject/30178432/)
- [FFmpeg 之 ffprobe](https://juejin.im/post/5d5cbb9af265da03f564e757)



