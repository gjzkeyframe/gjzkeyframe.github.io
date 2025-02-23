---
title: iOS 音频处理框架及重点 API 合集
description: 介绍 iOS 音频处理框架及重点 API 的细节。
author: Keyframe
date: 2025-02-23 08:08:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, 音频]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }

iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目的前面 6 篇 AVDemo 文章中，我们拆解了**音频**的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并基于 iOS 系统 API 实现了 Demo：


- [iOS AVDemo（1）：音频采集](https://mp.weixin.qq.com/s/FDR_5cMfAJQgZhSvjgeWYA)
- [iOS AVDemo（2）：音频编码](https://mp.weixin.qq.com/s/q4n1dYTjcJVJolX-Wrdr9Q)
- [iOS AVDemo（3）：音频封装](https://mp.weixin.qq.com/s/R86qnQAi2njr6k7tFvTF-w)
- [iOS AVDemo（4）：音频解封装](https://mp.weixin.qq.com/s/fCZfIXriTXUPcI4d4te_ew)
- [iOS AVDemo（5）：音频解码](https://mp.weixin.qq.com/s/7Db81B9i16cLuq0jS42bmg)
- [iOS AVDemo（6）：音频渲染](https://mp.weixin.qq.com/s/xrt277Ia1OFP_XtwK1qlQg)

**你可以在关注本公众号后，在公众号发送消息『AVDemo』来获取 Demo 的全部源码。**

如果你看完这些 Demo，对 iOS 平台的音视频开发多多少少会有一些认识了，在这个基础上我们来总结一下 iOS 音频处理框架，以及在前面的 Demo 中我们用到的主要 API 和数据结构有哪些。



## 1、iOS 音频框架

当我们想要了解 iOS 的音频处理框架时，以下是我们能比较容易找到的两张官方架构图。它们分别出自 [Audio Unit Hosting Guide for iOS](https://developer.apple.com/library/archive/documentation/MusicAudio/Conceptual/AudioUnitHostingGuide_iOS/AudioUnitHostingFundamentals/AudioUnitHostingFundamentals.html "Audio Unit Hosting Guide for iOS") 和 [Core Audio Overview](https://developer.apple.com/library/archive/documentation/MusicAudio/Conceptual/CoreAudioOverview/CoreAudioEssentials/CoreAudioEssentials.html "Core Audio Overview") 这两篇文档。

![iOS Audio Frameworks](assets/resource/av-demo/av-demo-ios-audio-api-1.png)
_iOS Audio Frameworks_


![Core Audio API Layers](assets/resource/av-demo/av-demo-ios-audio-api-2.png)
_Core Audio API Layers_


但这两篇文档已经比较陈旧了，是多年之前的文档，以至于和最新的 iOS 15 的框架有不少的出入。而新版本的 iOS 官方技术文档也没有给出比较清晰的音频架构图。所以在这里我们就按照 Demo 中涉及的系统 API 来挑选介绍几个相关的 Framework：

- Audio Unit Framework
- Core Media Framework
- Audio Toolbox Framework
- AVFoundation Framework



## 2、Audio Unit Framework

[Audio Unit Framework](https://developer.apple.com/documentation/audiounit?language=objc "Audio Unit")：最底层的音频处理 API，功能强大，直接驱动底层硬件，提供快速、模块化的音频处理。当你要实现`低延迟的音频处理`（比如 VoIP）、`对合成声音进行响应式的播放`（比如音乐游戏、合成乐器声音）、`实现特定的音频能力`（比如回声消除、混音、声音均衡）、`实现音频处理链支持灵活组装音频处理单元`时，你可以选择使用 Audio Unit 的 API。

**需要注意的是，在最新的 iOS 系统库架构中，Audio Unit Framework 的实现都已经迁移到 Audio Toolbox Framework 中去了。**


下面是 Audio Unit 框架的主要模块：

1）[Audio Component Services](https://developer.apple.com/documentation/audiounit/audio_component_services?language=objc "Audio Component Services")：定义了发现、开启和关闭音频单元（audio unit）以及音频编解码器的接口。

常用的数据类型：

- [AudioComponent](https://developer.apple.com/documentation/audiotoolbox/audiocomponent?language=objc "AudioComponent")：表示音频组件。一种音频组件通常由 type、subtype、manufacturer 三属性来唯一标识。
- [AudioComponentDescription](https://developer.apple.com/documentation/audiotoolbox/audiocomponentdescription?language=objc "AudioComponentDescription")：表示音频组件的描述。其中 type、subtype、manufacturer 三属性组合起来标识一种音频组件。
- [AudioComponentInstance](https://developer.apple.com/documentation/audiotoolbox/audiocomponentinstance?language=objc "AudioComponentInstance")：表示音频组件的实例。通常我们使用`查找方法（AudioComponentFindNext）`来找到符合描述的音频组件，然后再去使用`创建方法（AudioComponentInstanceNew）`创建一个对应的音频组件实例。

常用的接口：

- [AudioComponentFindNext(...)](https://developer.apple.com/documentation/audiotoolbox/1410445-audiocomponentfindnext?language=objc "AudioComponentFindNext(...)")：用于查找符合描述的音频组件。
- [AudioComponentGetDescription(...)](https://developer.apple.com/documentation/audiotoolbox/1410523-audiocomponentgetdescription?language=objc "AudioComponentGetDescription(...)")：用于获取一种音频组件对应的描述。
- [AudioComponentInstanceNew(...)](https://developer.apple.com/documentation/audiotoolbox/1410465-audiocomponentinstancenew?language=objc "AudioComponentInstanceNew(...)")：创建一个音频组件实例。
- [AudioComponentInstanceDispose(...)](https://developer.apple.com/documentation/audiotoolbox/1410508-audiocomponentinstancedispose "AudioComponentInstanceDispose(...)")：释放一个音频组件实例。

2）[Audio Unit Component Services](https://developer.apple.com/documentation/audiounit/audio_component_services?language=objc "Audio Unit Component Services")：提供了使用音频单元（audio unit）的 C 语言接口。一个音频单元（audio unit）是用来进行音频数据处理或者音频数据生成的插件单元。要发现、开启、关闭音频单元（audio unit）则可以使用 Audio Component Services。

常用的数据类型：

- [AudioUnit](https://developer.apple.com/documentation/audiotoolbox/audiounit?language=objc "AudioUnit")，`typedef AudioComponentInstance AudioUnit;`。AudioUnit 就是一种 AudioComponentInstance。
- [AudioUnitParameter](https://developer.apple.com/documentation/audiotoolbox/audiounitparameter?language=objc "AudioUnitParameter")：表示 AudioUnit 的参数，一个 AudioUnit 参数由 scope、element、parameterID 三属性定义。
- [AudioUnitProperty](https://developer.apple.com/documentation/audiotoolbox/audiounitproperty?language=objc "AudioUnitProperty")：表示 AudioUnit 的属性，一个 AudioUnitProperty 由 scope、element、propertyID 三属性定义。



常用的接口：

- [AudioUnitInitialize(...)](https://developer.apple.com/documentation/audiotoolbox/1439851-audiounitinitialize?language=objc "AudioUnitInitialize(...)")：初始化一个 AudioUnit。如果初始化成功，说明 input/output 的格式是可支持的，并且处于可以开始渲染的状态。
- [AudioUnitUninitialize(...)](https://developer.apple.com/documentation/audiotoolbox/1438415-audiounituninitialize?language=objc "AudioUnitUninitialize(...)")：卸载一个 AudioUnit。一旦一个 AudioUnit 被初始化后，要想改变它的状态来响应某些环境变化，就需要先卸载。这时候会使得 AudioUnit 释放它的资源。此后，调用者可以重新配置这个 AudioUnit 来适配新的环境，比如处理与之前不同的采样率。在这之后，可以重新初始化这个 AudioUnit 来应用这些更改。
- [AudioUnitRender(...)](https://developer.apple.com/documentation/audiotoolbox/1438430-audiounitrender?language=objc "AudioUnitRender(...)")：渲染输入的音频数据，数据量为 inNumberOfFrames。渲染结果的数据存储在 ioData 中。注意调用方需要提供音频时间戳，这个时间戳应该满足单调递增。
- [AudioUnitGetProperty(...)](https://developer.apple.com/documentation/audiotoolbox/1439840-audiounitgetproperty?language=objc "AudioUnitGetProperty(...)")：获取 AudioUnit 的属性。
- [AudioUnitSetProperty(...)](https://developer.apple.com/documentation/audiotoolbox/1440371-audiounitsetproperty?language=objc "AudioUnitSetProperty(...)")：设置 AudioUnit 的属性。
- [AudioUnitGetParameter(...)](https://developer.apple.com/documentation/audiotoolbox/1440055-audiounitgetparameter?language=objc "AudioUnitGetParameter(...)")：获取 AudioUnit 的参数。
- [AudioUnitSetParameter(...)](https://developer.apple.com/documentation/audiotoolbox/1438454-audiounitsetparameter?language=objc "AudioUnitSetParameter(...)")：设置 AudioUnit 的参数。

常用的回调：

- [AURenderCallback](https://developer.apple.com/documentation/audiotoolbox/aurendercallback?language=objc "AURenderCallback")：在以下几种情况会被系统调用：当 AudioUnit 需要输入采样数据；在一个渲染操作前；在一个渲染操作后。
- [AudioUnitPropertyListenerProc](https://developer.apple.com/documentation/audiotoolbox/audiounitpropertylistenerproc?language=objc "AudioUnitPropertyListenerProc")：当一个指定的 AudioUnit 的 property 发生改变时，会被系统调用。


3）[Output Audio Unit Services](https://developer.apple.com/documentation/audiounit/output_audio_unit_services?language=objc "Output Audio Unit Services")：提供了 start、stop 用于 I/O 的音频单元（通常是用于输出的音频单元）的 C 语言接口。

常用的接口：

- [AudioOutputUnitStart(...)](https://developer.apple.com/documentation/audiotoolbox/1439763-audiooutputunitstart?language=objc "AudioOutputUnitStart(...)")：启动一个 I/O AudioUnit，同时会启动与之连接的 AudioUnit Processing Graph。
- [AudioOutputUnitStop(...)](https://developer.apple.com/documentation/audiotoolbox/1440513-audiooutputunitstop?language=objc "AudioOutputUnitStop(...)")：关闭一个 I/O AudioUnit，同时会关闭与之连接的 AudioUnit Processing Graph。







## 3、Core Media Framework

[Core Media Framework](https://developer.apple.com/documentation/coremedia?language=objc "Core Media")：定义和封装了 AVFoundation 等更上层的媒体框架需要的媒体处理流水线（包含时间信息）以及其中使用的接口和数据类型。使用 Core Media 层的接口和数据类型可以高效的处理媒体采样数据、管理采样数据队列。下面是 Core Media 框架的主要模块：

1）[Sample Processing](https://developer.apple.com/documentation/coremedia?language=objc "Sample Processing")：采样数据处理。常用的数据类型：

- [CMSampleBuffer](https://developer.apple.com/documentation/coremedia/cmsamplebuffer-u71?language=objc "CMSampleBuffer")：系统用来在音视频处理的 pipeline 中使用和传递媒体采样数据的核心数据结构。你可以认为它是 iOS 音视频处理 pipeline 中的流通货币，摄像头采集的视频数据接口、麦克风采集的音频数据接口、编码和解码数据接口、读取和存储视频接口、视频渲染接口等等，都以它作为参数。通常，CMSampleBuffer 中要么包含一个或多个媒体采样的 CMBlockBuffer，要么包含一个 CVImageBuffer。
	- [CMSampleBufferCreateReady(...)](https://developer.apple.com/documentation/coremedia/1489513-cmsamplebuffercreateready?language=objc "CMSampleBufferCreateReady(...)")：基于媒体数据创建一个 CMSampleBuffer。 
	- [CMSampleBufferCreate(...)](https://developer.apple.com/documentation/coremedia/1489723-cmsamplebuffercreate?language=objc "CMSampleBufferCreate(...)")：创建一个 CMSampleBuffer，支持设置数据已准备好的回调。
	- [CMSampleBufferSetDataBufferFromAudioBufferList(...)](https://developer.apple.com/documentation/coremedia/1489725-cmsamplebuffersetdatabufferfroma?language=objc "CMSampleBufferSetDataBufferFromAudioBufferList(...)")：为指定的 CMSampleBuffer 创建其对应的 CMBlockBuffer，其中的数据拷贝自 AudioBufferList。
	- [CMSampleBufferGetFormatDescription(...)](https://developer.apple.com/documentation/coremedia/1489185-cmsamplebuffergetformatdescripti?language=objc "CMSampleBufferGetFormatDescription(...)")：返回 CMSampleBuffer 中的采样数据对应的 CMFormatDescription。
	- [CMSampleBufferGetDataBuffer(...)](https://developer.apple.com/documentation/coremedia/1489629-cmsamplebuffergetdatabuffer?language=objc "CMSampleBufferGetDataBuffer(...)")：返回 CMSampleBuffer 中的 CMBlockBuffer。注意调用方不会持有返回的 CMBlockBuffer，如果想要维护指向它的指针，需要显式 retain 一下。
	- [CMSampleBufferGetPresentationTimeStamp(...)](https://developer.apple.com/documentation/coremedia/1489252-cmsamplebuffergetpresentationtim?language=objc "CMSampleBufferGetPresentationTimeStamp(...)")：获取 CMSampleBuffer 中所有采样的最小的 pts 时间戳。
- [CMBlockBuffer](https://developer.apple.com/documentation/coremedia/cmblockbuffer-u9i?language=objc "CMBlockBuffer")：一个或多个媒体采样的的裸数据。其中可以封装：音频采集后、编码后、解码后的数据（如：PCM 数据、AAC 数据）；视频编码后的数据（如：H.264 数据）。
	- [CMBlockBufferCreateWithMemoryBlock(...)](https://developer.apple.com/documentation/coremedia/1489501-cmblockbuffercreatewithmemoryblo?language=objc "CMBlockBufferCreateWithMemoryBlock(...)")：基于内存数据创建一个 CMBlockBuffer。
	- [CMBlockBufferGetDataPointer(...)](https://developer.apple.com/documentation/coremedia/1489264-cmblockbuffergetdatapointer?language=objc "CMBlockBufferGetDataPointer(...)")：获取访问 CMBlockBuffer 中数据的地址。
- [CMFormatDescription](https://developer.apple.com/documentation/coremedia/cmformatdescription-u8g?language=objc "CMFormatDescription")：用于描述 CMSampleBuffer 中采样的格式信息。
	- [CMFormatDescriptionCreate(...)](https://developer.apple.com/documentation/coremedia/1489182-cmformatdescriptioncreate?language=objc "CMFormatDescriptionCreate(...)")：创建一个 CMFormatDescription。
- [CMAudioFormatDescription](https://developer.apple.com/documentation/coremedia/cmaudioformatdescription?language=objc "CMAudioFormatDescription")：`typedef CMFormatDescription CMAudioFormatDescription;`。CMAudioFormatDescription 是一种 CMFormatDescription。
	- [CMAudioFormatDescriptionCreate(...)](https://developer.apple.com/documentation/coremedia/1489522-cmaudioformatdescriptioncreate?language=objc "CMAudioFormatDescriptionCreate(...)")：基于 AudioStreamBasicDescription 来创建一个 CMAudioFormatDescription。
	- [CMAudioFormatDescriptionGetStreamBasicDescription(...)](https://developer.apple.com/documentation/coremedia/1489226-cmaudioformatdescriptiongetstrea?language=objc "CMAudioFormatDescriptionGetStreamBasicDescription(...)")：返回一个指向 CMFormatDescription（通常应该是一个 CMAudioFormatDescription） 中的 AudioStreamBasicDescription 的指针。如果是非音频格式，就返回 NULL。
- [CMAttachment](https://developer.apple.com/documentation/coremedia/cmattachment?language=objc "CMAttachment")：为 CMSampleBuffer 添加支持的 metadata。


这里我们还要补充介绍 CoreAudioTypes Framework 中的几种数据类型：

- [AudioStreamBasicDescription](https://developer.apple.com/documentation/coreaudiotypes/audiostreambasicdescription?language=objc "AudioStreamBasicDescription")：用于描述音频流数据格式信息，比如采样位深、声道数、采样率、每帧字节数、每包帧数、每包字节数、格式标识等。
- [AudioBuffer](https://developer.apple.com/documentation/coreaudiotypes/audiobuffer?language=objc "AudioBuffer")：存储并描述音频数据的缓冲区。mData 中存储着数据。可以存储两种不同类型的音频：1）单声道音频数据；2）多声道的交错音频数据，这时候 mNumberChannels 指定了声道数。
- [AudioBufferList](https://developer.apple.com/documentation/coreaudiotypes/audiobufferlist?language=objc "AudioBufferList")：一组 AudioBuffer。
- [AudioTimeStamp](https://developer.apple.com/documentation/coreaudiotypes/audiotimestamp?language=objc "AudioTimeStamp")：从多维度来表示一个时间戳的数据结构。



2）[Time Representation](https://developer.apple.com/documentation/coremedia?language=objc "Time Representation")：时间信息表示。常用的数据类型：

- [CMTime](https://developer.apple.com/documentation/coremedia/cmtime-u58?language=objc "CMTime")：用 value/timescale 的方式表示时间。这样可以解决浮点运算时的精度损失问题。timescale 表示时间刻度，通常在处理视频内容时常见的时间刻度为 600，这是大部分常用视频帧率 24fps、25fps、30fps 的公倍数，音频数据常见的时间刻度就是采样率，比如 44100 或 48000。
- [CMTimeRange](https://developer.apple.com/documentation/coremedia/cmtimerange-qts?language=objc "CMTimeRange")：用 start+duration 的方式表示一段时间。
- [CMSampleTimingInfo](https://developer.apple.com/documentation/coremedia/cmsampletiminginfo?language=objc "CMSampleTimingInfo")：一个 CMSampleBuffer 的时间戳信息，包括 pts、dts、duration。



3）[Queues](https://developer.apple.com/documentation/coremedia?language=objc "Queues")：数据容器。常用的数据类型：

- [CMSimpleQueue](https://developer.apple.com/documentation/coremedia/cmsimplequeue?language=objc "CMSimpleQueue")：一个简单地、无锁的 FIFO 队列，可以放 `(void *)` 元素，元素不能是 NULL 或 0，如果元素是指向分配内存的指针，其内存生命周期要在外面自己管理。可以用作音视频采样数据（CMSampleBufferRef）的队列，不过要自己加锁。
- [CMBufferQueue](https://developer.apple.com/documentation/coremedia/cmbufferqueue?language=objc "CMBufferQueue")：支持存储任何 CFTypeRef 类型的数据，但是数据类型需要有 duration 的概念，在创建 CMBufferQueue 的时候，会有一些回调，其中一个必须的回调是要返回队列中对象的 duration。CMBufferQueue 是设计用于在生产者/消费者模型中在不同的线程中读写数据。通常是两个线程（一个是生产者入队线程，一个是消费者出队线程），当然更多的线程也是可以的。
- [CMMemoryPool](https://developer.apple.com/documentation/coremedia/cmmemorypool-u89?language=objc "CMMemoryPool")：内存池容器，对使用大块的内存有优化。一个 CMMemoryPool 的实例实际上维护一个最近释放内存的池子用于内存分配服务。这样的目的是加快随后的内存分配。在需要重复分配大块内存时，比如输出视频编码数据，可以使用这个数据结构。



## 4、Audio Toolbox Framework


[Audio Toolbox Framework](https://developer.apple.com/documentation/audiotoolbox?language=objc "Audio Toolbox")：提供了音频录制、播放、流解析、编码格式转换、Audio Session 管理等功能接口。下面是 Audio Toolbox 框架的主要模块：

1）[Audio Units](https://developer.apple.com/documentation/audiotoolbox?language=objc "Audio Units")：音频单元。关于 Audio Unit 的内容，还可以参考上面讲到的 Audio Unit Framework。

- [Audio Unit v3 Plug-Ins](https://developer.apple.com/documentation/audiotoolbox/audio_unit_v3_plug-ins?language=objc "Audio Unit v3 Plug-Ins")：基于 AUv3 应用扩展接口来提供自定义的音效、乐器及其他音频能力。
- [Audio Components](https://developer.apple.com/documentation/audiotoolbox/audio_components?language=objc "Audio Components")：定义了发现、加载、配置和关闭音频组件（包括音频单元（audio unit）、音频编解码器（audio codec））的接口。
- [Audio Unit v2 (C) API](https://developer.apple.com/documentation/audiotoolbox/audio_unit_v2_c_api?language=objc "Audio Unit v2 (C) API")：配置一个音频单元（audio unit）以及进行音频渲染。
- [Audio Unit Properties](https://developer.apple.com/documentation/audiotoolbox/audio_unit_properties?language=objc "Audio Unit Properties")：获取有关内置混音器、均衡器、滤波器、特效及音频应用扩展的信息。
- [Audio Unit Voice I/O](https://developer.apple.com/documentation/audiotoolbox/audio_unit_voice_i_o?language=objc "Audio Unit Voice I/O")：配置系统语音处理、响应语音事件。


2）[Playback and Recording](https://developer.apple.com/documentation/audiotoolbox?language=objc "Playback and Recording")：音频播放和录制。

- [Audio Queue Services](https://developer.apple.com/documentation/audiotoolbox/audio_queue_services?language=objc "Audio Queue Services")：提供了简单的、低开销的方式来录制和播放音频的 C 语言接口。支持 Linear PCM、AAC 的录制和播放。实现了连接音频硬件、管理内存、根据需要使用解码器解码音频、调解录音和播放。但是要实现低延迟、回声消除、混音等功能，还得使用 AudioUnit。
- [Audio Services](https://developer.apple.com/documentation/audiotoolbox/audio_services?language=objc "Audio Services")：提供了一组 C 语言接口来实现播放短声或触发 iOS 设备的振动效果。
- [Music Player](https://developer.apple.com/documentation/audiotoolbox/music_player?language=objc "Music Player")：支持播放一组音轨，并管理播放的各种的事件。

3）[Audio Files and Formats](https://developer.apple.com/documentation/audiotoolbox?language=objc "Audio Files and Formats")：音频文件和格式。


- [Audio Format Services](https://developer.apple.com/documentation/audiotoolbox/audio_format_services?language=objc "Audio Format Services")：获取音频格式和编解码器的信息。
- [Audio File Services](https://developer.apple.com/documentation/audiotoolbox/audio_file_services?language=objc "Audio File Services")：从磁盘或内存读写各种音频数据。
- [Extended Audio File Services](https://developer.apple.com/documentation/audiotoolbox/extended_audio_file_services?language=objc "Extended Audio File Services")：通过组合 Audio File Services 和 Audio Converter Services 来提供读写音频编码文件或 LPCM 音频文件。
- [Audio File Stream Services](https://developer.apple.com/documentation/audiotoolbox/audio_file_stream_services?language=objc "Audio File Stream Services")：解析音频流数据。
- [Audio File Components](https://developer.apple.com/documentation/audiotoolbox/audio_file_components?language=objc "Audio File Components")：获取音频文件格式以及文件中包含的数据的信息。
- [Core Audio File Format](https://developer.apple.com/documentation/audiotoolbox/core_audio_file_format?language=objc "Core Audio File Format")：解析 Core Audio 文件的结构。


4）[Utilities](https://developer.apple.com/documentation/audiotoolbox?language=objc "Utilities")：其他音频功能支持。


- [Audio Converter Services](https://developer.apple.com/documentation/audiotoolbox/audio_converter_services?language=objc "Audio Converter Services")：音频编解码。支持 LPCM 各种格式转换，以及 LPCM 与编码格式（如 AAC）的转换。常用的接口：
	- [AudioConverterNew(...)](https://developer.apple.com/documentation/audiotoolbox/1502936-audioconverternew?language=objc "AudioConverterNew(...)")：根据指定的输入和输出音频格式创建对应的转换器（编解码器）实例。
	- [AudioConverterNewSpecific(...)](https://developer.apple.com/documentation/audiotoolbox/1503356-audioconverternewspecific?language=objc "AudioConverterNewSpecific(...)")：根据指定的 codec 来创建一个新的音频转换器（编解码器）实例。
	- [AudioConverterReset(...)](https://developer.apple.com/documentation/audiotoolbox/1503102-audioconverterreset?language=objc "AudioConverterReset(...)")：重置音频转换器（编解码器）实例，并清理它的缓冲区。
	- [AudioConverterDispose(...)](https://developer.apple.com/documentation/audiotoolbox/1502671-audioconverterdispose?language=objc "AudioConverterDispose(...)")：释放音频转换器（编解码器）实例。
	- [AudioConverterGetProperty(...)](https://developer.apple.com/documentation/audiotoolbox/1502731-audioconvertergetproperty?language=objc "AudioConverterGetProperty(...)")：获取音频转换器（编解码器）的属性。
	- [AudioConverterSetProperty(...)](https://developer.apple.com/documentation/audiotoolbox/1501675-audioconvertersetproperty?language=objc "AudioConverterSetProperty(...)")：设置音频转换器（编解码器）的属性。
	- [AudioConverterConvertBuffer(...)](https://developer.apple.com/documentation/audiotoolbox/1503345-audioconverterconvertbuffer?language=objc "AudioConverterConvertBuffer(...)")：只用于一种特殊的情况下将音频数据从一种 LPCM 格式转换为另外一种，并且前后采样率一致。这个接口不支持大多数压缩编码格式。
	- [AudioConverterFillComplexBuffer(...)](https://developer.apple.com/documentation/audiotoolbox/1503098-audioconverterfillcomplexbuffer?language=objc "AudioConverterFillComplexBuffer(...)")：转换（编码）回调函数提供的音频数据，支持不交错和包格式。大部分情况下都建议用这个接口，除非是要将音频数据从一种 LPCM 格式转换为另外一种。
	- [AudioConverterComplexInputDataProc](https://developer.apple.com/documentation/audiotoolbox/audioconvertercomplexinputdataproc?language=objc "AudioConverterComplexInputDataProc")：为 `AudioConverterFillComplexBuffer(...)` 接口提供输入数据的回调。
- [Audio Codec](https://developer.apple.com/documentation/audiotoolbox/audio_codec?language=objc "Audio Codec")： 提供了支持将音频数据进行编码格式转换的 API。具体支持哪些编码格式取决于系统提供了哪些编解码器。



## 5、AVFoundation Framework


[AVFoundation Framework](https://developer.apple.com/documentation/avfoundation?language=objc "AVFoundation Framework") 是更上层的面向对象的一个音视频处理框架。它提供了音视频资源管理、相机设备管理、音视频处理、系统级音频交互管理的能力，功能非常强大。如果对其功能进行细分，可以分为如下几个模块：

- Assets，音视频资源管理。
- Playback，媒体播放及自定义播放行为支持。
- Capture，内置及外置的相机、麦克风等采集设备管理，图片、音视频录制。
- Editing，音视频编辑。
- Audio，音频播放、录制和处理，App 系统音频行为配置。
- Speech，文本语音转换。

在我们前面的 Demo 中封装 Muxer 和 Demuxer 及设置 AudioSession 时会用到 AVFoundation Framework 的一些能力，我们这里对应地介绍一下。


- [AVAssetWriter](https://developer.apple.com/documentation/avfoundation/avassetwriter?language=objc "AVAssetWriter")：支持将媒体数据写入 QuickTime 或 MPEG-4 格式的文件中，支持对多轨道的媒体数据进行交错处理来提高播放和存储的效率，支持对媒体采样进行转码，支持写入 metadata。需要注意的是，一个 AVAssetWriter 实例只能对应写一个文件，如果要写入多个文件，需要创建多个 AVAssetWriter 实例。
	- [canAddInput:](https://developer.apple.com/documentation/avfoundation/avassetwriter/1387863-canaddinput?language=objc "canAddInput:")：检查 AVAssetWriter 是否支持添加对应的 AVAssetWriterInput。
	- [addInput:](https://developer.apple.com/documentation/avfoundation/avassetwriter/1390389-addinput?language=objc "addInput:")：给 AVAssetWriter 添加一个 AVAssetWriterInput。注意必须在 AVAssetWriter 开始写入之前添加。
	- [startWriting](https://developer.apple.com/documentation/avfoundation/avassetwriter/1386724-startwriting?language=objc "startWriting")：开始写入。必须在配置好 AVAssetWriter 添加完 AVAssetWriterInput 做好准备后再调用这个方法。在调用完这个方法后，需要调用 `startSessionAtSourceTime:` 开始写入会话，此后就可以使用对应的 AVAssetWriterInput 来写入媒体采样数据。
	- [startSessionAtSourceTime:](https://developer.apple.com/documentation/avfoundation/avassetwriter/1389908-startsessionatsourcetime?language=objc "startSession(atSourceTime:)")：开启写入会话。在 `startWriting` 后调用，在写入媒体采样数据之前调用。
	- [endSessionAtSourceTime:](https://developer.apple.com/documentation/avfoundation/avassetwriter/1389921-endsessionatsourcetime?language=objc "endSessionAtSourceTime:")：结束写入会话。结束时间是会话结束时样本数据在时间轴上的时刻。如果没有显示调用这个方法，系统会在你调用 `finishWritingWithCompletionHandler:
` 结束写入时自动调用。
	- [finishWritingWithCompletionHandler:](https://developer.apple.com/documentation/avfoundation/avassetwriter/1390432-finishwriting?language=objc "finishWritingWithCompletionHandler:")：标记 AVAssetWriter 的所有 input 为结束，完成写入。为了保证 AVAssetWriter 完成所有采样数据的写入，要在调用添加数据正确返回后调用这个方法。
	- [cancelWriting](https://developer.apple.com/documentation/avfoundation/avassetwriter/1387234-cancelwriting?language=objc "cancelWriting")：取消创建输出文件。如果 AVAssetWriter 的状态是 Failed 或 Completed，调用这个方法无效，否则，调用它会阻塞调用线程，直到会话取消完成。如果 AVAssetWriter 已经创建了输出文件，调用这个方法会删除这个文件。
- [AVAssetWriterInput](https://developer.apple.com/documentation/avfoundation/avassetwriterinput?language=objc "AVAssetWriterInput")：用于向 AVAssetWriter 实例的输出文件的一个轨道添加媒体采样数据。一个实例只能对应一个轨道媒体数据或 metadata 数据的写入，当使用多个实例向多个轨道写入数据时，需要注意检查 AVAssetWriterInput 的 readyForMoreMediaData 属性。
	- [expectsMediaDataInRealTime](https://developer.apple.com/documentation/avfoundation/avassetwriterinput/1387827-expectsmediadatainrealtime?language=objc "expectsMediaDataInRealTime")：输入是否为实时数据源，比如相机采集。当设置这个值为 YES 时，会优化用于实时使用的输入来精准计算 `readyForMoreMediaData` 的状态。
	- [readyForMoreMediaData](https://developer.apple.com/documentation/avfoundation/avassetwriterinput/1389084-readyformoremediadata?language=objc "readyForMoreMediaData")：表示 AVAssetWriterInput 是否已经准备好接收媒体数据。
	- [requestMediaDataWhenReadyOnQueue:usingBlock:](https://developer.apple.com/documentation/avfoundation/avassetwriterinput?language=objc "requestMediaDataWhenReadyOnQueue:usingBlock:")：告诉 AVAssetWriterInput 在方便的时候去请求数据并写入输出文件。在对接拉取式的数据源时，可以用这个方法。
	- [appendSampleBuffer:](https://developer.apple.com/documentation/avfoundation/avassetwriterinput/1389566-appendsamplebuffer?language=objc "appendSampleBuffer:")：通过 AVAssetWriterInput 向输出文件添加媒体数据，但是添加之前媒体数据的顺序需要自己处理。注意，调用这个方法添加采样数据后，不要更改采样数据的内容。
	- [markAsFinished](https://developer.apple.com/documentation/avfoundation/avassetwriterinput/1390122-markasfinished?language=objc "markAsFinished")：标记 AVAssetWriterInput 为完成，表示已经完成向它添加媒体数据了。
- [AVAssetReader](https://developer.apple.com/documentation/avfoundation/avassetreader?language=objc "AVAssetReader")：用于从 AVAsset 资源中读取媒体数据。这个 AVAsset 可以是 QuickTime 或 MPEG-4 文件，也可以是编辑创作的 AVComposition。
	- [canAddOutput:](https://developer.apple.com/documentation/avfoundation/avassetreader/1387485-canaddoutput?language=objc "canAddOutput:")：检查 AVAssetReader 是否支持添加对应的 AVAssetReaderOutput。
	- [addOutput:](https://developer.apple.com/documentation/avfoundation/avassetreader/1390110-addoutput?language=objc "addOutput:")：给 AVAssetReader 添加一个 AVAssetReaderOutput。注意必须在 AVAssetReader 开始读取之前添加。
	- [startReading](https://developer.apple.com/documentation/avfoundation/avassetreader/1390286-startreading?language=objc "startReading")：开始读取。
	- [cancelReading](https://developer.apple.com/documentation/avfoundation/avassetreader/1390258-cancelreading?language=objc "cancelReading")：在读完数据之前取消读取可以调用这个接口。
- [AVAssetReaderOutput](https://developer.apple.com/documentation/avfoundation/avassetreaderoutput?language=objc "AVAssetReaderOutput")：一个抽象类，定义了从 AVAsset 资源中读取媒体采样数据的接口。通常我们可以使用 `AVAssetReaderTrackOutput`、`AVAssetReaderVideoCompositionOutput` 等具体的实现类。
- [AVAssetReaderTrackOutput](https://developer.apple.com/documentation/avfoundation/avassetreadertrackoutput?language=objc "AVAssetReaderTrackOutput")：
	- [alwaysCopiesSampleData](https://developer.apple.com/documentation/avfoundation/avassetreaderoutput/1389189-alwayscopiessampledata?language=objc "alwaysCopiesSampleData")：是否总是拷贝采样数据。如果要修改读取的采样数据，可以设置 YES，否则就设置 NO，这样性能会更好。
	- [copyNextSampleBuffer](https://developer.apple.com/documentation/avfoundation/avassetreaderoutput/1385732-copynextsamplebuffer?language=objc "copyNextSampleBuffer")：从 Output 拷贝下一个 CMSampleBuffer。
- [AVAudioSession](https://developer.apple.com/documentation/avfaudio/avaudiosession?language=objc "AVAudioSession")：在最新版本的 iOS 系统库中，AVAudioSession 已经迁移到 AVFAudio Framework 中了。AVAudioSession 是系统用来管理 App 对音频硬件资源的使用的，比如：设置当前 App 与其他 App 同时使用音频时，是否混音、打断或降低其他 App 的声音；手机静音键打开时是否还可以播放声音；指定音频输入或者输出设备；是否支持录制或边录制边播放；声音被打断时的通知。我们这里只简单介绍下 Demo 中用到的接口：
	- [setCategory:withOptions:error:](https://developer.apple.com/documentation/avfaudio/avaudiosession/1616442-setcategory?language=objc "setCategory:withOptions:error:")：设置 AudioSession 的类型和选项参数。比如类型为 AVAudioSessionCategoryPlayback 表示支持播放；AVAudioSessionCategoryPlayAndRecord 表示同时支持播放和录制等等。
	- [setMode:error:](https://developer.apple.com/documentation/avfaudio/avaudiosession/1616614-setmode?language=objc "setMode:error:")：设置 AudioSession 的模式。AudioSession 的类型和模式一起决定了 App 如何使用音频。通常需要在激活 AudioSession 之前设置类型和模式。比如模式为 AVAudioSessionModeVideoRecording 表示当期要录制视频；AVAudioSessionModeVoiceChat 表示语音聊天。
	- [setActive:withOptions:error:](https://developer.apple.com/documentation/avfaudio/avaudiosession?language=objc "setActive:withOptions:error:")：激活或释放 AudioSession 的使用。








以上这些框架及 API 基本上可以覆盖我们在前面的 Demo 中用到的能力了。

