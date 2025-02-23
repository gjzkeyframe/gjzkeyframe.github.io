---
title: iOS 视频处理框架及重点 API 合集
description: 介绍 iOS 视频处理框架及重点 API 功能。
author: Keyframe
date: 2025-02-23 09:28:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, 视频]
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

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目的 13 篇 AVDemo 文章中，我们拆解了**音频和视频**的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并基于 iOS 系统 API 实现了 Demo：

- 音频 Demo 合集：
	- [iOS AVDemo（1）：音频采集](https://mp.weixin.qq.com/s/FDR_5cMfAJQgZhSvjgeWYA)
	- [iOS AVDemo（2）：音频编码](https://mp.weixin.qq.com/s/q4n1dYTjcJVJolX-Wrdr9Q)
	- [iOS AVDemo（3）：音频封装](https://mp.weixin.qq.com/s/R86qnQAi2njr6k7tFvTF-w)
	- [iOS AVDemo（4）：音频解封装](https://mp.weixin.qq.com/s/fCZfIXriTXUPcI4d4te_ew)
	- [iOS AVDemo（5）：音频解码](https://mp.weixin.qq.com/s/7Db81B9i16cLuq0jS42bmg)
	- [iOS AVDemo（6）：音频渲染](https://mp.weixin.qq.com/s/xrt277Ia1OFP_XtwK1qlQg)
- 视频 Demo 合集：
	- [iOS AVDemo（7）：视频采集](https://mp.weixin.qq.com/s/CJAhkk9BmhMOXgD2pl_rjg)
	- [iOS AVDemo（8）：视频编码](https://mp.weixin.qq.com/s/M2l-9_W8heu_NjSYKQLCRA)
	- [iOS AVDemo（9）：视频封装](https://mp.weixin.qq.com/s/W17eLiUeCszNM8Kg-rlmBg)
	- [iOS AVDemo（10）：视频解封装](https://mp.weixin.qq.com/s/4Ua9PZllWRLYF79hwsH0DQ)
	- [iOS AVDemo（11）：音视频转封装](https://mp.weixin.qq.com/s/VVItfhebc6L-JQFCGBtapQ)
	- [iOS AVDemo（12）：视频解码](https://mp.weixin.qq.com/s/BIazU0Wd5_p4bx4nKJoH-g)
	- [iOS AVDemo（13）：视频渲染](https://mp.weixin.qq.com/s/4K8xPX_A8NA01ecmA6UCtw)

如果你看完这些 Demo，对 iOS 平台的音视频开发多多少少会有一些认识了。在[《iOS 音频处理框架及重点 API 合集》](https://mp.weixin.qq.com/s/w_5pZoeV0GdcFppIpuvVcw)一文中，我们总结了一下 iOS 音频处理框架以及音频相关的 Demo 中用到的主要 API 和数据结构。

接下来，我们再来总结一下 iOS 视频处理框架以及视频相关的 Demo 中用到的主要 API 和数据结构。

## 1、iOS 视频框架

当我们想要了解 iOS 的视频处理框架时，以下是我们能比较容易找到的两张官方架构图。它们出自 [AVFoundation Programming Guide](https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/00_Introduction.html "AVFoundation Programming Guide") 这篇已经过陈旧过时的文档。

![AVFoundation Stack on iOS](assets/resource/av-demo/av-demo-ios-video-api-3.png)
_AVFoundation Stack on iOS_

![AVFoundation Stack on OS X](assets/resource/av-demo/av-demo-ios-video-api-4.png)
_AVFoundation Stack on OS X_

时至今日，iOS 平台的视频处理框架已经有了很多更新，上图中很多在 OS X 上的模块也有了 iOS 版本的实现。虽然有变化，但是上面的架构图，对我们了解现在 iOS 平台的视频处理框架还是有参考价值的。

根据我们的视频 Demo 中涉及的系统 API，我们这里挑选介绍几个相关的 Framework：

- Video Toolbox Framework
- Core Media Framework
- Core Video Framework
- AVFoundation Framework



## 2、Video Toolbox Framework

[Video Toolbox Framework](https://developer.apple.com/documentation/videotoolbox?language=objc "Video Toolbox Framework") 主要用于支持视频硬编码和硬解码。这里我们主要介绍一下编码和解码相关的 API：

1）[Data Compression](https://developer.apple.com/documentation/videotoolbox?language=objc "Data Compression")：通过一个编码 Session 来管理对输入视频数据的压缩操作。

- [VTCompressionSession](https://developer.apple.com/documentation/videotoolbox/vtcompressionsession?language=objc "VTCompressionSession")：编码器 Session。
	- [VTCompressionSessionCreate(...)](https://developer.apple.com/documentation/videotoolbox/1428285-vtcompressionsessioncreate?language=objc "VTCompressionSessionCreate(...)")：创建编码器 Session。
	- [VTSessionSetProperty(...)](https://developer.apple.com/documentation/videotoolbox/1536144-vtsessionsetproperty?language=objc "VTSessionSetProperty(...)")：设置编解码器 Session 的属性。
	- [VTCompressionSessionPrepareToEncodeFrames(...)](https://developer.apple.com/documentation/videotoolbox/1428283-vtcompressionsessionpreparetoenc?language=objc "VTCompressionSessionPrepareToEncodeFrames(...)")：让编码器尽量初始化编码需要的资源。这个方法调用是可选的，如果没有调用这个方法，所有编码需要的资源初始化会在第一次调用 `VTCompressionSessionEncodeFrame(...)` 时做。
	- [VTCompressionSessionEncodeFrame(...)](https://developer.apple.com/documentation/videotoolbox/1428287-vtcompressionsessionencodeframe?language=objc "VTCompressionSessionEncodeFrame(...)")：送数据给编码器编码。被编码好的数据可能不会立即返回，所以在不要修改送给编码器的 `CVPixelBuffer`。
	- [VTCompressionOutputCallback](https://developer.apple.com/documentation/videotoolbox/vtcompressionoutputcallback?language=objc "VTCompressionOutputCallback")：编码数据回调。
	- [VTCompressionSessionCompleteFrames(...)](https://developer.apple.com/documentation/videotoolbox/1428303-vtcompressionsessioncompletefram?language=objc "VTCompressionSessionCompleteFrames(...)")：强制编码器完成所有或者指定时间点（completeUntilPresentationTimeStamp）及之前的所有帧。
	- [VTCompressionSessionInvalidate(...)](https://developer.apple.com/documentation/videotoolbox/1428295-vtcompressionsessioninvalidate?language=objc "VTCompressionSessionInvalidate(...)")：终止编码。调用该方法后，记得用 CFRelease 释放编码器。
- [VTEncodeInfoFlags](https://developer.apple.com/documentation/videotoolbox/vtencodeinfoflags?language=objc "VTEncodeInfoFlags")：编码时返回编码操作相关信息，可以在调用 `VTCompressionSessionEncodeFrame(...)` 等编码接口时传入，也会在编码数据回调中收到。有如下值：
	- [kVTEncodeInfo_Asynchronous](https://developer.apple.com/documentation/videotoolbox/vtencodeinfoflags/kvtencodeinfo_asynchronous?language=objc "kVTEncodeInfo_Asynchronous")：表示异步编码。
	- [kVTEncodeInfo_FrameDropped](https://developer.apple.com/documentation/videotoolbox/vtencodeinfoflags/kvtencodeinfo_framedropped?language=objc "kVTEncodeInfo_FrameDropped")：表示该帧被丢弃。
- [VTIsHardwareDecodeSupported(...)](https://developer.apple.com/documentation/videotoolbox/2887343-vtishardwaredecodesupported?language=objc "VTIsHardwareDecodeSupported(...)")：当前系统和硬件是否支持指定编解码器类型。

2）[Data Decompression](https://developer.apple.com/documentation/videotoolbox?language=objc "Data Decompression")：通过一个解码 Session 来管理对输入视频数据的解压缩操作。

- [VTDecompressionSession](https://developer.apple.com/documentation/videotoolbox/vtdecompressionsession?language=objc "VTDecompressionSession")：解码器 Session。
	- [VTDecompressionSessionCreate(...)](https://developer.apple.com/documentation/videotoolbox/1536134-vtdecompressionsessioncreate?language=objc "VTDecompressionSessionCreate(...)")：创建解码器 Session。解码数据通过传入的回调（outputCallback）返回。
	- [VTSessionSetProperty(...)](https://developer.apple.com/documentation/videotoolbox/1536144-vtsessionsetproperty?language=objc "VTSessionSetProperty(...)")：设置编解码器 Session 的属性。
	- [VTDecompressionSessionDecodeFrame(...)](https://developer.apple.com/documentation/videotoolbox/1536071-vtdecompressionsessiondecodefram?language=objc "VTDecompressionSessionDecodeFrame(...)")：解码送入的数据。
	- [VTDecompressionOutputCallbackRecord](https://developer.apple.com/documentation/videotoolbox/vtdecompressionoutputcallbackrecord?language=objc "VTDecompressionOutputCallbackRecord")：解码数据回调。
	- [VTDecompressionSessionFinishDelayedFrames(...)](https://developer.apple.com/documentation/videotoolbox/1536101-vtdecompressionsessionfinishdela?language=objc "VTDecompressionSessionFinishDelayedFrames(...)")：指示解码器完成所有输入数据的解码。默认情况下，解码器不会无限期的延迟解码某一帧，除非该帧在输入给解码器时被设置了 `kVTDecodeFrame_EnableTemporalProcessing` 这个 VTDecodeFrameFlags。这个方法会在所有延迟的帧解码输出后返回，要等待它们，则需要调用 `VTDecompressionSessionWaitForAsynchronousFrames(...)`。
	- [VTDecompressionSessionWaitForAsynchronousFrames(...)](https://developer.apple.com/documentation/videotoolbox/1536066-vtdecompressionsessionwaitforasy?language=objc "VTDecompressionSessionWaitForAsynchronousFrames(...)")：等待所有异步或延迟的帧解码完成后再返回。调用这个方法后会自动调用 `VTDecompressionSessionFinishDelayedFrames(...)`，所以使用方不用自己调。
	- [VTDecompressionSessionInvalidate(...)](https://developer.apple.com/documentation/videotoolbox/1536093-vtdecompressionsessioninvalidate?language=objc "VTDecompressionSessionInvalidate(...)")：终止解码。调用该方法后，记得用 CFRelease 释放解码器。
- [VTDecodeInfoFlags](https://developer.apple.com/documentation/videotoolbox/vtdecodeinfoflags?language=objc "VTDecodeInfoFlags")：解码时返回解码操作相关信息，可以在调用 `VTDecompressionSessionDecodeFrame(...)` 等解码接口时传入，也会在解码数据回调中收到。有如下值：
	- [kVTDecodeInfo_Asynchronous](https://developer.apple.com/documentation/videotoolbox/vtdecodeinfoflags/kvtdecodeinfo_asynchronous?language=objc "kVTDecodeInfo_Asynchronous")：异步解码。
	- [kVTDecodeInfo_FrameDropped](https://developer.apple.com/documentation/videotoolbox/vtdecodeinfoflags/kvtdecodeinfo_framedropped?language=objc "kVTDecodeInfo_FrameDropped")：帧被丢弃。
	- [kVTDecodeInfo_ImageBufferModifiable](https://developer.apple.com/documentation/videotoolbox/vtdecodeinfoflags/kvtdecodeinfo_imagebuffermodifiable?language=objc "kVTDecodeInfo_ImageBufferModifiable")：可以修改 Buffer。
- [VTDecodeFrameFlags](https://developer.apple.com/documentation/videotoolbox/vtdecodeframeflags?language=objc "VTDecodeFrameFlags")：用于指导解码器行为的指令。
	- [kVTDecodeFrame_EnableAsynchronousDecompression](https://developer.apple.com/documentation/videotoolbox/vtdecodeframeflags/kvtdecodeframe_enableasynchronousdecompression?language=objc "kVTDecodeFrame_EnableAsynchronousDecompression")：告诉解码器当前帧可以异步解码。但是非强制的。设置允许异步解码之后，解码器会同时解码几帧数据，带来的后果是，解码总体时间更短，但是前面几帧回调的时间可能长一些。
	- [kVTDecodeFrame_DoNotOutputFrame](https://developer.apple.com/documentation/videotoolbox/vtdecodeframeflags/kvtdecodeframe_donotoutputframe?language=objc "kVTDecodeFrame_DoNotOutputFrame")：告诉解码器不对该帧回调解码输出数据，而是返回 NULL。某些情况我们不需要解码器输出帧，比如发生解码器状态错误的时候。
	- [kVTDecodeFrame_1xRealTimePlayback(...)](https://developer.apple.com/documentation/videotoolbox/vtdecodeframeflags/kvtdecodeframe_1xrealtimeplayback?language=objc "kVTDecodeFrame_1xRealTimePlayback(...)")：告诉解码器可以使用低功耗模式解码，设置之后处理器消耗会变少，解码速度会变慢，通常我们不会设置这个参数，因为硬解码使用的是专用处理器，不消耗 CPU，所以越快越好。
	- [kVTDecodeFrame_EnableTemporalProcessing(...)](https://developer.apple.com/documentation/videotoolbox/vtdecodeframeflags/kvtdecodeframe_enabletemporalprocessing?language=objc "kVTDecodeFrame_EnableTemporalProcessing(...)")：通知解码器需要处理帧序，设置之后解码回调会变慢，因为无论是异步解码还是 pts、dts 不相等的时候都需要进行帧排序，会耗时，苹果官方文档不建议我们使用这个参数。

## 3、Core Media Framework

在前面介绍 iOS 音频处理框架时，我们已经介绍过 [Core Media Framework](https://developer.apple.com/documentation/coremedia?language=objc "Core Media") 了，这个 Framework 中定义和封装了 AVFoundation 等更上层的媒体框架需要的媒体处理流水线（包含时间信息）以及其中使用的接口和数据类型。使用 Core Media 层的接口和数据类型可以高效的处理媒体采样数据、管理采样数据队列。这里，我们着重介绍一下其中跟视频处理相关的部分。

1）[Sample Processing](https://developer.apple.com/documentation/coremedia?language=objc "Sample Processing")：采样数据处理。常用的数据类型：

- [CMSampleBuffer](https://developer.apple.com/documentation/coremedia/cmsamplebuffer-u71?language=objc "CMSampleBuffer")：系统用来在音视频处理的 pipeline 中使用和传递媒体采样数据的核心数据结构。你可以认为它是 iOS 音视频处理 pipeline 中的流通货币，摄像头采集的视频数据接口、麦克风采集的音频数据接口、编码和解码数据接口、读取和存储视频接口、视频渲染接口等等，都以它作为参数。通常，CMSampleBuffer 中要么包含一个或多个媒体采样的 CMBlockBuffer，要么包含一个 CVImageBuffer（也作 CVPixelBuffer）。
	- [CMSampleBufferGetFormatDescription(...)](https://developer.apple.com/documentation/coremedia/1489185-cmsamplebuffergetformatdescripti?language=objc "CMSampleBufferGetFormatDescription(...)")：返回 CMSampleBuffer 中的采样数据对应的 CMFormatDescription。
	- [CMSampleBufferGetDataBuffer(...)](https://developer.apple.com/documentation/coremedia/1489629-cmsamplebuffergetdatabuffer?language=objc "CMSampleBufferGetDataBuffer(...)")：返回 CMSampleBuffer 中的 CMBlockBuffer。注意调用方不会持有返回的 CMBlockBuffer，如果想要维护指向它的指针，需要显式 retain 一下。
	- [CMSampleBufferGetPresentationTimeStamp(...)](https://developer.apple.com/documentation/coremedia/1489252-cmsamplebuffergetpresentationtim?language=objc "CMSampleBufferGetPresentationTimeStamp(...)")：获取 CMSampleBuffer 中所有采样的最小的 pts 时间戳。因为 CMSampleBuffer 中的采样是按照解码顺序存储的，展示顺序可能与解码顺序一致，也可能不一致。
	- [CMSampleBufferGetDecodeTimeStamp(...)](https://developer.apple.com/documentation/coremedia/1489404-cmsamplebuffergetdecodetimestamp?language=objc "CMSampleBufferGetDecodeTimeStamp(...)")：获取 CMSampleBuffer 中所有采样的第一个采样的 dts 时间戳。在 CMSampleBuffer 中，采样是以解码顺序存储的，即使与展示顺序不一致。
	- [CMSampleBufferGetSampleAttachmentsArray(...)](https://developer.apple.com/documentation/coremedia/1489189-cmsamplebuffergetsampleattachmen?language=objc "CMSampleBufferGetSampleAttachmentsArray(...)")：获取 CMSampleBuffer 中采用数据对应的附属数据（attachment）数组。这些附属数据可能有下面这些 key：
		- [kCMSampleAttachmentKey_NotSync](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_notsync?language=objc "kCMSampleAttachmentKey_NotSync")：Sync Sample 即 IDR 帧，可以用这个 key 对应的值来判断当前帧是否是 IDR 帧，当对应的值为 kCFBooleanFalse 表示是 IDR 帧。这个属性会被写入媒体文件或从媒体文件中读取。
		- [kCMSampleAttachmentKey_PartialSync](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_partialsync?language=objc "kCMSampleAttachmentKey_PartialSync")：当前帧是否 Partial Sync Sample，Partial Sync Sample 可以不依赖前序帧就完成解码（可认为是普通的 I 帧），两个连续的 Partial Sync Sample 随后的帧也可以不依赖这两帧的前序的帧完成解码。当设置一个帧为 Partial Sync Sample 时，需要同时设置 `kCMSampleAttachmentKey_PartialSync` 和 `kCMSampleAttachmentKey_NotSync` 两个属性为 kCFBooleanTrue（可以认为是 I 帧，但是又区别于 IDR 帧）。这个属性会被写入媒体文件或从媒体文件中读取。
		- [kCMSampleAttachmentKey_DependsOnOthers](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_dependsonothers?language=objc "kCMSampleAttachmentKey_DependsOnOthers")：当前帧是否依赖其他帧才能完成解码。如果对应的值为 kCFBooleanTrue，表示依赖。比如，P 或 B 帧。这个属性会被写入媒体文件或从媒体文件中读取。
		- [kCMSampleAttachmentKey_IsDependedOnByOthers](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_isdependedonbyothers?language=objc "kCMSampleAttachmentKey_IsDependedOnByOthers")：表示当前帧是否被其他帧依赖。如果对应的值为 kCFBooleanFalse，表示不被其他帧依赖，这时候是可以丢掉该帧的。这个属性会被写入媒体文件或从媒体文件中读取。
		- [kCMSampleAttachmentKey_DisplayImmediately](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_displayimmediately?language=objc "kCMSampleAttachmentKey_DisplayImmediately")：如果对应的值为 kCFBooleanTrue，表示当前帧应该马上渲染，即使还未到其 pts 时间。一般是在运行时来使用这个属性来触发一些渲染操作，比如 `AVSampleBufferDisplayLayer` 就可能用到。这个属性不会被写入媒体文件。
		- [kCMSampleAttachmentKey_DoNotDisplay](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_donotdisplay?language=objc "kCMSampleAttachmentKey_DoNotDisplay")：表示当前帧是否只解码不渲染。一般是在运行时来使用这个属性来触发一些渲染操作，比如 `AVSampleBufferDisplayLayer` 就可能用到。这个属性不会被写入媒体文件。
		- [kCMSampleAttachmentKey_EarlierDisplayTimesAllowed](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_earlierdisplaytimesallowed?language=objc "kCMSampleAttachmentKey_EarlierDisplayTimesAllowed")：表示后面的帧是否有更早的显示时间。
		- [kCMSampleAttachmentKey_HasRedundantCoding](https://developer.apple.com/documentation/coremedia/kcmsampleattachmentkey_hasredundantcoding?language=objc "kCMSampleAttachmentKey_HasRedundantCoding")：表示当前帧是否有冗余编码。
- [CMBlockBuffer](https://developer.apple.com/documentation/coremedia/cmblockbuffer-u9i?language=objc "CMBlockBuffer")：一个或多个媒体采样的的裸数据。其中可以封装：音频采集后、编码后、解码后的数据（如：PCM 数据、AAC 数据）；视频编码后的数据（如：H.264 数据）。
	- [CMBlockBufferGetDataPointer(...)](https://developer.apple.com/documentation/coremedia/1489264-cmblockbuffergetdatapointer?language=objc "CMBlockBufferGetDataPointer(...)")：获取访问 CMBlockBuffer 中数据的地址。
- [CMFormatDescription](https://developer.apple.com/documentation/coremedia/cmformatdescription-u8g?language=objc "CMFormatDescription")：用于描述 CMSampleBuffer 中采样的格式信息。
	- [CMFormatDescriptionCreate(...)](https://developer.apple.com/documentation/coremedia/1489182-cmformatdescriptioncreate?language=objc "CMFormatDescriptionCreate(...)")：创建一个 CMFormatDescription。
	- [CMVideoCodecType](https://developer.apple.com/documentation/coremedia/cmvideocodectype?language=objc "CMVideoCodecType")：`typedef FourCharCode CMVideoCodecType`，视频编码类型。
	- [CMMediaType](https://developer.apple.com/documentation/coremedia/1564193-cmmediatype/ "CMMediaType")：`typedef FourCharCode CMMediaType`，媒体类型。
- [CMVideoFormatDescription](https://developer.apple.com/documentation/coremedia/cmvideoformatdescription?language=objc "CMVideoFormatDescription")：`typedef CMFormatDescriptionRef CMVideoFormatDescriptionRef;`CMVideoFormatDescription 是一种 CMFormatDescriptionRef。
	- [CMVideoFormatDescriptionCreate(...)](https://developer.apple.com/documentation/coremedia/1489743-cmvideoformatdescriptioncreate?language=objc "CMVideoFormatDescriptionCreate(...)")：基于 CMVideoCodecType 来创建一个 CMVideoFormatDescription。
	- [CMVideoFormatDescriptionGetDimensions(...)](https://developer.apple.com/documentation/coremedia/1489287-cmvideoformatdescriptiongetdimen?language=objc "CMVideoFormatDescriptionGetDimensions(...)")：返回视频编码后的像素尺寸 `CMVideoDimensions`。


2）[Time Representation](https://developer.apple.com/documentation/coremedia?language=objc "Time Representation")：时间信息表示。常用的数据类型：

- [CMTime](https://developer.apple.com/documentation/coremedia/cmtime-u58?language=objc "CMTime")：用 value/timescale 的方式表示时间。这样可以解决浮点运算时的精度损失问题。timescale 表示时间刻度，通常在处理视频内容时常见的时间刻度为 600，这是大部分常用视频帧率 24fps、25fps、30fps 的公倍数，音频数据常见的时间刻度就是采样率，比如 44100 或 48000。
- [CMTimeRange](https://developer.apple.com/documentation/coremedia/cmtimerange-qts?language=objc "CMTimeRange")：用 start+duration 的方式表示一段时间。
- [CMSampleTimingInfo](https://developer.apple.com/documentation/coremedia/cmsampletiminginfo?language=objc "CMSampleTimingInfo")：一个 CMSampleBuffer 的时间戳信息，包括 pts、dts、duration。


3）[Queues](https://developer.apple.com/documentation/coremedia?language=objc "Queues")：数据容器。常用的数据类型：

- [CMSimpleQueue](https://developer.apple.com/documentation/coremedia/cmsimplequeue?language=objc "CMSimpleQueue")：一个简单地、无锁的 FIFO 队列，可以放 `(void *)` 元素，元素不能是 NULL 或 0，如果元素是指向分配内存的指针，其内存生命周期要在外面自己管理。可以用作音视频采样数据（CMSampleBufferRef）的队列，不过要自己加锁。
- [CMBufferQueue](https://developer.apple.com/documentation/coremedia/cmbufferqueue?language=objc "CMBufferQueue")：支持存储任何 CFTypeRef 类型的数据，但是数据类型需要有 duration 的概念，在创建 CMBufferQueue 的时候，会有一些回调，其中一个必须的回调是要返回队列中对象的 duration。CMBufferQueue 是设计用于在生产者/消费者模型中在不同的线程中读写数据。通常是两个线程（一个是生产者入队线程，一个是消费者出队线程），当然更多的线程也是可以的。
- [CMMemoryPool](https://developer.apple.com/documentation/coremedia/cmmemorypool-u89?language=objc "CMMemoryPool")：内存池容器，对使用大块的内存有优化。一个 CMMemoryPool 的实例实际上维护一个最近释放内存的池子用于内存分配服务。这样的目的是加快随后的内存分配。在需要重复分配大块内存时，比如输出视频编码数据，可以使用这个数据结构。


## 4、Core Video Framework

[Core Video Framework](https://developer.apple.com/documentation/corevideo?language=objc "Core Video Framework") 主要用于支持数字视频及数字图像帧的处理，提供基于处理 Pipeline 的 API，并且同时支持 Metal 和 OpenGL。

这里我们主要介绍 CoreVideo Framework 中的几种数据类型：

- [CVImageBuffer](https://developer.apple.com/documentation/corevideo/cvimagebuffer-q40 "CVImageBuffer")：其中包含媒体流中 CMSampleBuffers 的格式描述、每个采样的宽高和时序信息、缓冲级别和采样级别的附属信息。缓冲级别的附属信息是指缓冲区整体的信息，比如播放速度、对后续缓冲数据的操作等。采样级别的附属信息是指单个采样的信息，比如视频帧的时间戳、是否关键帧等。其中可以封装：视频采集后、解码后等未经编码的数据（如：YCbCr 数据、RGBA 数据）。
- [CVPixelBuffer](https://developer.apple.com/documentation/corevideo/cvpixelbuffer?language=objc "CVPixelBuffer")：`typedef CVImageBufferRef CVPixelBufferRef`，像素缓冲区。这是 iOS 平台进行视频编解码及图像处理相关最重要的数据结构之一。它是在 `CVImageBuffer` 的基础上实现了内存存储。并且，CVPixelBuffer 还可以实现 CPU 和 GPU 共享内存，为图像处理提供更高的效率。
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


## 5、AVFoundation Framework

[AVFoundation Framework](https://developer.apple.com/documentation/avfoundation?language=objc "AVFoundation Framework") 是更上层的面向对象的一个音视频处理框架。它提供了音视频资源管理、相机设备管理、音视频处理、系统级音频交互管理的能力，功能非常强大。如果对其功能进行细分，可以分为如下几个模块：

- Assets，音视频资源管理。
- Playback，媒体播放及自定义播放行为支持。
- Capture，内置及外置的相机、麦克风等采集设备管理，图片、音视频录制。
- Editing，音视频编辑。
- Audio，音频播放、录制和处理，App 系统音频行为配置。
- Speech，文本语音转换。


在我们前面的 Demo 中封装 Video Capture、Muxer、Demuxer 及设置 AudioSession 时会用到 AVFoundation Framework 的一些能力，我们这里对应地介绍一下。



1）Video Capture

关于 iOS 视频采集相关的架构，可以参考下面两张图：

![AVCaptureSession 配置多组输入输出](assets/resource/av-demo/av-demo-ios-video-api-1.png)
_AVCaptureSession 配置多组输入输出_

![AVCaptureConnection 连接单或多输入和单输出](assets/resource/av-demo/av-demo-ios-video-api-2.png)
_AVCaptureConnection 连接单或多输入和单输出_

- [AVCaptureDevice](https://developer.apple.com/documentation/avfoundation/avcapturedevice?language=objc "AVCaptureDevice")：为音频和视频采集会话提供输入的设备，并且可以提供相关硬件设备的控制能力，比如：摄像头选择、曝光、对焦、景深、缩放、闪光灯、夜景、帧率、白平衡、ISO、HDR、颜色空间、几何失真等等。这里不过多介绍，只介绍我们 Demo 中用到的一些接口：
	- [-lockForConfiguration:](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1387810-lockforconfiguration?language=objc "-lockForConfiguration:")：在配置硬件相关的属性时，需要先调用这个方法来锁定。
	- [-unlockForConfiguration:](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1387917-unlockforconfiguration?language=objc "-unlockForConfiguration:")：配置完后，解锁。
	- [-activeFormat](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1389221-activeformat?language=objc "-activeFormat")：属性，获取或设置当前采集设备采集的媒体格式。
	- [-activeVideoMinFrameDuration](https://developer.apple.com/documentation/avfoundation/avcapturedevice/1389290-activevideominframeduration?language=objc "-activeVideoMinFrameDuration")：属性，获取或设置最低帧率。
	- [-activeVideoMaxFrameDuration]( "-activeVideoMaxFrameDuration")：属性，获取或设置最高帧率。
- [AVCaptureDeviceInput](https://developer.apple.com/documentation/avfoundation/avcapturedeviceinput?language=objc "AVCaptureDeviceInput")：采集输入，从采集设备提供采集数据给采集会话。
- [AVCaptureDeviceFormat](https://developer.apple.com/documentation/avfoundation/avcapturedeviceformat?language=objc "AVCaptureDeviceFormat")：用于采集设备的媒体格式或采集配置，比如视频分辨率、帧率等。
- [AVCaptureDevicePosition](https://developer.apple.com/documentation/avfoundation/avcapturedeviceposition?language=objc "AVCaptureDevicePosition")：采集设备物理位置。
- [AVCaptureSession](https://developer.apple.com/documentation/avfoundation/avcapturesession?language=objc "AVCaptureSession")：采集会话。用于管理采集活动，协调采集数据在采集设备和采集输出对象之间的流转。
	- [-sessionPreset](https://developer.apple.com/documentation/avfoundation/avcapturesession/1389696-sessionpreset?language=objc "-sessionPreset")：获取或设置采集预设配置，设定采集输出的质量级别或码率。
	- [-addInput:](https://developer.apple.com/documentation/avfoundation/avcapturesession/1387239-addinput?language=objc "-addInput:")：为采集会话添加输入。
	- [-addOutput:](https://developer.apple.com/documentation/avfoundation/avcapturesession/1387325-addoutput?language=objc "-addOutput:")：为采集会话添加输出。
	- [-addConnection:](https://developer.apple.com/documentation/avfoundation/avcapturesession/1389687-addconnection?language=objc "-addConnection:")：为采集会话添加连接对象。
- [AVCaptureSessionPreset](https://developer.apple.com/documentation/avfoundation/avcapturesessionpreset?language=objc "AVCaptureSessionPreset")：采集预设配置，表示采集输出的质量级别或码率。比如：AVCaptureSessionPresetLow、AVCaptureSessionPresetHigh、AVCaptureSessionPreset1280x720、AVCaptureSessionPreset1920x1080 等。
- [AVCaptureSessionRuntimeErrorNotification](https://developer.apple.com/documentation/avfoundation/avcapturesessionruntimeerrornotification?language=objc "AVCaptureSessionRuntimeErrorNotification")：采集会话是否发生错误的通知。
- [AVCaptureConnection](https://developer.apple.com/documentation/avfoundation/avcaptureconnection?language=objc "AVCaptureConnection")：在采集会话中连接一对采集输入和输出。可以设置采集视频镜像、防抖等。
	- [-videoMirrored](https://developer.apple.com/documentation/avfoundation/avcaptureconnection/1389172-videomirrored?language=objc "-videoMirrored")：经过 Connection 的视频是否镜像。
	- [-preferredVideoStabilizationMode](https://developer.apple.com/documentation/avfoundation/avcaptureconnection/1620484-preferredvideostabilizationmode?language=objc "preferredVideoStabilizationMode")：设置采集图像防抖模式。
- [AVCaptureVideoDataOutput](https://developer.apple.com/documentation/avfoundation/avcapturevideodataoutput?language=objc "AVCaptureVideoDataOutput")：采集视频的输出对象。提供访问视频帧进行图像处理的能力，可以通过 `-captureOutput:didOutputSampleBuffer:fromConnection:` 回调获取采集的数据。
	- [-setSampleBufferDelegate:queue:](https://developer.apple.com/documentation/avfoundation/avcapturevideodataoutput/1389008-setsamplebufferdelegate?language=objc "-setSampleBufferDelegate:queue:")：设置采集视频的回调和任务队列。
	- [-captureOutput:didOutputSampleBuffer:fromConnection:]( "-captureOutput:didOutputSampleBuffer:fromConnection:")：采集视频输出数据回调。
	- [-alwaysDiscardsLateVideoFrames](https://developer.apple.com/documentation/avfoundation/avcapturevideodataoutput/1385780-alwaysdiscardslatevideoframes?language=objc "-alwaysDiscardsLateVideoFrames")：采集视频输出时，当帧到的太晚是否丢弃。默认 YES。如果设置 NO，会给 `-captureOutput:didOutputSampleBuffer:fromConnection:` 更多时间处理帧，但是这时候内存占用可能会更大。
- [AVCaptureVideoPreviewLayer](https://developer.apple.com/documentation/avfoundation/avcapturevideopreviewlayer?language=objc "AVCaptureVideoPreviewLayer")：可以用来渲染采集视频数据的 Core Animation Layer。
	- [-setVideoGravity:](https://developer.apple.com/documentation/watchkit/wkinterfacemovie/1628130-setvideogravity?language=objc "-setVideoGravity:")：设置渲染的内容填充模式。


2）Muxer

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


3）Demuxer


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













